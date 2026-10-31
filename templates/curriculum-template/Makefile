# Makefile — LaTeX curriculum build orchestration
# --------------------------------------------------
# Targets:
#   make (or make instructor)  build instructor edition (default)
#   make student               build student edition
#   make chapter C=1           fast single-chapter build (instructor)
#   make check                 run QA checks on both editions
#   make preview               render PNG previews of every page
#   make open                  open instructor PDF in default viewer
#   make clean                 remove build artifacts
#   make init NAME=../my-curriculum TITLE="..." AUTHOR="..."
#                              scaffold a new curriculum from this template

EDITION_INSTRUCTOR := instructor
EDITION_STUDENT    := student
OUT                := output

.DEFAULT_GOAL := instructor

.PHONY: instructor student check preview open clean distclean init chapter

# ---- Editions -----------------------------------------------------------
instructor:
	latexmk $(EDITION_INSTRUCTOR).tex

student:
	latexmk $(EDITION_STUDENT).tex

# Fast single-chapter loop: writes instructor-include.tex picked up by main.tex
chapter:
	@test -n "$(C)" || { echo "Usage: make chapter C=<chapter number, e.g. C=1>"; exit 2; }
	@printf '\\includeonly{chapters/chapter-0%s}\n' "$(C)" > instructor-include.tex
	latexmk -g $(EDITION_INSTRUCTOR).tex
	@echo "Single-chapter PDF: $(OUT)/$(EDITION_INSTRUCTOR).pdf (instructor-include.tex left in place; 'make clean' removes it)"

# ---- Quality ------------------------------------------------------------
check:
	scripts/qa-check.sh

preview: instructor
	@mkdir -p $(OUT)/preview
	gs -q -dNOPAUSE -dBATCH -sDEVICE=png16m -r120 \
	  -sOutputFile=$(OUT)/preview/%03d.png $(OUT)/$(EDITION_INSTRUCTOR).pdf
	@echo "Previews written to $(OUT)/preview/"

open:
	open $(OUT)/$(EDITION_INSTRUCTOR).pdf 2>/dev/null || \
	  open $(OUT)/$(EDITION_STUDENT).pdf

# ---- Hygiene ------------------------------------------------------------
clean:
	latexmk -c
	@rm -f *-include.tex

distclean:
	latexmk -C
	@rm -rf $(OUT)/preview *-include.tex

# ---- Scaffolding ---------------------------------------------------------
init:
	@test -n "$(NAME)" || { echo "Usage: make init NAME=path/to/new-project [TITLE=...] [AUTHOR=...]"; exit 2; }
	scripts/init-curriculum.sh "$(NAME)" "$(TITLE)" "$(AUTHOR)"
