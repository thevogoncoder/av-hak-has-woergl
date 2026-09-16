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

The site inherits everything from Blowfish and overrides the following. Keep this list
accurate — it is the blast radius to re-check on every theme upgrade.

**Additive extension points** (Blowfish ships no version of these, so they never conflict):

- `partials/extend-head.html` — injects the Netlify Identity widget script
- `partials/extend-footer.html` — redirects authenticated users to `/admin/`
- `partials/homepage-cta.html` — "Hermes aktuell" membership call-to-action card
- `partials/homepage-sponsors.html` — sponsor banner with light/dark logo variants

**Real overrides of theme files** (these shadow an upstream file and must be rebased when
the upstream copy changes):

| Override | Upstream base | Our delta |
|----------|---------------|-----------|
| `partials/recent-articles/main.html` | same path | appends the `homepage-cta` + `homepage-sponsors` partials |
| `partials/recent-articles/list.html` | same path | filters out posts whose `hideFromHomeAfter` has passed |
| `shortcodes/carousel.html` | same path | adds the `maxSize` param → `images.AutoOrient` + fit/WebP processing |
| `shortcodes/gallery.html` | same path | adds the `maxSize` param → `images.AutoOrient` + fit/WebP processing |

Theme configuration (color scheme, homepage layout, dark mode, search, etc.) is controlled via `config/_default/params.toml`.

### Upgrading the theme

Blowfish pins a supported Hugo window in its `config.toml` (`module.hugoVersion` min/max).
Keep all three in sync or the build warns and may break:

1. `themes/blowfish` submodule tag
2. `HUGO_VERSION` in `netlify.toml`
3. `ghcr.io/devcontainers/features/hugo` version in `.devcontainer/devcontainer.json`

To rebase an override, 3-way merge it rather than hand-patching — this reliably keeps our
delta and picks up upstream's changes:

```bash
git show <old-tag>:layouts/shortcodes/gallery.html > /tmp/base.html   # run in themes/blowfish
git show <new-tag>:layouts/shortcodes/gallery.html > /tmp/new.html
git merge-file -p layouts/shortcodes/gallery.html /tmp/base.html /tmp/new.html > merged.html
```

**Stage the submodule pointer** (`git add themes/blowfish`) after checking out a new tag.
Netlify builds run `git submodule update --init --recursive`, which resets the submodule to
the commit recorded in the parent repo — an unstaged upgrade silently builds the old theme.

### Content Model

**All content is in German.** Default language is `de`.

Two content patterns are used:

1. **Page bundles** (directory with `index.md` + co-located assets): used for blog posts and photo galleries. Post slugs follow `YYYY-MM-DD-{slug}/index.md`. Images, QR codes, etc. live alongside the `index.md`.
2. **Standalone pages** (single `.md` file): used for `kontakt.md`, `impressum.md`, `links.md`.

Pages that should not appear in listing/taxonomy use `_build: list: never` in front matter.

Use `<!--more-->` as the summary divider in posts.

Photo galleries use the Blowfish `{{</* gallery */>}}` shortcode with `<img>` tags and `class="grid-wNN"` for layout.

The CMS collection config is in `static/admin/config.yml` — update this when adding new content fields.

## Post Types

There are two kinds of posts published on the website: **events** and **general posts**.

### All Posts

- Always use `heroStyle: "background"` in the front matter unless otherwise instructed.
- If a single image is provided, use it as the `featured` image for the post.
- If multiple images are provided, use the first as `featured` and add a `{{</* carousel */>}}` shortcode at the end of the post (unless otherwise instructed).
- Convert every image used in a post to WebP before adding it to the page bundle — fit within 1200x1200, quality 82, EXIF auto-oriented, metadata stripped. Commit only the `.webp`, not the original JPEG/PNG:
  ```bash
  magick <source> -auto-orient -resize 1200x1200\> -quality 82 -define webp:method=6 -strip <name>.webp
  ```
