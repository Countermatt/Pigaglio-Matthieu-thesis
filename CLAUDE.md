# Sharded Blockchains — PhD Thesis

This repository is Matthieu Pigaglio's PhD thesis, written in LaTeX, on **blockchain scalability** — specifically sharding and Layer 2 (L2) optimization using **Data Availability Sampling (DAS)**. It uses UCLouvain's (EPL) official thesis template, built to meet CIACO printing requirements.

## Structure

- `thesis_a4.tex` / `thesis_elec.tex` — the two entry points (A4 vs CIACO electronic format). Compile one of these, never `thesis.tex` directly.
- `thesis.tex` — shared preamble: packages, macros, document skeleton (`\include{...}` order). Edited rarely; defines `\thesisTitle` and `\thesisAuthor`.
- `these-ucl.cls` — the UCLouvain thesis class file (template-provided, not authored content).
- `commands.tex` — custom user-defined LaTeX macros.
- `preamble.tex`, `title.tex`, `copyright.tex`, `citation.tex`, `acknowledgments.tex` — front-matter chapters.
- `tips.tex`, `long.tex` — main-matter chapters (content chapters; likely renamed/expanded as the thesis grows).
- `ccl.tex` — conclusion chapter.
- `biblio.bib` — bibliography source, compiled with **Biber** (not BibTeX — the template explicitly requires Biber for biblatex).
- `figures/` — figure assets.
- `plan_these.md` — thesis planning notes (currently empty scratch file).

## Build

Compile via `latexmk` (a `.fdb_latexmk`/`.fls` build cache is checked in from a prior local build). Standard sequence for biblatex+Biber:

```
pdflatex thesis_elec.tex
biber thesis_elec
pdflatex thesis_elec.tex
pdflatex thesis_elec.tex
```

Or simply `latexmk -pdf thesis_elec.tex` if latexmk is configured for biber.

## CI

`.github/` contains a GitHub Actions workflow that auto-builds the thesis PDF on push. Configured via repo env vars `MAIN_LATEX`, `FINAL_FILENAME` (UCLouvain naming convention, e.g. `Pigaglio_XXXXXXXX_2026.pdf`), and `DATE_TIMEZONE`. See `README.md` for details.

## Working conventions

- Never edit `thesis.tex`'s package/macro setup casually — it's shared template infrastructure; prefer adding thesis-specific macros to `commands.tex`.
- Bibliography entries go in `biblio.bib`; cite with biblatex commands (`\cite`, `\fullcite`, or `\myfullcite` for self-citations that should appear in bibliographic notes but not the main reference list — see `thesis.tex` for the `\mybibexclude` mechanism).
- Generated build artifacts (`.aux`, `.bbl`, `.bcf`, `.blg`, `.fdb_latexmk`, `.fls`, `.log`, `.out`, `.run.xml`, `.synctex.gz`, `.toc`, and the built PDF) are build byproducts, not hand-edited source — treat changes to them as noise unless diagnosing a build issue.
