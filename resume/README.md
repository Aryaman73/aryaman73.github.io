# Résumé (editable source)

`resume.tex` is the editable LaTeX source for the résumé. Edit it, then rebuild:

```bash
tectonic resume.tex      # produces resume.pdf
```

`tectonic` (install via `brew install tectonic`) is a self-contained LaTeX
engine — it downloads any needed packages on first run, no full TeX install.

## Publishing to the website
The live site serves `content/resume.pdf` (linked from the nav). To update it:

```bash
cp resume/resume.pdf content/resume.pdf
git add content/resume.pdf resume/
git commit -m "Update resume" && git push   # CI rebuilds + deploys
```

## Tweaks
- **Font:** swap the `lmodern` package line. Sans-serif option: replace
  `\usepackage{lmodern}` with `\usepackage[default]{lato}` (or `\usepackage{helvet}\renewcommand{\familydefault}{\sfdefault}`).
- **Accent color:** edit the `accent` definition (`\definecolor{accent}{HTML}{6B2FA0}`).
- **Margins:** edit the `geometry` margin (currently `0.5in`).
