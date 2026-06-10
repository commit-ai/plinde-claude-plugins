---
name: pandoc
description: Use this skill when converting documents between formats (Markdown, DOCX, PDF, HTML, PPTX, LaTeX, etc.) using pandoc. Use for format conversion, document generation, slide shows and presentations, preparing markdown for Google Docs or other word processors, and applying custom styles via reference docs. Trigger whenever the user mentions pandoc, document conversion, creating slides/presentations from markdown, or wants to produce a styled Word or PowerPoint file.
---

# Pandoc Document Conversion Skill

Convert documents between formats using pandoc.

**For DOCX-specific details** (styles, reference docs, Google Docs): read `references/docx.md`
**For PPTX/slide shows** (layouts, speaker notes, columns, reference pptx): read `references/pptx.md`

## Prerequisites

```bash
pandoc --version          # verify installed
brew install pandoc       # install if missing
```

## Core Principle: Never Modify Input

When the user provides input content, convert it as-is. Do not rewrite, reformat, or restructure the content — only control the pandoc command and options.

## Common Conversions

### Markdown → Word (.docx)

```bash
pandoc input.md -o output.docx
pandoc input.md --reference-doc=template.docx -o output.docx   # styled
pandoc input.md --toc -o output.docx                            # with TOC
```

See `references/docx.md` for full details on reference docs and custom styles.

### Markdown → PowerPoint (.pptx)

```bash
pandoc input.md -o output.pptx
pandoc input.md --reference-doc=template.pptx -o output.pptx   # styled
pandoc input.md --slide-level=2 -o output.pptx                  # explicit slide level
```

See `references/pptx.md` for full details on slide structure, layouts, speaker notes.

### Markdown → PDF

```bash
# Requires LaTeX: brew install --cask basictex
pandoc input.md -o output.pdf
pandoc input.md --toc --toc-depth=2 -V geometry:margin=1in -o output.pdf

# Unicode characters (arrows, box-drawing, emojis)
export PATH="/Library/TeX/texbin:$PATH"
pandoc input.md --pdf-engine=xelatex -o output.pdf
```

| Engine | Use when |
|--------|----------|
| `pdflatex` | Default, ASCII content |
| `xelatex` | Unicode characters |
| `lualatex` | Complex typography, OpenType fonts |

**No LaTeX?** Use the HTML print-to-PDF workflow below.

### Markdown → HTML

**Always use `-f gfm`** for reliable list and line-break handling.

```bash
# Recommended: GFM with inline styling
pandoc -f gfm -s -H <(cat << 'STYLE'
<style>
body{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;max-width:800px;margin:0 auto;padding:2em;line-height:1.6}
h1{border-bottom:2px solid #333;padding-bottom:0.3em}
h2{border-bottom:1px solid #ccc;padding-bottom:0.2em;margin-top:1.5em}
ul,ol{margin:0.5em 0 0.5em 1.5em;padding-left:1em}
ul{list-style-type:disc}ol{list-style-type:decimal}
li{margin:0.3em 0}
table{border-collapse:collapse;width:100%;margin:1em 0}
th,td{border:1px solid #ddd;padding:8px;text-align:left}
th{background-color:#f5f5f5}
code{background-color:#f4f4f4;padding:2px 6px;border-radius:3px}
pre{background-color:#f4f4f4;padding:1em;overflow-x:auto;border-radius:5px}
blockquote{border-left:4px solid #ddd;margin:1em 0;padding-left:1em;color:#666}
</style>
STYLE
) input.md -o output.html

# Quick
pandoc -f gfm -s input.md -o output.html

# Self-contained (embeds images/CSS)
pandoc -f gfm -s --embed-resources --standalone input.md -o output.html
```

### HTML for Print-to-PDF (no LaTeX)

```bash
pandoc -f gfm input.md -s --toc --toc-depth=2 --embed-resources --standalone -o output.html
open output.html   # then Cmd+P > Save as PDF
```

### Word → Markdown

```bash
pandoc input.docx -o output.md
pandoc -f docx+styles input.docx -o output.md   # preserve custom styles
```

## Useful Options

| Option | Description |
|--------|-------------|
| `-s` / `--standalone` | Standalone document with header/footer |
| `--toc` | Table of contents |
| `--toc-depth=N` | TOC depth (default 3) |
| `--reference-doc=FILE` | Style reference for docx/pptx |
| `--slide-level=N` | Which heading level creates slides |
| `-i` / `--incremental` | Incremental list display in slides |
| `-V key=value` | Set template variable |
| `--metadata key=value` | Set metadata field |
| `--number-sections` | Numbered headings |
| `-f FORMAT` | Input format override |
| `-t FORMAT` | Output format override |

## Format Identifiers

| Format | ID |
|--------|----|
| GitHub Markdown | `gfm` |
| Word | `docx` |
| PowerPoint | `pptx` |
| PDF | `pdf` |
| HTML | `html5` |
| LaTeX | `latex` |
| Beamer slides | `beamer` |
| reveal.js | `revealjs` |

## Troubleshooting

**Lists/lines merging (HTML):** Use `-f gfm`

**Unicode PDF errors:**
```bash
export PATH="/Library/TeX/texbin:$PATH"
pandoc input.md --pdf-engine=xelatex -o output.pdf
```

**"pdflatex not found":**
```bash
brew install --cask basictex
eval "$(/usr/libexec/path_helper)"
```

**Tables not rendering:** Ensure proper markdown table syntax with `|---|` separator row.

**Code blocks missing highlighting:** Use fenced blocks with language: ` ```python `.
