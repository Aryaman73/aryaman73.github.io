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
- `.DS_Store` files litter the working tree but are gitignored (not tracked) — leave them.
- External links need the scheme: `[text](https://...)`.
- Static images: `![alt](/folder/file.png)`. Per-post bundle images: `![alt](../slug/file.png)`.
