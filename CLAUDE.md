# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo static site for **Absolventenverein HAK/HAS Wörgl** (German-language alumni association) using the **Blowfish** theme, with **Decap CMS** for content management, deployed on **Netlify**.

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

- **Hugo** generates static HTML from Markdown content
- **Blowfish theme** — git submodule in `themes/blowfish/`, provides all base templates and Tailwind CSS styling
- **Decap CMS 3.0.0** — browser-based editor at `/admin/` (loaded from CDN)
- **Netlify Identity + Git Gateway** — handles CMS authentication and commits
- **Goldmark** with `unsafe = true` — raw HTML in Markdown is rendered (used for forms, iframes, custom markup)
- **Dev Container** — Hugo Extended + Node.js LTS environment (ports 1313 + 8081 forwarded)

### Key Paths

| Path | Purpose |
|------|---------|
| `config/_default/` | Hugo config: `hugo.toml`, `params.toml`, `languages.de.toml`, `menus.de.toml`, `markup.toml` |
| `content/` | Markdown content — homepage, blog posts, standalone pages |
| `layouts/partials/` | Custom template overrides (see Theme Customization below) |
| `themes/blowfish/` | Git submodule — **do not edit directly** |
| `static/admin/` | Decap CMS interface (`index.html`) and collection config (`config.yml`) |
| `static/img/` | Shared images (logo, background, link logos) |
| `netlify.toml` | Build settings; deploy preview uses `--buildFuture` flag |

### Theme Customization

The site inherits everything from Blowfish and overrides these partials in `layouts/partials/`:

- `extend-head.html` — injects Netlify Identity widget script + custom CSS (e.g. `.qr-code` border)
- `extend-footer.html` — redirects authenticated users to `/admin/`
- `recent-articles/main.html` — overrides Blowfish's recent articles partial to inject the CTA block
- `homepage-cta.html` — "Hermes aktuell" membership call-to-action card on the homepage

Theme configuration (color scheme, homepage layout, dark mode, search, etc.) is controlled via `config/_default/params.toml`.

### Content Model

**All content is in German.** Default language is `de`.

Two content patterns are used:

1. **Page bundles** (directory with `index.md` + co-located assets): used for blog posts and photo galleries. Post slugs follow `YYYY-MM-DD-{slug}/index.md`. Images, QR codes, etc. live alongside the `index.md`.
2. **Standalone pages** (single `.md` file): used for `kontakt.md`, `impressum.md`, `links.md`.

Pages that should not appear in listing/taxonomy use `_build: list: never` in front matter.

Use `<!--more-->` as the summary divider in posts.

Photo galleries use the Blowfish `{{</* gallery */>}}` shortcode with `<img>` tags and `class="grid-wNN"` for layout.

The CMS collection config is in `static/admin/config.yml` — update this when adding new content fields.

## Netlify-Specific Patterns

### Forms
Netlify detects forms at deploy time via the `data-netlify="true"` attribute. Two patterns are used:

- **Honeypot spam protection**: Add `netlify-honeypot="bot-field"` to the `<form>` tag and include a hidden field `<input name="bot-field" />`.
- **reCAPTCHA**: Add `<div data-netlify-recaptcha="true"></div>` inside the form for built-in captcha.

See `content/mitgliederanmeldung/index.md` for the full working example with both honeypot and reCAPTCHA.

### Image Processing

The `carousel` and `gallery` shortcodes in `layouts/shortcodes/` override the Blowfish originals to add Hugo image processing. All images are auto-oriented (EXIF), resized to fit 1200x1200 max, and converted to WebP at build time. Processed images are cached in `resources/_gen/`.