- Use `<!--more-->` as the summary divider.
- When a reference text or document is given, use the provided text as closely as possible (fixing typos is fine), but **do not** replicate its styling — fit the content into the site's Markdown/shortcode conventions instead.

### Events

- Add the tag `Veranstaltung` to all event posts.
- Add `hideFromHomeAfter: <event_date + 1 week>` to the front matter (unless otherwise instructed).
- If the event has an itinerary, consider using the `timeline` shortcode to display it.
- Add a registration form at the end of the post if appropriate (e.g. the event requires sign-up). When adding a form:
  - Follow the Netlify Forms patterns (honeypot + reCAPTCHA, see below).
  - **Also create an email notification** for that specific form on Netlify (see Form Notifications below):
    - Recipient: `sekretariat@hak-woergl.at` (unless otherwise instructed)
    - Subject: `[AV] Neue Anmeldung für %{formName}` (unless otherwise instructed)

### General Posts

No additional requirements beyond the common rules above.

## Netlify-Specific Patterns

### Forms
Netlify detects forms at deploy time via the `data-netlify="true"` attribute. Two patterns are used:

- **Honeypot spam protection**: Add `netlify-honeypot="bot-field"` to the `<form>` tag and include a hidden field `<input name="bot-field" />`.
- **reCAPTCHA**: Add `<div data-netlify-recaptcha="true"></div>` inside the form for built-in captcha.

See `content/mitgliederanmeldung.md` for a full worked example with both honeypot and reCAPTCHA.
Note it is currently commented out — that page now embeds a Vereinsplaner iframe instead — so it
serves as a reference template, not a live form.

### Form Notifications

Email notifications per form are managed via the Netlify API (not exposed in the MCP server).

- **Site:** `whimsical-quokka-302475` — <https://av-hakhas-woergl.at>
- **Site ID:** `10744c26-5ad0-45f4-b062-5c744d395b4f`

#### Auth token

Stored in `~/.config/netlify/config.json` after `netlify login`. The `users` map can hold
**several entries** — `netlify logout` leaves stale ones behind and `userId` is often `null`.
Do not read the first entry blindly; select by email. `netlify status` resolves the active
user correctly and is the quickest cross-check.

```bash
TOKEN=$(python3 -c "
import json, os
c = json.load(open(os.path.expanduser('~/.config/netlify/config.json')))
print(next(u['auth']['token'] for u in c['users'].values()
           if u.get('email') == '<login-email>'))
")
```

#### Form IDs

Via the MCP `get-forms-for-project` tool, or directly:

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://api.netlify.com/api/v1/sites/10744c26-5ad0-45f4-b062-5c744d395b4f/forms"
```

#### Create the notification

```bash
curl -s -X POST "https://api.netlify.com/api/v1/hooks" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "site_id": "10744c26-5ad0-45f4-b062-5c744d395b4f",
    "form_id": "<form-id>",
    "type": "email",
    "event": "submission_created",
    "data": {
      "email": "<recipient>",
      "subject_template": "[AV] Neue Anmeldung für %{formName}"
    }
  }'
```

`form_id` must be at the top level (not inside `data`) for per-form scoping to work.
The subject field is named `subject_template` (not `subject`) — it is easy to read back as
missing if you check the wrong key. It can also be set in the Netlify UI.

A site-wide `submission_created` hook to `kontakt@av-hakhas-woergl.at` already exists, so that
address receives a copy of every form's submissions in addition to any per-form notification.

#### Diagnosing auth failures

- `401` on `/forms` — no or invalid token.
- `404` on `/forms` — authenticated, but the account has no access to the site.
- `GET /sites/<id>` returns public metadata **without** auth, so a successful response there
  proves nothing about the token. Verify with `/user` or `/forms`.

### Image Processing

The `carousel` and `gallery` shortcodes in `layouts/shortcodes/` override the Blowfish originals to add Hugo image processing. All images are auto-oriented (EXIF), resized to fit 1200x1200 max, and converted to WebP at build time. Processed images are cached in `resources/_gen/`.
