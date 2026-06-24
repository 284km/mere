# contrib/markdown — Markdown subset converter

Three files for converting Markdown to HTML / plain text / TOC. A CommonMark
subset aimed at personal README generation / blogging / docs automation.

## Files

| file | export | use |
|---|---|---|
| `to_html.mere` | `module MarkdownHtml { render, render_line, render_inline, starts_with }` (`__md_find_double` / `__md_find_single` are kept outside the module due to inner-fn mutual reference) | Markdown line list → HTML |
| `to_text.mere` | `strip_markdown: str -> str` | strip Markdown decorations to plain text |
| `toc.mere` | `extract_toc: str list -> str` | extract headings only and build a nested-list TOC |

## Usage

```sh
cp contrib/markdown/to_html.mere  my_project/
```

The demo at the end of each file may be removed for real use.

## Supported Markdown subset

| feature | to_html | to_text | toc |
|---|---|---|---|
| heading `# ## ###` through `######` | ✓ | ✓ | ✓ |
| unordered list `- foo` (`<ul>`/`<li>` wrapping) | ✓ | ✓ | ✗ |
| ordered list `1. foo` / `2. foo` (`<ol>`/`<li>` wrapping) | ✓ | ✗ | ✗ |
| nested list (2-space indent, **arbitrary depth**) | ✓ | ✗ | ✗ |
| **bold** (`**…**`) | ✓ | ✓ | ✗ |
| *italic* (`*…*` / `_…_`) | ✓ | ✓ | ✗ |
| inline code `` `…` `` | ✓ | ✓ | ✗ |
| blockquote `> …` | ✓ | ✓ | ✗ |
| fenced code block `` ``` `` | ✓ | ✗ | ✗ |
| link `[X](Y)` (`.md` → `.html` auto-rewrite) | ✓ | ✗ | ✗ |
| image `![alt](url)` | ✓ | ✗ | ✗ |
| horizontal rule `---` / `***` / `___` | ✓ | ✗ | ✗ |
| table `\| col \| col \|` + separator row | ✓ | ✗ | ✗ |
| paragraph (blank-line separated) | ✓ | ✓ | ✗ |
| **Unsupported** (future extensions): footnote / definition list / autolink / strikethrough | | | |

## Position

Stage 2 contrib (incubation). Planned to be split out to separate repo
`mere-markdown` as a graduation candidate after public release + pkg manager
lands (internal design notes §3.2).
