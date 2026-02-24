# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo static site with Decap CMS (formerly Netlify CMS) for content management, deployed on Netlify. Simplified from the original "One-click Hugo CMS" template — webpack, custom CSS, and JS tooling have been removed.

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

No npm/yarn dependencies, linting, or test suite — those were removed from the original template.

## Architecture

- **Hugo 0.156.0** generates static HTML from Markdown content
- **Decap CMS 3.0.0** provides a browser-based editor at `/admin/` (loaded from CDN)
- **Netlify Identity + Git Gateway** handles CMS authentication and commits
- **Dev Container** provides Hugo Extended + Node.js LTS environment

### Key Paths

| Path | Purpose |
|------|---------|
| `content/` | Markdown content (homepage `_index.md`, blog posts in `posts/`) |
| `layouts/` | Hugo templates (`_default/baseof.html` is the base, `index.html` is homepage) |
| `static/admin/` | Decap CMS interface (`index.html`) and config (`config.yml`) |
| `static/img/` | Image uploads managed by CMS |
| `hugo.toml` | Hugo configuration |
| `netlify.toml` | Netlify build settings and deploy contexts |

### Content Model

Posts live in `content/posts/` with slug pattern `YYYY-MM-DD-{slug}.md`. Fields: title, date, draft, body. The CMS collection config is in `static/admin/config.yml`.

### Template Hierarchy

`baseof.html` wraps all pages (includes Netlify Identity widget for CMS login). `single.html` renders individual posts, `list.html` renders archive pages, `index.html` renders the homepage.
