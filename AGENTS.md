# AGENTS

## Purpose

This file defines the authoring rules for working with this Hugo repository.
Keep it focused on repository conventions, Hugo usage, content structure, and asset workflow.
Do not use this file to preserve sample content or legacy template structure.

## Project Overview

Bilingual (English + Vietnamese) Hugo documentation site using the [hugo-theme-learn](https://github.com/matcornic/hugo-theme-learn) theme with GitHub Pages deployment.

- **Hugo version:** 0.134.3 extended
- **Theme:** `hugo-theme-learn` (Git submodule at `themes/hugo-theme-learn`)
- **Config:** `config/_default/hugo.toml` (production), `config/development/hugo.toml` (local)
- **CI/CD:** `.github/workflows/hugo.yml` — builds and deploys to GitHub Pages on push to `main`

## Content Structure

- Organize content as a Hugo page tree.
- A folder with `_index.md` is a section or branch bundle.
- A regular content page can be either:
  - `name.md`
  - `name/_index.md`
- Prefer folder bundles when a page may later need images or attachments.
- Use stable, lowercase, hyphenated names for folders and files.

Example tree:

```
content/
  _index.md              # Home page (English)
  _index.vi.md           # Home page (Vietnamese)
  01-section-name/
    _index.md            # Section landing page (English)
    _index.vi.md         # Section landing page (Vietnamese)
    some-page/
      index.md           # English content
      index.vi.md        # Vietnamese content
      _diagrams/
        diagram.png
      _tools/
        generate.py
```

## Multilingual Content

- This repository uses paired language files in the same folder:
  - `*.md` for English
  - `*.vi.md` for Vietnamese
- Create both language files together for every new page.
- Keep page purpose, front matter shape, and relative placement aligned across both languages.
- All `*.vi.md` files must use proper Vietnamese with diacritics, not ASCII transliteration.
- The English version is the canonical/primary version; write it first, then translate to Vietnamese.
- Translate all visible content (headings, body text, image alt text) but NOT directory names, file names, or code blocks.

## Front Matter

Use front matter consistently so sidebar order and rendering stay predictable.

### Section page

```md
---
title: "Section Title"
date: 2026-04-22
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
```

### Regular page

```md
---
title: "Page Title"
date: 2026-04-22
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---
```

### Conventions

| Field | Required | Description |
|-------|----------|-------------|
| `title` | Yes | Displayed page title |
| `date` | Yes | Creation or publication date (`YYYY-MM-DD`) |
| `weight` | Yes | Sibling ordering (lower = earlier in sidebar) |
| `chapter` | No | Keep `false` unless a chapter-style layout is explicitly needed |
| `pre` | No | HTML prefix for sidebar numbering, e.g. `" <b> 1. </b> "` |
| `draft` | No | Set `true` for unpublished pages (used in archetype) |
| `menuTitle` | No | Short alternative title for sidebar when `title` is too long |

Do not place numbering inside `title`; use `pre` instead.

## Writing Rules

- Use normal Markdown headings.
- Prefer a short introductory paragraph before long sections.
- Store related content in the same bundle when possible.
- If a section should list its child pages, use the `children` shortcode.

Example:

```md
{{% children description="true" /%}}
```

## Assets

- Prefer page-bundled assets for page-specific diagrams, screenshots, and attachments.
- Use `_`-prefixed support folders for page-local assets or tooling, such as `_diagrams/` and `_tools/`.
- Treat folders without publishable content files as support folders, not subpages.
- Keep shared global assets such as favicon, fonts, and site-wide images under `static/`.
- For page-bundled images, use page-relative paths in Markdown.
- Keep asset paths stable after publishing to avoid broken links.
- Keep generator scripts near the page they support when practical.

Example:

```md
![Diagram](_diagrams/example-diagram.png)
```

## Diagram Rules

- Prefer reproducible diagrams generated from nearby `_tools/` scripts when practical.
- Match the diagram layout to the relationship being explained, not only to what the generator defaults produce.
- If a diagram has a `1 -> n` or hub-and-spoke structure, prefer straight arrows over rectangular/elbow connectors.
- Use orthogonal or rectangular connectors only when they improve clarity for dependency chains, layered architecture, or screen-space constraints.

## Screenshot Workflow

- Store original AWS Console screenshots in `raw/` as PNG files.
- Treat `raw/` as immutable source material, not publishable site assets.
- Derive focused publishable images from `raw/` and place them in the related page bundle, typically under `_diagrams/`.
- Keep PNG for capture and intermediate editing.
- Prefer WebP for cropped screenshots served on the website.
- Use stable, descriptive, lowercase, hyphenated filenames based on what the crop proves, not only the step number.
- One cropped image should support one main point, or at most two tightly related points.
- Remove irrelevant browser chrome, AWS header area, empty whitespace, and unrelated panels unless they are needed for orientation.
- If one screenshot contains multiple important details, split it into multiple focused crops instead of embedding one large full-screen image.
- Reuse the same cropped asset across English and Vietnamese pages when the visual evidence is identical.
- If a crop should be reproducible, store its crop specification in a sibling `_tools/` script or manifest.

## Shortcodes

This repository supports these shortcodes. Prefer them over raw HTML for consistent styling.

| Shortcode | Usage |
|-----------|-------|
| `children` | `{{% children description="true" /%}}` — list child pages with descriptions |
| `notice` | `{{% notice info %}}Text{{% /notice %}}` — info/warning/error/tip callout boxes |
| `expand` | `{{% expand "Title" %}}Content{{% /expand %}}` — collapsible sections |
| `tabs` / `tab` | `{{< tabs >}}{{< tab name="Tab1" >}}Content{{< /tab >}}{{< /tabs >}}` — tabbed content |
| `attachments` | `{{% attachments /%}}` — list file attachments in page bundle |
| `ref` | `{{< ref "path/to/page" >}}` — link to another Hugo page by source path |
| `ghcontributors` | `{{< ghcontributors "repo-url" >}}` — GitHub contributor list |

Examples:

```md
{{% notice info %}}
Important content goes here.
{{% /notice %}}
```

```md
{{% expand "Click to expand" %}}
Hidden content here.
{{% /expand %}}
```

### Mermaid Diagrams

Always use the `{{< mermaid >}}` shortcode — never use fenced code blocks (` ```mermaid `). Hugo's Goldmark renderer treats fenced code blocks as `<pre><code>`, which renders as raw text. Only the shortcode produces the `<div class="mermaid">` that triggers client-side rendering.

```md
{{< mermaid >}}
graph LR;
  A-->B;
  B-->C;
{{< /mermaid >}}
```

## Page Creation Checklist

1. Create the English page.
2. Create the matching Vietnamese page in the same folder.
3. Add consistent front matter to both files.
4. Set `weight` and `pre` if sidebar ordering matters.
5. Add page-specific assets under `_diagrams/` or another `_`-prefixed support folder if needed.
6. Run Hugo locally and verify both languages.

## Hugo Commands

Run locally:

```bash
hugo server -D
```

Build production output:

```bash
hugo
```

Production build with minification:

```bash
hugo --gc --minify
```

Regenerate bundled diagrams:

```bash
python3 tools/build_diagrams.py
```

Create a new page quickly:

```bash
hugo new some-section/some-page/_index.md
```

After generating a page with the Hugo CLI:

- remove `draft: true` when ready to publish
- add `weight`
- add `pre` only if sidebar numbering is needed
- create the matching `.vi.md` file

## Pre-Commit Checklist

- [ ] Both English and Vietnamese files created for every new page
- [ ] Front matter present with `title`, `date`, and `weight`
- [ ] `draft` removed from front matter for published pages
- [ ] All image paths use page-relative (`_diagrams/...`) not root-relative paths
- [ ] `hugo server` runs without errors and page renders correctly in both languages
- [ ] Screenshots cropped and optimized (WebP for web, PNG for source)
- [ ] No broken internal links — use `{{< ref >}}` shortcode for internal links
- [ ] Section `pre` prefixes are consistent across language pairs

## Templating Rules

When modifying layouts, partials, or shortcodes:

1. **Do not modify files in `themes/hugo-theme-learn/`** — create overrides in `layouts/` instead
2. Follow Hugo's lookup order: `layouts/` overrides `themes/hugo-theme-learn/layouts/`
3. Match the directory structure exactly (e.g., override `themes/.../layouts/_default/list.html` at `layouts/_default/list.html`)
4. Provide English-language fallbacks in template logic for any localized UI strings

### Existing Override Map

| Override Path | Replaces Theme | Purpose |
|---------------|---------------|---------|
| `layouts/partials/logo.html` | `partials/logo.html` | AWS logo in sidebar |
| `layouts/partials/custom-footer.html` | Empty hook | Google Analytics |
| `layouts/partials/menu-footer.html` | `partials/menu-footer.html` | Dynamic last-updated date + team links |
| `layouts/_default/list.html` | Theme `_default/list.html` | Image path converter for section pages |
| `layouts/shortcodes/children.html` | Theme `children` shortcode | Enhanced sorting, depth, and styling |

## Commit Convention

Use conventional commits:

```
feat(section-name): add bilingual page for XYZ
fix(assets): correct image paths in ABC
docs: update authoring guidelines
```

## Guardrails

- Keep `AGENTS.md` focused on Hugo usage, authoring rules, and repository workflow.
- Do not encode sample content structure, filler content, or legacy template sections here.
- Update this file when repository conventions change.
