---
name: latex-curriculum
description: >-
  Production environment for beautiful, standardized LaTeX mathematics
  curriculum documents — olympiad textbooks, chapter-based teaching
  materials, problem sets with hints and full solutions, worked-example
  handouts, and assessments. Use when the user wants to create, extend,
  compile, or QA a mathematics curriculum/book in LaTeX, needs separate
  student and instructor editions, or asks to typeset theorems, proofs,
  exercises with difficulty ratings, or olympiad problem collections.
  Provides a reusable design system (colors, typography, boxed theorem
  environments), latexmk/Makefile build automation, edition switching,
  and a QA gate. Not for one-off letters/CVs, Beamer slides, or papers
  aimed at journals (use a journal class directly).
---

# LaTeX Curriculum Production

A complete, self-contained production environment for publication-quality
mathematics curriculum documents. Everything compiles with **LuaLaTeX**
via **latexmk**; one source tree produces **Student** and **Instructor**
editions.

## When to use

- "Write a curriculum/textbook/workbook for [topic]"
- "Create a chapter / problem set / handout with solutions"
- "Make an olympiad training book with difficulty stars"
- "Build student and instructor editions from the same source"
- "Check/compile/fix my LaTeX curriculum project"

## Quick start

```bash
# 1. Scaffold a new curriculum (copies this skill's bundled template).
#    SKILL_DIR = the directory containing this SKILL.md, e.g.
#    ~/.agents/skills/latex-curriculum (global) or ~/.zcode/skills/latex-curriculum
SKILL_DIR/templates/curriculum-template/scripts/init-curriculum.sh \
    ~/courses/number-theory "Number Theory I" "Jane Doe"

# 2. Build
cd ~/courses/number-theory
make instructor   # full edition: theory + problems + hints + solutions
make student      # student edition: no solutions, no instructor notes
make chapter C=1  # fast single-chapter build loop
make check        # QA gate (refs, citations, overfull boxes, PDF validity)
make preview      # PNG previews of every page in output/preview/
```

The template's `chapters/chapter-01.tex` is a complete reference
implementation (AM–GM and Cauchy–Schwarz, incl. an IMO problem) — study
it, then replace it with real content.

## Environment requirements

macOS with MacTeX/BasicTeX + user-mode packages:

| Component | Check | Install if missing |
|---|---|---|
| LuaLaTeX | `lualatex --version` | MacTeX (brew install --cask basictex|mactex) |
| latexmk | `latexmk --version` | `curl -fsSL -o ~/.local/bin/latexmk https://mirrors.ctan.org/support/latexmk/latexmk.pl && chmod +x ~/.local/bin/latexmk` |
| Packages | `kpsewhich tcolorbox.sty` | see below |

Packages (user-mode, no sudo): `tcolorbox thmtools enumitem cleveref
titlesec pgfplots tex-gyre tex-gyre-math lualatex-math tikzfill pdfcol
environ trimspaces`:

```bash
tlmgr init-usertree   # once; creates ~/Library/texmf on macOS
tlmgr --usermode install tcolorbox thmtools enumitem cleveref titlesec \
    pgfplots tex-gyre tex-gyre-math lualatex-math tikzfill pdfcol
mktexlsr ~/Library/texmf   # REQUIRED: tlmgr's own mktexlsr skips the user tree
```

## Architecture

```
curriculum-template/
├── main.tex            # edition-agnostic core (loads config/, chapters)
├── student.tex         # wrapper: sets \studentmark -> student edition
├── instructor.tex      # wrapper: sets \instructormark -> + solutions
├── latexmkrc           # LuaLaTeX, output/ dir, bibtex
├── Makefile            # instructor | student | chapter C= | check | preview
├── config/
│   ├── colors.tex      # palette "Oxford Study" — re-skin here only
│   ├── typography.tex  # TeX Gyre Pagella (toggle to Latin Modern)
│   ├── layout.tex      # A4 geometry, fancyhdr, titlesec hierarchy
│   ├── commands.tex    # \difficulty, math macros, curriculum metadata
│   └── environments.tex# all theorem/exercise/hint/solution boxes
├── components/         # cover, frontmatter, chapter summary
├── chapters/chapter-NN.tex
├── exercises/{problems,hints,solutions}-NN.tex
├── references/bibliography.bib
└── scripts/{init-curriculum,add-chapter,qa-check}.sh
```

## Design conventions

**Environment suite** (all breakable across pages):

