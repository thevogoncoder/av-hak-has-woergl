# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo static site for AV HAK/HAS Wörgl using the **Blowfish** theme (git submodule), with **Decap CMS** for content management, deployed on **Netlify**.

## Commands

### Development
```bash
hugo server --bind 0.0.0.0 --baseURL http://localhost:1313   # Dev server (port 1313)
npx decap-server                                               # CMS local proxy (port 8081)
```

### Build
```bash
hugo --gc --minify          # Production build (output: public/)
```

No npm/yarn dependencies, linting, or test suite.

## Architecture

- **Hugo 0.156.0** generates static HTML from Markdown content
- **Blowfish theme v2.98.0** — git submodule in `themes/blowfish/`, provides all base templates and Tailwind CSS styling
- **Decap CMS 3.0.0** — browser-based editor at `/admin/` (loaded from CDN)
- **Netlify Identity + Git Gateway** — handles CMS authentication and commits
- **Dev Container** — Hugo Extended + Node.js LTS environment

### Key Paths

| Path | Purpose |
|------|---------|
| `config/_default/` | Hugo configuration split across `hugo.toml`, `params.toml`, `languages.en.toml`, `menus.en.toml`, `markup.toml` |
| `content/` | Markdown content (homepage `_index.md`, blog posts in `posts/`) |
| `layouts/partials/` | **Only custom templates** — `extend-head.html` (Netlify Identity widget) and `extend-footer.html` (CMS login redirect) |
| `themes/blowfish/` | Git submodule — all base templates, styles, shortcodes |
| `static/admin/` | Decap CMS interface (`index.html`) and collection config (`config.yml`) |
| `static/img/` | Image uploads managed by CMS |
| `netlify.toml` | Netlify build settings and deploy contexts |

### Theme Customization

The site inherits everything from Blowfish and only overrides two partials:
- `layouts/partials/extend-head.html` — injects Netlify Identity widget script
- `layouts/partials/extend-footer.html` — redirects authenticated users to `/admin/`

Theme configuration (color scheme, homepage layout, dark mode, search, etc.) is controlled via `config/_default/params.toml`.

### Content Model

Posts live in `content/posts/` with slug pattern `YYYY-MM-DD-{slug}.md`. Fields: title, date, draft, body. The CMS collection config is in `static/admin/config.yml`.
