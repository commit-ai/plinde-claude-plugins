# Pandoc DOCX Reference

## Markdown to Word (.docx)

```bash
# Basic conversion
pandoc input.md -o output.docx

# With table of contents
pandoc input.md --toc -o output.docx

# With custom reference doc (for styling)
pandoc input.md --reference-doc=template.docx -o output.docx

# Standalone with metadata
pandoc input.md -s --metadata title="Document Title" -o output.docx
```

## Reference Doc Styling

`--reference-doc` controls all visual styling. The contents of the reference docx are **ignored** — only its stylesheets and document properties (margins, page size, header, footer) are used.

**Get the default reference to modify:**
```bash
pandoc -o custom-reference.docx --print-default-data-file reference.docx
# Then open in Word/LibreOffice, modify styles, save
```

**Paragraph styles pandoc uses:**

| Style | Used for |
|-------|----------|
| `Normal` | Default paragraph |
| `Body Text` | Body text |
| `First Paragraph` | First paragraph in block |
| `Heading 1`–`Heading 9` | Headings |
| `Block Text` | Block quotes |
| `Source Code` | Code blocks |
| `Caption` / `Table Caption` / `Image Caption` | Captions |
| `Title`, `Subtitle`, `Author`, `Date` | Title block |
| `Abstract`, `AbstractTitle` | Abstract |
| `Bibliography` | Bibliography entries |
| `TOC Heading` | Table of contents heading |

**Character styles:**

| Style | Used for |
|-------|----------|
| `Verbatim Char` | Inline code |
| `Hyperlink` | Links |
| `Footnote Reference` | Footnote markers |
| `Section Number` | Auto-numbered sections |

**Table style:** `Table`

## Custom Styles

Apply arbitrary Word styles using `custom-style` attribute on divs and spans:

```markdown
[Emphatically styled text]{custom-style="Emphatically"}

::: {custom-style="Poetry"}
| A Bird came down the Walk---
| He did not know I saw---
:::
```

- Character style → use `[text]{custom-style="StyleName"}` (spans)
- Paragraph style → use `::: {custom-style="StyleName"}` (divs)
- If style doesn't exist in reference doc, pandoc creates it inheriting from Normal
- If style already exists in reference doc, pandoc uses it as-is

## Google Docs Workflow

```bash
# 1. Convert to docx
pandoc document.md -o document.docx

# 2. Upload to Google Drive → Right-click > Open with > Google Docs
```

Preserves: headings, bold/italic, lists, tables, links, code blocks (as monospace).

## Word to Markdown

```bash
pandoc input.docx -o output.md

# Preserve custom styles as divs/spans
pandoc -f docx+styles input.docx -o output.md
```