| Environment | Look | Use |
|---|---|---|
| `definition`, `theorem`, `lemma`, `proposition`, `corollary` | green / navy box, shared theorem counter | named results; amsthm styles via thmtools |
| `example` + `exsolution` | slate box, "Solution." block | worked examples |
| `exercise{d}` / `challenge{d}` | slate / gold box, difficulty stars in title bar | `\begin{exercise}[label=ex:1-1]{3}` |
| `hint` | compact slate box | inline or collected hints |
| `solution{label}` | blue-gray box, titled "Solution to Exercise N" | instructor edition only |
| `objectives` (\objective), `prerequisites`, `notation` | opening furniture | chapter openers |
| `chaptersummary` (\summaryitem) | navy box | chapter closers |
| `keyidea`, `strategy`, `pitfall` | slate / slate / crimson remark boxes | guidance & warnings |
| `instructornote` | crimson box (student edition: silently absorbed) | teaching notes |
| `\answer{...}`, `\source{...}`, `\points{...}` | metadata lines | quick keys, provenance |

**Difficulty scale** — `\difficulty{n}` renders n gold stars (use
`\difficultywhite` on colored bars): 1 routine … 3 hard … 5 IMO-P6 level.

**Editions** — `\ifstudent` guards solutions; wrappers flip it. Keep
solutions in `exercises/solutions-NN.tex`, included only when
`\ifstudent` is false.

**Labels** — `ex:C-N` for exercises, `thm:C-N`, `def:C-N`, `ch:C-N`
for chapters. Solutions reference their exercise:
`\begin{solution}{ex:1-3}`.

## Math typesetting conventions

- `\R \Z \N \Q \C`, `\abs{}` `\norm{}` `\floor{}` `\ceil{}`, `\sumi`
  are predefined — do not redefine per chapter.
- `\coloneqq` for definitions; `\qedhere` for equation-terminal proofs.
- Display equations: `\[ ... \]` (not `$$`); align only when aligning.
- Cross-reference with `\cref{...}` everywhere (capitalized at sentence
  start via `\Cref`); citations `\cite{key}` with natbib+plainnat.
- TikZ/PGFplots figures live OUTSIDE tcolorbox environments (floats
  cannot nest in boxes — the demo chapter shows the pattern).
- Never leave a `\label` dangling after a sectioning command inside boxes.

## QA procedure

`make check` (scripts/qa-check.sh) fails the build on: LaTeX errors,
undefined references/citations, multiply-defined labels, missing glyphs,
broken PDF; warns on overfull hboxes > 3pt. For release also run
`make preview` and visually inspect (or hand pages to a visual-judge
agent): check star visibility on colored bars, box break integrity
across pages, figure label collisions.

## Troubleshooting

| Symptom | Cause & fix |
|---|---|
| `File 'X.sty' not found` right after `tlmgr --usermode install` | Stale ls-R: run `mktexlsr ~/Library/texmf` |
| `tlmgr: package X is not relocatable` (latexmk, biber) | Binary/script packages can't go user-mode; install latexmk from CTAN to `~/.local/bin` (see above); prefer natbib+bibtex over biber |
| nicematrix: "LaTeX release too old" | Frozen BasicTeX kernel; skip nicematrix (booktabs + pmatrix cover curriculum needs) |
| Difficulty stars invisible on a box title bar | Stars colored same as the bar; use `\difficultywhite` |
| `Not in outer par mode` | A float (`figure`/`table`) inside a tcolorbox; move it outside the box |
| `Font shape ... undefined` fontspec italic errors | Wrong font file naming (e.g. Heros uses `-Italic`, not `-Oblique`) |
| cref prints lowercase type name | `\crefname` must come after `\usepackage{cleveref}` (main.tex does this) |
| Everything recompiles from scratch | latexmk remembers failed state; use `latexmk -g` after fixing |
| Overfull hbox on a `\rule{\textwidth}` | New paragraph inherits `\parindent`; prefix `\noindent` |

## Extending

- **New environment**: add a `\tcbset` style + `\newtcolorbox` in
  `config/environments.tex`; register `\crefname` in `main.tex`.
- **Re-skin colors**: edit only `config/colors.tex`.
- **Font switch**: `\pagellafontsfalse` in `config/typography.tex`
  (Latin Modern fallback), or add a new `\if…fonts` branch.
- **New chapter**: `scripts/add-chapter.sh 2 "Title"` wires everything.
