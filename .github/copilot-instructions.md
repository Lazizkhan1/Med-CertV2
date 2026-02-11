---
applyTo: "**/*.py"
---

# Copilot Instructions — Med-Cert (DMED Document Viewer)
## Project Overview
Static-site document viewer mimicking the **DMED** (Uzbekistan Unified Medical Information System) portal at `docs.dmed.uz`. Each document is identified by a UUID and protected by a 4-digit PIN. No build tools, frameworks, or backend — pure HTML/CSS/JS served as static files.

## Architecture

### Two-layer page model
| Layer | Directory | Purpose |
|-------|-----------|---------|
| **Access gate** | `access/<uuid>/index.html` | PIN entry screen — validates a hardcoded 4-digit PIN client-side, then redirects to the document |
| **Document view** | `el-docs/<uuid>/index.html` | Rendered medical certificate (A4 layout) with print-to-PDF button |

Each UUID folder pair (`access/` + `el-docs/`) represents one document. The PIN is embedded in the access page's `handleInput` JS function (`if (pin === "XXXX")`), and the same PIN is displayed in the el-doc footer (`#pin-code` span).

### Shared assets
- `assets/logo.svg`, `assets/favicon.ico`, `assets/image.svg` — global branding
- `assets/lang/{en,ru,uz}.svg` — flag icons for language switcher
- `assets/<uuid>.svg` — per-document QR code
- `el-docs/styles.css` — shared stylesheet for all document pages (A4 paper simulation, print styles)

## Adding a New Document

1. Generate a UUID (e.g. `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
2. Choose a 4-digit PIN
3. Copy an existing `access/<uuid>/index.html` into a new folder; update:
   - The PIN comparison string in `handleInput`: `if (pin === "NNNN")`
   - The redirect URL: `window.location.href = "/el-docs/<new-uuid>/index.html"`
4. Copy an existing `el-docs/<uuid>/index.html` into a new folder; update:
   - All patient/medical data in the table cells
   - `#pin-code` span to match the chosen PIN
   - `#uuid` span with the new UUID
   - `#document-date` span
   - QR code `<img src="/assets/<new-uuid>.svg">`
5. Place the QR code SVG at `assets/<new-uuid>.svg`

## Key Conventions

- **No build step** — all files are hand-authored HTML. Styles are inlined in access pages and shared via `el-docs/styles.css` for document pages.
- **Inline styles in el-docs** — document HTML uses verbose inline styles (`box-sizing: border-box; user-select: none`) on every element to guarantee print fidelity. Preserve this pattern.
- **i18n via JS object** — access pages have a `languages` object with `uz`, `ru`, `en` keys. Default language is set via `setLanguage('ru')` at the bottom of the script. Text keys: `heading`, `button`, `error`.
- **Font** — Inter (Google Fonts) for access pages; InterVariable / system sans-serif for document pages.
- **Print layout** — document pages target A4 (`595pt` width, `@page { size: A4; margin: 0 }`). The `#printBtn` is hidden in `@media print`.
- **External API references** — document pages load logos from `https://docs-api.dmed.uz/api/logo` and `https://docs-api.dmed.uz/api/dmed-logo`. These are remote URLs, not local assets.

## File Patterns
```
access/<uuid>/index.html   — self-contained PIN gate (CSS + JS inlined)
el-docs/<uuid>/index.html  — self-contained document (inline styles, shared CSS import, inline JS)
el-docs/styles.css         — base doc-content layout + print media queries
assets/<uuid>.svg          — QR code per document
assets/lang/*.svg          — language flag icons
```

## Gotchas
- Access pages duplicate ~300 lines of identical CSS/JS. When changing shared behavior (language options, styling, PIN UX), update **all** access pages.
- PIN validation is purely client-side; there is no server-side check.
- The `<img>` for QR codes has a stray `</svg>` closing tag after it — this is present in all documents.
