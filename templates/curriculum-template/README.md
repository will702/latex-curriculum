# LaTeX Curriculum Template

A production environment for **publication-quality mathematics
curriculum documents** — olympiad textbooks, chapter-based teaching
materials, problem sets with hints and solutions, and assessments.
One source tree, two editions (Student / Instructor), one command.

```
make instructor   # → output/instructor.pdf  (theory + problems + hints + solutions)
make student      # → output/student.pdf     (problems + hints only)
```

## What's inside

| Path | Purpose |
|---|---|
| `main.tex` | Edition-agnostic master document |
| `student.tex` / `instructor.tex` | Thin wrappers selecting the edition |
| `latexmkrc` | LuaLaTeX engine, `output/` build dir, bibtex wiring |
| `Makefile` | Build, single-chapter loop, QA, previews, scaffolding |
| `config/colors.tex` | The palette ("Oxford Study") — re-skin here only |
| `config/typography.tex` | TeX Gyre Pagella + Pagella Math (one-toggle Latin Modern fallback), microtype |
| `config/layout.tex` | A4 print geometry, running heads, sectioning |
| `config/commands.tex` | `\difficulty` stars, math macros, metadata macros |
| `config/environments.tex` | Theorem family, exercises, hints, solutions, chapter furniture |
| `components/` | Cover page, imprint + TOC, chapter summary |
| `chapters/chapter-01.tex` | **Reference implementation**: AM–GM & Cauchy–Schwarz (with IMO 2001/2) |
| `exercises/` | Per-chapter `problems`/`hints`/`solutions` files |
| `references/bibliography.bib` | natbib bibliography |
| `scripts/` | `init-curriculum.sh`, `add-chapter.sh`, `qa-check.sh` |

## Setup (one-time, macOS)

Requires MacTeX (Basic is enough) with these **user-mode** packages —
no sudo needed:

```bash
tlmgr init-usertree    # once
tlmgr --usermode install tcolorbox thmtools enumitem cleveref \
    titlesec pgfplots tex-gyre tex-gyre-math lualatex-math tikzfill pdfcol
mktexlsr ~/Library/texmf    # tlmgr's own mktexlsr skips the user tree!
```

`latexmk` (single-file Perl tool, no Homebrew formula exists):

```bash
curl -fsSL -o ~/.local/bin/latexmk https://mirrors.ctan.org/support/latexmk/latexmk.pl
chmod +x ~/.local/bin/latexmk
```

## Daily workflow

```bash
make instructor              # full build
make student                 # student edition
make chapter C=1             # iterate on chapter 1 only (fast)
make check                   # QA gate: errors, refs, citations, glyphs, PDF
make preview                 # PNG page previews in output/preview/
make open                    # open the PDF
make clean / make distclean  # remove build artifacts / everything
```

**Add a chapter:**

```bash
scripts/add-chapter.sh 2 "Number Theory Foundations"
# creates chapters/chapter-02.tex + exercises/*-02.tex and wires main.tex
```

**Start a new curriculum from this template:**

```bash
scripts/init-curriculum.sh ~/courses/geometry "Geometry I" "Jane Doe"
# or: make init NAME=../geometry TITLE="Geometry I" AUTHOR="Jane Doe"
```

## Authoring conventions

- **Exercises** carry difficulty stars and a label:
  `\begin{exercise}[label=ex:1-7]{3} ... \end{exercise}`
  (1=routine … 3=hard … 5=IMO P6 caliber). Challenges:
  `\begin{challenge}[label=ch:1-2]{5}`.
- **Solutions** reference their exercise by label and live in
  `exercises/solutions-NN.tex`:
  `\begin{solution}{ex:1-7} ... \end{solution}`.
  The student edition drops the whole solutions section automatically.
- **Instructor-only content**: wrap in `instructornote` (absorbed
  silently in the student edition) or guard with `\ifstudent ... \fi`.
- **Cross-references**: always `\cref{...}`; bookmarks/TOC are
  hyperlinked.
- **Figures** must live outside boxed environments (floats can't nest
  in tcolorboxes).
- **Equality cases** are mandatory in inequality solutions (this is a
  math-education convention the demo chapter models).

## Design language

| Element | Treatment |
|---|---|
| Definitions | forest green edge, pale green wash |
| Theorem family | navy edge, pale blue wash, shared counter |
| Examples / exercises | slate frame, white body |
| Challenges | gold edge, pale gold wash, white stars on gold bar |
| Hints / remarks / strategies | slate, compact |
| Pitfalls | crimson |
| Solutions (instructor) | blue-gray wash |

## Troubleshooting

See the `latex-curriculum` agent skill's `SKILL.md` (installed under your
agent skills directory, e.g. `~/.agents/skills/latex-curriculum/`) for a
full troubleshooting matrix. The two most common:

1. **`X.sty not found` after installing** — run
   `mktexlsr ~/Library/texmf`.
2. **latexmk keeps failing after you fixed the source** — run with
   `-g` to force a rebuild of stale targets.

## Maintenance

- Re-skin an entire series by editing `config/colors.tex` only.
- Switch to Latin Modern: `\pagellafontsfalse` in `config/typography.tex`.
- Upgrading biber/biblatex is unnecessary — natbib+bibtex is wired and
  dependency-free; switch in `main.tex` if your TeX tree allows it.
