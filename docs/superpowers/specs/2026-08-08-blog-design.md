# Personal Blog — Design Spec

**Date:** 2026-08-08
**URL:** https://blog.berkelunstad.com
**Repo:** github.com/lunstb/blog

## Goal

A personal blog with an ultra-minimal, text-only design. Publishing a post means
merging a PR that adds one markdown file. No CI to maintain, no JS, no framework
dependencies.

## Stack

- **Jekyll**, built natively by GitHub Pages (no Actions workflow).
- **Hand-rolled layouts** — no external theme. The ultra-minimal aesthetic
  (monospace type, black on white, plain underlined links) is two small layout
  files and ~40 lines of CSS we own outright.
- **jekyll-feed** plugin (GitHub Pages whitelisted) for RSS at `/feed.xml`.
- Native Pages builds use Jekyll 3.x. Accepted tradeoff: irrelevant at this
  scale; can migrate to an Actions build later if ever needed.

## Repository structure

```
_config.yml            # title, url, plugins: [jekyll-feed], permalink style
_layouts/default.html  # html shell + header (name, subtitle, LinkedIn link)
_layouts/post.html     # wraps default: title, date, content, back link
index.html             # reverse-chronological post list: date + linked title
assets/style.css       # monospace styling, ~40 lines
_posts/                # posts named YYYY-MM-DD-slug.md
CNAME                  # contains: blog.berkelunstad.com
```

## Design

Ultra-minimal "just text" direction (selected from mockups):

- Monospace font stack (`ui-monospace, Menlo, monospace`), dark text on white.
- **Header:** "Berke Lunstad" bold, muted subtitle, small LinkedIn link on the
  header line.
- **Homepage:** list of posts, each `YYYY-MM-DD  <underlined title>`, newest
  first. Nothing else.
- **Post page:** title, date, body, back link to home.
- No JS, no analytics, no images required by the design.

## Publishing flow

1. Open a PR adding `_posts/YYYY-MM-DD-slug.md` with frontmatter:
   ```yaml
   ---
   title: My Post Title
   ---
   ```
   (Layout defaulted via `_config.yml`; date comes from the filename.)
2. Merge to `main`. GitHub Pages rebuilds automatically (~1 min).

## DNS / hosting setup

1. Repo settings → Pages: build from `main` branch, custom domain
   `blog.berkelunstad.com`, enforce HTTPS once cert provisions.
2. Cloudflare: `CNAME` record `blog` → `lunstb.github.io`, **DNS only**
   (grey cloud) at least until GitHub's TLS cert is provisioned.

## Testing

- Local preview optional via `jekyll serve` (not required for publishing).
- Verification after deploy: homepage lists a sample post, post page renders,
  `/feed.xml` valid, custom domain resolves with HTTPS.

## Out of scope (YAGNI)

Tags/categories, search, comments, analytics, dark mode, pagination, about page.
Any of these can be added later without restructuring.
