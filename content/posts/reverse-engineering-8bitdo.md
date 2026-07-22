---
title: "Reverse-Engineering the 8bitDo Keyboard"
date: 2026-07-18T14:00:00-04:00
draft: false
tags: ["2026", "technical", "ai"]
aiAuthored: true
---

> *This post was written by AI and edited by a human.*

I have an [8BitDo Retro Mechanical Keyboard, N Edition](https://www.8bitdo.com/retro-mechanical-keyboard/), the cream-and-grey one that looks like an NES typewriter. It comes with "Super Buttons" that beg to be mapped to *something*. My something was simple: I wanted one of them to fire a single global hotkey so I could trigger [Wispr Flow](https://wisprflow.ai/) dictation.

![8BitDo Retro Mechanical Keyboard, N Edition](../8bd-config/keyboard.jpg)

The catch: the only software that programs this keyboard is 8BitDo's official Windows app, and I'm on a Mac. There *is* an 8BitDo macOS app, but it only supports a pricier version of the keyboard. Same hardware as the standard Retro Mechanical Keyboard, same USB VID/PID, but the app checks the model name at startup and politely refuses. So the sanctioned path was to boot a Windows machine or VM and remap the keys there. I didn't want to do that. I wanted the remap to live on the keyboard's own firmware, natively, from my Mac.

The usual macOS escape hatch is [Karabiner-Elements](https://karabiner-elements.pqrs.org/), but Karabiner intercepts keys at the OS layer. It's a daemon that has to be running, it remaps *every* keyboard, and it's one more thing to babysit across machines. I didn't want a software shim sitting between me and my keys. I wanted the remap baked into the keyboard the way the official app would do it. So I decided to just talk to the keyboard directly.

## The premise: it's all USB HID

The keyboard's configuration isn't some encrypted black box, it's plain **USB HID** in userland. The good news was that someone had already done the hard archaeology: [goncalor/8bitdo-kbd-mapper](https://github.com/goncalor/8bitdo-kbd-mapper), a GPL-3 Python tool that documented the protocol for the standard keyboard. Since the N Edition is the same silicon, that protocol *should* just work.

I worked through the whole project with **Claude Code** as a pair, and the first thing we did was kill the scariest assumption before writing any real code: *can macOS even let me touch this interface without an entitlement or a permission prompt?*

The keyboard exposes three USB interfaces. Interface #1 is the protected keyboard interface - macOS correctly refuses to let userland touch it, which is the anti-keylogger protection doing its job. But **interface #2** is a vendor config channel on HID usage page `0x8c`, and macOS opens it from userland with no entitlement and no prompt. A tiny [`node-hid`](https://github.com/node-hid/node-hid) probe read the current profile name straight off the keyboard, and the biggest risk was gone.

A side-finding: the upstream Python tool leans on a libusb path that's really Linux-oriented. On macOS the IOHIDManager / `node-hid` route is the one that actually works, which conveniently justified building the whole thing in **TypeScript** so the eventual app could be one language top to bottom.

## Building the thing

The protocol is refreshingly plain. Commands are HID reports where the first byte is a report ID (`0x52` for host→device, `0x54` for device→host), and the handshake is `ATTN` → read → command.

```sh
node src/cli.ts status      # read the current profile + remapped keys (READ-ONLY)
node src/cli.ts list-keys   # every hardware key and every mappable target
node src/cli.ts map capslock esc   # write a remap
```

The read path came first, because reads are harmless - worst case you learn nothing. Then the write path, validated against real hardware one careful command at a time. The one genuinely confusing gotcha cost me an afternoon: a remap *doesn't take physical effect* until you (1) save it into a **named profile** and (2) press the keyboard's little Profile (heart) button so its LED lights up. An unnamed profile lives only on the PC; a named one only applies while the toggle is on.

I drew two lines I never crossed: **no macros** (out of scope) and **no firmware flashing**. Flashing is the only operation that can actually brick the device, it's AES-locked anyway, and there was zero upside for my one-button goal. Everything I did is reversible with a factory reset, which makes poking at your own hardware a lot more fun.

## Cracking chords without a Windows capture

A single keypress maps trivially, but my Wispr hotkey needed a **chord** - ⌘ plus something. I assumed chords would require some special protocol I'd have to sniff off the Windows app over USB. They didn't.

Staring at the 3-byte mapping values, the pattern was right there: it's `07 <modifier-usage> <key-usage>`.

- A plain key leaves the modifier byte zero: `07 00 04` = `a`.
- A plain modifier leaves the key byte zero: `07 e2 00` = Left Alt.
- A **chord fills both**: `07 e3 68` = ⌘+F13.

I mapped the on-board Super A key to `07 e3 04`, pressed it, and macOS received ⌘+A. No packet capture, no Windows machine, just reading the bytes. That became `map-combo`:

```sh
node src/cli.ts map-combo supera cmd f13   # Super A → ⌘+F13, my dictation hotkey
```

⌘+F13 is a chord nothing else on macOS uses, which makes it a clean dedicated trigger for Wispr Flow, Raycast, or a Shortcuts action. This is the part I actually needed, and it's done - fully native, nothing running in the background.

## The rabbit hole: the *external* Super Buttons

I could've stopped there. But the big circular accessories that plug into the keyboard's 3.5mm jack turned out to be a completely different animal from the on-board Super keys, and I couldn't leave them alone.

Every scan of the legacy `0x52` channel came up empty, because these live in a separate, **UUID-addressed accessory store** with its own command family. To find it, I decompiled 8BitDo Ultimate Software V2 (the Windows .NET app) *on macOS*: `ilspycmd` for the managed code, `radare2` for the native `8BitDoAdvance.dll`. Out came the whole protocol - the commands (`0x11` read / `0x12` write / `0x15` delete / `0x17` recognize / `0x19` index), the packet framing, the checksum, the full data model down to the `PAT_KEYBOARD_MAPPING_INFO` struct. As far as I can tell, nobody public had written this down before.

Then I spent a while walking into walls, and it's worth being straight about where they were, because my first read of the situation was wrong.

The commands are 64-byte HID reports on report ID `0x81`. On Windows the app pushes them out with plain `WriteFile`/`ReadFile`, which on a HID handle means the interface's **interrupt endpoints**. Interface #2 has exactly those - an interrupt IN and an interrupt OUT. So the transport is knowable and simple. The problem is purely who's allowed to drive it on macOS:

- macOS's HID API (`IOHIDManager`) only exposes the *control* endpoint (`SET_REPORT`/`GET_REPORT`), never the interrupt OUT pipe. I can shove the `0x81` command down the control endpoint and the device takes it - but the firmware doesn't route it to the accessory handler, so it just gets a generic "don't know this report" reject. The handler only listens on the interrupt endpoint Windows uses.
- So I tried to grab that endpoint directly with raw USB (`IOUSBLib`), skipping HID entirely. macOS said no: `kIOReturnExclusiveAccess`. The built-in `AppleUserHIDDevice` driver has already claimed interface #2 exclusively, and it won't share the pipe with a userland process, seize or not.

(Somewhere in the middle of this I got briefly excited that 8BitDo's *macOS* app might hold the answer, since it clearly programs super buttons through IOKit. It does - but for a different keyboard, so its exact convention wasn't authoritative for mine. Dead end, honestly noted.)

So the native-macOS version of this specific feature is blocked, not by the protocol, but by the OS driver model: the one interface I need is spoken for by Apple's own kernel driver. The only way through *natively* would be to write my own DriverKit extension that claims interface #2 instead - which needs a paid Apple developer account, an entitlement request, and a real gamble on whether macOS even lets a third-party driver preempt a keyboard-class interface. That's a whole driver project to move two buttons. Not worth it.

Here's the part that makes the wall not actually matter: the mapping gets written into the keyboard's **firmware** and stays there. I don't need a permanent macOS tool - I need to send the config exactly once. So the pragmatic answer is to run the real Windows app inside a USB-passthrough VM ([UTM](https://mac.getutm.app/) does this fine on Apple Silicon), let it write to the keyboard, and shut it down. After that the buttons work natively on macOS with nothing running.

Which is, yes, a slightly funny place to land - I set out to avoid Windows and, for the external buttons specifically, the tidy answer is "use Windows, just boxed up in a VM and only once." 😅 But the thing I actually wanted - a super key that fires my dictation hotkey - I got natively, and the full external-accessory protocol is now documented in [`SUPER-BUTTON-PROTOCOL.md`](https://github.com/Aryaman73/8db-retro-config/blob/main/SUPER-BUTTON-PROTOCOL.md) for anyone who wants to take the DriverKit route further than I cared to.

## Wrapping up

The reflex when a device "isn't supported" is to assume there's a wall protecting something. Usually there's just an allowlist, and underneath it the hardware speaks an ordinary protocol that's been sitting in userland the whole time - the model-name check was guarding 8BitDo's product tiers, not me. Where I *did* hit a real wall, it wasn't the vendor at all; it was macOS deciding who gets to hold a USB interface. Different kind of wall, and a fair one.

The code is a small TypeScript + `node-hid` CLI, GPL-3 (inherited from the upstream protocol work it builds on), and it's [up on GitHub](https://github.com/Aryaman73/8db-retro-config). The someday-plan is an Electron GUI with a clickable keyboard layout, so nobody else has to type raw HID usage bytes to get their one button back.
