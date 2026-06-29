---
title: "Reverse-Engineering the 8bitDo Retro Mechanical Keyboard"
date: 2026-06-29T15:45:00-04:00
draft: false
tags: ["2026", "technical"]
---

I have an [8BitDo Retro Mechanical Keyboard, N Edition](https://www.8bitdo.com/retro-mechanical-keyboard/), the cream-and-grey one that looks like an NES typewriter. It comes with "Super Buttons" that beg to be mapped to *something*. My something was simple: I wanted one of them to fire a single global hotkey so I could trigger [Wispr Flow](https://wisprflow.ai/) dictation without contorting my hand.

![8BitDo Retro Mechanical Keyboard, N Edition](../8bd-config/keyboard.jpg)

The catch: the only software that programs this keyboard is 8BitDo's official Windows app, and I only had a mac. There is a similar software provided by 8BitDo on macOS, but it only works with a certain version of the retro keyboard (that is, of course, more expensive). Same hardware as the standard Retro Mechanical Keyboard, same USB VID/PID, but the app checks the model name at startup and politely refuses. So the official path was to boot a Windows machine or VM, install the app, and use it to remap the keys. I, of course, didn't want to do that - I wanted a native macOS solution.

The usual macOS escape hatch is [Karabiner-Elements](https://karabiner-elements.pqrs.org/), but Karabiner intercepts keys at the OS layer. It's a daemon that has to be running, it remaps *every* keyboard, and it's one more thing to babysit across machines. I didn't want a software shim. I wanted the remap to live **on the keyboard's own firmware**, the way the official app would have done it. So I decided to just talk to the keyboard directly.

## The premise: it's all USB HID

Here's the thing that made this tractable. The keyboard's configuration isn't some encrypted black box, it's plain **USB HID** in userland. Someone had already done the hard archaeology: [goncalor/8bitdo-kbd-mapper](https://github.com/goncalor/8bitdo-kbd-mapper), a GPL-3 Python tool that documented the protocol for the standard keyboard. Since the N Edition is the same silicon, that protocol *should* just work.

I worked through this whole project with **Claude Code** as a pair, and the first thing we did was de-risk the scariest assumption before writing any real code: *can macOS even let me touch this interface without an entitlement or a permission prompt?*

The keyboard exposes three USB interfaces. Interface #1 is the protected keyboard interface (macOS correctly refuses to let us touch it, which is the anti-keylogger protection working as intended). But **interface #2** is a vendor config channel on HID usage page `0x8c`, and macOS opens it from userland with no entitlement and no prompt. We confirmed it with a tiny [`node-hid`](https://github.com/node-hid/node-hid) probe, read the current profile name straight off the keyboard, and the single biggest risk evaporated.

A nice side-finding: the upstream Python tool leans on a libusb path that's really Linux-oriented. On macOS the IOHIDManager / `node-hid` route is the one that actually works, which conveniently justified building the whole thing in **TypeScript** so the eventual app could be one language top to bottom.

## Building the thing

The shape of the protocol is satisfyingly simple once you see it. Commands are HID reports where the first byte is a report ID (`0x52` for host→device, `0x54` for device→host) and the handshake is `ATTN` → read → command. From there it was a steady march:

```sh
node src/cli.ts status      # read the current profile + remapped keys (READ-ONLY)
node src/cli.ts list-keys   # every hardware key and every mappable target
node src/cli.ts map capslock esc   # write a remap
```

The read path came first because reads are harmless. Worst case you learn nothing. Then the write path, validated against real hardware one careful command at a time. The one genuinely confusing gotcha cost an afternoon: a remap *doesn't take physical effect* until you (1) save it into a **named profile** and (2) press the keyboard's little Profile (heart) button so its LED lights up. An unnamed profile lives only on the PC; a named one only applies while the toggle is on.

I drew two firm lines in the sand and never crossed them: **no macros** (out of scope) and **no firmware flashing**. Flashing is the *only* operation that can actually brick the device, it's AES-locked anyway, and there was zero upside for my one-button goal. Everything I do is reversible by a factory reset. Reverse-engineering your own hardware is a lot more fun when you know you can't brick it.

## Cracking chords without a Windows capture

A single keypress maps trivially, but my Wispr hotkey needed a **chord**, ⌘ plus something. The natural assumption was that chords required some special protocol I'd have to capture from the Windows app over USB. They didn't.

Staring at the 3-byte mapping values, the encoding gave itself up: it's `07 <modifier-usage> <key-usage>`.

- A plain key leaves the modifier byte zero: `07 00 04` = `a`.
- A plain modifier leaves the key byte zero: `07 e2 00` = Left Alt.
- A **chord fills both**: `07 e3 68` = ⌘+F13.

I mapped the on-board Super A key to `07 e3 04` and pressed it, and macOS received ⌘+A. It physically worked. No packet capture, no Windows machine, just reading the bytes and noticing the symmetry. That became `map-combo`:

```sh
node src/cli.ts map-combo supera cmd f13   # Super A → ⌘+F13, my dictation hotkey
```

⌘+F13 is a chord nothing else on macOS uses, which makes it a perfect dedicated trigger for Wispr Flow, Raycast, or a Shortcuts action. 🫡

## The rabbit hole: the *external* Super Buttons

I could have stopped there. But the big circular accessories that plug into the 3.5mm jack turned out to be a completely *different* animal from the on-board Super keys, and reverse-engineering them was too tempting to leave alone.

Every scan of the legacy `0x52` channel for these came up empty, because they live in a separate, **UUID-addressed accessory store** with its own command family. To find it, we decompiled the canonical 8BitDo Ultimate Software V2 (the Windows .NET app) **on macOS**: `ilspycmd` for the managed code, `radare2` for the native `8BitDoAdvance.dll`. We recovered the entire protocol: commands (`0x11` read / `0x12` write / `0x15` delete / `0x17` recognize / `0x19` index), the packet framing, the checksum, and the full data model right down to the `PAT_KEYBOARD_MAPPING_INFO` struct. As far as I can tell, **nobody public had documented this before.**

And then I hit a wall I'm honest about: the Advance protocol uses **64-byte reports on report ID `0x81`**, but the keyboard's macOS-visible config interface only declares **32-byte** reports. The `0x81` reports simply can't be delivered through that interface on macOS as-is. The protocol is fully decoded; the *transport* on macOS isn't closed. Finishing it cleanly would take a ~1-minute USB capture of the Windows app, and now I know exactly what to look for. So I wrote it all up in [`SUPER-BUTTON-PROTOCOL.md`](https://github.com/Aryaman73/8db-retro-config/blob/main/SUPER-BUTTON-PROTOCOL.md) and parked it. My ⌘+F13 button already does what I needed; this last mile is a "someday" with a clear map.

## What I took away

The reflex when a device "isn't supported" is to assume there's a wall. Usually there's just an allowlist, and underneath it the hardware speaks a perfectly ordinary protocol that's been sitting in userland the whole time. The official app wasn't protecting me from anything, it was protecting 8BitDo's product matrix.

Doing this with Claude Code as a partner changed the *pace* more than anything. The de-risking-first instinct (prove macOS HID access works before writing a single feature), the discipline of read-paths before write-paths, decompiling a .NET binary on the wrong OS, and noticing the `07 <mod> <key>` symmetry instead of reaching for a packet capture.

The code is a small TypeScript + `node-hid` CLI, GPL-3 (inherited from the upstream protocol work it builds on), and it's [up on GitHub](https://github.com/Aryaman73/8db-retro-config). The eventual plan is an Electron GUI with a clickable keyboard layout so nobody else has to type raw HID usage bytes to get their one button back.
