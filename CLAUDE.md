# CLAUDE.md — aryaman73.github.io

Personal website / blog. **Hugo** static site, **PaperMod** theme, hosted on **GitHub Pages**.
Live at `aryaman73.github.io` and `aryamans.me` (custom domain via Namecheap; see `CNAME` = `aryamans.me`).

## Repo layout

| Path | Purpose |
|------|---------|
| `config.yml` | Hugo site config (PaperMod). Home blurb, social icons, nav menu, `baseURL`. `params.mainSections: [none]` deliberately keeps blog posts OFF the home page — they're reached via the "blog" nav link (`/posts/`). The home page shows only the `homeInfoParams` intro. |
| `content/posts/*.md` | Blog posts (one markdown file each). Front matter controls `title`, `date`, `draft`, `tags`. |
| `content/posts/<slug>/*.png` | Per-post images (page-bundle style, referenced as `../<slug>/file.png`). |
| `content/resume.pdf` | Résumé, served at `/resume.pdf` (linked from nav menu). |
| `static/` | Files copied verbatim to site root: `favicon.ico`, `CNAME`, and image folders (`react-zendesk/`, `j1-visa-guide/`) referenced as `/folder/file.png`. |
| `themes/PaperMod/` | The theme, committed as plain files (NOT a git submodule — there is no `.gitmodules`). Has a stray embedded `.git/` dir from the original clone. Upstream: github.com/adityatelange/hugo-PaperMod. |
| `archetypes/` | Template for `hugo new posts/...`. |
| `resources/` | Hugo's generated asset cache (resized images, compiled SCSS). |
| `public/` | Hugo build output. **Gitignored** — built by CI, not committed. |
| `.github/workflows/gh-pages.yml` | CI deploy workflow. |
| `CNAME` (root) + `static/CNAME` | Custom-domain marker for GitHub Pages. |
| `assets/css/extended/*.css` | Site CSS on top of the theme. See the skin section below. |
| `layouts/` | Project-level template overrides (see below). |

## The skin — cognitohazardcore

The visual identity: an institutional containment document. Black void, bone-white
document text, one acid-lime hazard accent (`--hazard`) and one containment red
(`--alert`), everything monospace and square-cornered, with a CRT overlay and a
giant slow-turning eye behind the content.

| File | Role |
|------|------|
| `assets/css/extended/z1-cognitohazard-tokens.css` | Palette + metrics. Redefines PaperMod's own vars (`--theme`, `--entry`, `--primary`, `--border`, `--radius`) so every stock rule re-skins itself. |
| `assets/css/extended/z2-cognitohazard-chrome.css` | The iris layer, the CRT overlay, the fixed containment strip, header/nav, footer. |
| `assets/css/extended/z3-cognitohazard-content.css` | Page headers, home dossier, post cards, article typography, redaction, project cards. |
| `layouts/partials/home_info.html` | Fork of the theme's — wraps the `homeInfoParams` intro in the dossier frame. |
| `layouts/shortcodes/redact.html` | `{{</* redact */>}}text{{</* /redact */>}}` — blacked out until hovered/focused. |

Two rules keep it maintainable:

1. **Skin by token, not by rewrite.** Almost nothing here restyles a theme rule
   directly; it changes the variable the theme rule already reads. The three `z`
   files are additive on top of `ai-badge.css` / `mobile-nav.css` /
   `project-cards.css`, which still own layout and structure.
2. **No template forks for chrome.** The containment strip is built from
   `.header::before` / `::after`, and the wordmark glitch is faked with animated
   text-shadows — so neither needed a fork of `header.html`. This follows the same
   choice `extend_footer.html` made for the mobile nav.

## How content flows

`content/` + `static/` + `themes/PaperMod` + `config.yml` → `hugo --minify` → `public/`.
`hugo new posts/name.md` scaffolds a post. Drafts (`draft: true`) are excluded from a normal build (included only with `hugo -D`).

## Deployment workflow

`.github/workflows/gh-pages.yml`: on push to `master`, CI sets up pinned Hugo (extended), runs `hugo --minify`, then `peaceiris/actions-gh-pages@v4` pushes the freshly built `./public` to the `gh-pages` branch, which GitHub Pages serves.

**No manual builds.** Just edit source and push to `master`; CI builds and deploys. `public/` is gitignored.

### History (fixed 2026-06-27)
The build step used to be commented out — the workflow deployed a hand-committed `public/`. README blamed Hugo issue #7087 (PDF handling). That bug is fixed in modern Hugo; a clean `hugo --minify` (verified v0.128.0 extended) builds fine and emits a valid `resume.pdf`. The manual workflow had caused drift — `draft: true` posts (`hello-world`, `ht6`, `why-day`, `My UWaterloo Story`) were stale-published in the committed `public/`. They were intentionally let-drop, so the first CI build removes them from the live site.

Also note: the abandoned `origin/hugo-switch-1` branch was a bad earlier fix attempt (switched branch to `main`, disabled `extended`) — ignore it.

### Possible future improvements
- Move to the official GitHub Pages action (`actions/configure-pages` + `upload-pages-artifact` + `deploy-pages`); requires switching the repo's Pages source from the `gh-pages` branch to "GitHub Actions" in repo Settings.
- Bump the pinned `hugo-version` periodically.

## Local dev

- Preview: `hugo server -D` (`-D` shows drafts), then open the printed localhost URL.
- Build: `hugo --minify`.
- Hugo installed locally via Homebrew (`hugo v0.128.0+extended`).

## Gotchas
- **Extended CSS is concatenated in filename order.** PaperMod's `head.html` does
  `resources.Match "css/extended/*.css"`, so a file that needs to win the cascade
  has to sort after the one it's overriding — that's the only reason the skin
  files are `z`-prefixed. Renaming them silently breaks the overrides.
- **`html` paints the page background, not `body`.** The skin needs both body
  pseudo-elements for its atmosphere layers, so `z2` moves the background up to
  `html` and makes `body` (including PaperMod's `.list` variant) transparent.
  Setting a background on `body` again will bury the iris.
- **Favicons come as a set of five.** PaperMod's `head.html` unconditionally emits five icon
  links — `favicon.ico`, `favicon-16x16.png`, `favicon-32x32.png`, `apple-touch-icon.png`,
  `safari-pinned-tab.svg` — all of which must exist in `static/`. Chrome/Arc prefer the
  explicitly-sized PNGs over the `.ico` and show *no* icon when those 404 rather than falling
  back (this was the bug fixed 2026-07-29 — only the `.ico` existed). Replacing the icon means
  regenerating all five.
- `.DS_Store` files litter the working tree but are gitignored (not tracked) — leave them.
- External links need the scheme: `[text](https://...)`.
- Static images: `![alt](/folder/file.png)`. Per-post bundle images: `![alt](../slug/file.png)`.
