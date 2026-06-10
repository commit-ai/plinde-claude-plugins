# Pandoc PPTX (PowerPoint) Reference

## Creating a Presentation

```bash
# Basic — pandoc infers slide structure from headings
pandoc input.md -o output.pptx

# With reference doc for styling/branding
pandoc input.md --reference-doc=template.pptx -o output.pptx

# Set slide level explicitly (default: auto-detected)
pandoc input.md --slide-level=2 -o output.pptx
```

**Critical:** Never modify the input content — only control how it maps to slides.

## Slide Structure

Pandoc auto-detects the **slide level**: the highest heading level that is directly followed by content (not another heading). Use `--slide-level=N` to override.

**Slide boundaries:**
- A heading at the slide level → new slide
- A horizontal rule (`---`) → new slide always
- Headings **above** slide level → "Section Header" title slide
- Headings **below** slide level → sub-heading within the slide

**Example (slide-level=2):**
```markdown
# Section Title         ← Section Header slide

## Slide One            ← new slide
Content here.

## Slide Two            ← new slide
More content.

---                     ← new slide (horizontal rule)
Content on blank slide.
```

## YAML Metadata (Title Slide)

```yaml
---
title: My Presentation
author: Jane Doe
date: 2026-06-10
---
```

This generates the **Title Slide** automatically.

## Layout Mapping

The pptx writer picks layouts from the reference doc by name:

| Layout Name | When used |
|-------------|-----------|
| `Title Slide` | First slide, built from YAML metadata |
| `Title and Content` | Default for all content slides |
| `Section Header` | Headings above slide level ("title slides") |
| `Two Content` | Slides with two-column `{.columns}` div |
| `Comparison` | Two-column slides where a column has text then non-text |
| `Content with Caption` | Non-two-column slide with text then non-text (e.g., image) |
| `Blank` | Slides with only blank content or only speaker notes |

## Reference Doc Styling

The reference pptx controls all visual styling (fonts, colors, backgrounds, layouts).
Content in the reference pptx is ignored — only layouts and theme are used.

**Required layout names:** all seven layouts from the Layout Mapping table above must exist in the reference pptx.

**Get the default reference to modify:**
```bash
pandoc -o custom-reference.pptx --print-default-data-file reference.pptx
# Then open in PowerPoint, modify theme/layouts, save
```

MS PowerPoint 2013+ templates (`.pptx` or `.potx`) work directly.
Check available layouts: Home → Layout.

**Background images:** Set on relevant layouts in the reference pptx.

## Speaker Notes

Supported in pptx output. Add after content on any slide:

```markdown
::: notes
This is my note.

- It can contain Markdown
- like this list
:::
```

**Title slide notes** — use YAML metadata:
```yaml
---
title: My Presentation
notes: |
  Welcome everyone. Remember to introduce yourself.
---
```

Notes appear in PowerPoint's presenter view and handouts.

## Incremental Lists

Default: all items appear at once.

```markdown
::: incremental
- First item
- Second item
- Third item
:::

::: nonincremental
- All at once
- Even with -i flag
:::
```

Or use `-i` / `--incremental` flag to make all lists incremental by default.

Note: the `. . .` pause syntax is **not** supported in pptx.

## Two-Column Layout

```markdown
:::::::::::::: {.columns}
::: {.column width="40%"}
Left column content
:::
::: {.column width="60%"}
Right column content
:::
::::::::::::::
```

Note: `width` attributes are ignored in pptx — column widths are fixed by the layout.

## Per-Slide Background Image

```markdown
## My Slide {background-image="/path/to/image.png"}
Content here.
```

## Inspecting an Existing PPTX

Pandoc can read `.pptx` files as input, not just write them. This is useful for extracting content — including image alt text — without opening PowerPoint.

**Human-readable inspection:**
```bash
pandoc input.pptx -t native
```

The native output shows the parsed document tree. Images appear as `Image` nodes with their alt text as the first argument:

```
Image ("", [], []) [Str "A bar chart showing Q4 revenue"] ("image1.png", "")
```

The bracketed string (`[Str "..."]`) is the alt text.

**Structured extraction with jq:**
```bash
pandoc input.pptx -t json | jq '.. | objects | select(.t == "Image") | .c[1]'
```

This returns each image's alt text as a JSON array of inline elements. To get plain strings:
```bash
pandoc input.pptx -t json \
  | jq -r '.. | objects | select(.t == "Image") | .c[1][] | select(.t == "Str") | .c'
```

**Note:** Alt text is only present when it was set in the original file. Images with no alt text produce an empty array (`[]`).
