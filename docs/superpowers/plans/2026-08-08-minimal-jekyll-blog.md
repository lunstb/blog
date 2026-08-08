# Minimal Jekyll Blog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship an ultra-minimal Jekyll blog served by native GitHub Pages at https://blog.berkelunstad.com, where publishing = merging a PR with one markdown file.

**Architecture:** Hand-rolled Jekyll layouts (no external theme) built natively by GitHub Pages from the `main` branch — no Actions workflow. Custom domain wired via a `CNAME` file plus a Cloudflare DNS record. RSS via the Pages-whitelisted `jekyll-feed` plugin.

**Tech Stack:** Jekyll 3.x (GitHub Pages native build), jekyll-feed, plain CSS, `gh` CLI for Pages configuration, Cloudflare DNS.

## Global Constraints

- Site URL: `https://blog.berkelunstad.com`; repo: `github.com/lunstb/blog`; Pages source: `main` branch, root.
- No Actions workflows, no external Jekyll theme, no JS, no analytics.
- Only Pages-whitelisted plugins (here: `jekyll-feed` only).
- Header copy exactly: name "Berke Lunstad", subtitle "a blog", links "linkedin" (https://www.linkedin.com/in/berkelunstad/) and "rss" (`/feed.xml`).
- Monospace font stack: `ui-monospace, Menlo, monospace`; dark text (#111) on white.
- Post filenames: `_posts/YYYY-MM-DD-slug.md`; frontmatter needs only `title:` (layout defaulted in `_config.yml`).
- No local Ruby/Jekyll toolchain is assumed on this machine — verification of rendering happens against the live Pages build (Task 2), not a local build.

---

### Task 1: Jekyll site scaffold

**Files:**
- Create: `_config.yml`
- Create: `_layouts/default.html`
- Create: `_layouts/post.html`
- Create: `index.html`
- Create: `assets/style.css`
- Create: `CNAME`
- Create: `_posts/2026-08-08-hello-world.md`

**Interfaces:**
- Consumes: nothing (first task).
- Produces: a complete Jekyll source tree on `main` that Task 2 pushes and builds. Task 2 relies on: `CNAME` containing exactly `blog.berkelunstad.com`, homepage at `/`, post URL `/hello-world/` (from `permalink: /:title/`), feed at `/feed.xml`.

- [ ] **Step 1: Write `_config.yml`**

```yaml
title: Berke Lunstad
url: https://blog.berkelunstad.com
permalink: /:title/
plugins:
  - jekyll-feed
defaults:
  - scope:
      path: ""
      type: posts
    values:
      layout: post
```

- [ ] **Step 2: Write `_layouts/default.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% if page.title %}{{ page.title }} · {{ site.title }}{% else %}{{ site.title }}{% endif %}</title>
  <link rel="stylesheet" href="{{ '/assets/style.css' | relative_url }}">
  {% feed_meta %}
</head>
<body>
  <header>
    <div class="name"><a href="{{ '/' | relative_url }}">Berke Lunstad</a></div>
    <div class="subtitle">a blog · <a href="https://www.linkedin.com/in/berkelunstad/">linkedin</a> · <a href="{{ '/feed.xml' | relative_url }}">rss</a></div>
  </header>
  <main>
    {{ content }}
  </main>
</body>
</html>
```

- [ ] **Step 3: Write `_layouts/post.html`**

```html
---
layout: default
---
<article>
  <h1>{{ page.title }}</h1>
  <p class="post-date">{{ page.date | date: "%Y-%m-%d" }}</p>
  {{ content }}
  <p class="back"><a href="{{ '/' | relative_url }}">← back</a></p>
</article>
```

- [ ] **Step 4: Write `index.html`**

```html
---
layout: default
---
<ul class="post-list">
  {% for post in site.posts %}
  <li><span class="date">{{ post.date | date: "%Y-%m-%d" }}</span> <a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
```

- [ ] **Step 5: Write `assets/style.css`**

```css
body {
  font-family: ui-monospace, Menlo, monospace;
  font-size: 14px;
  line-height: 1.7;
  color: #111;
  background: #fff;
  max-width: 42rem;
  margin: 0 auto;
  padding: 2.5rem 1.25rem;
}

header { margin-bottom: 2rem; }
header .name { font-weight: 700; font-size: 16px; }
header .name a { color: #111; text-decoration: none; }
header .subtitle { color: #666; }
header .subtitle a { color: #666; }

a { color: #111; }

.post-list { list-style: none; padding: 0; margin: 0; }
.post-list li { margin-bottom: 0.4rem; }
.post-list .date { color: #888; margin-right: 0.5rem; }

article h1 { font-size: 18px; margin-bottom: 0.25rem; }
.post-date { color: #888; margin-top: 0; }
article img { max-width: 100%; }
pre { overflow-x: auto; background: #f6f6f6; padding: 0.75rem; }
code { background: #f6f6f6; }
blockquote { border-left: 3px solid #ddd; margin-left: 0; padding-left: 1rem; color: #444; }
.back { margin-top: 2rem; }
```

- [ ] **Step 6: Write `CNAME`**

File contains exactly one line, no trailing whitespace:

```
blog.berkelunstad.com
```

- [ ] **Step 7: Write `_posts/2026-08-08-hello-world.md`**

```markdown
---
title: Hello, world
---

This blog now exists. Posts are markdown files merged via pull request —
nothing fancier than that.
```

- [ ] **Step 8: Static verification (no local Jekyll available)**

Run each check; all must pass:

```bash
# CNAME is exactly the domain
[ "$(cat CNAME)" = "blog.berkelunstad.com" ] && echo CNAME-OK

# Every {% ... %} / {{ ... }} tag is balanced in each html file
grep -c '{%' _layouts/default.html index.html _layouts/post.html
grep -c '%}' _layouts/default.html index.html _layouts/post.html
# counts must match per file

# post frontmatter parses: starts and ends with ---
head -1 _posts/2026-08-08-hello-world.md   # expect: ---
```

Expected: `CNAME-OK`, matching tag counts, `---` as first line of the post.

- [ ] **Step 9: Commit**

```bash
git add _config.yml _layouts/ index.html assets/ CNAME _posts/
git commit -m "feat: add minimal Jekyll site scaffold"
```

---

### Task 2: Deploy to GitHub Pages

**Files:**
- Modify: none (uses Task 1's tree; operations are `git push` + `gh api`)

**Interfaces:**
- Consumes: complete Jekyll tree on `main` from Task 1 (`CNAME` = `blog.berkelunstad.com`, post at `/hello-world/`, feed at `/feed.xml`).
- Produces: a live Pages site (GitHub-hosted at `lunstb.github.io/blog` origin) with custom domain `blog.berkelunstad.com` registered on the Pages settings. Task 3 relies on the Pages site existing and the custom domain being set so GitHub can provision the TLS cert once DNS resolves.

- [ ] **Step 1: Push main**

```bash
git push origin main
```

- [ ] **Step 2: Enable GitHub Pages on main branch root**

```bash
gh api -X POST repos/lunstb/blog/pages \
  -f "source[branch]=main" -f "source[path]=/" \
  || gh api -X PUT repos/lunstb/blog/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

Expected: JSON response including `"status"` (POST succeeds on first enable; PUT covers the already-enabled case).

- [ ] **Step 3: Wait for the build to succeed**

```bash
for i in $(seq 1 12); do
  status=$(gh api repos/lunstb/blog/pages/builds/latest --jq .status)
  echo "$status"; [ "$status" = "built" ] && break; sleep 10
done
```

Expected: final line `built`. If `errored`, fetch the error: `gh api repos/lunstb/blog/pages/builds/latest --jq .error.message` — fix and re-push before continuing.

- [ ] **Step 4: Verify custom domain registered from CNAME file**

```bash
gh api repos/lunstb/blog/pages --jq .cname
```

Expected: `blog.berkelunstad.com`. If null, set it explicitly:

```bash
gh api -X PUT repos/lunstb/blog/pages -f cname=blog.berkelunstad.com
```

- [ ] **Step 5: Verify the site serves from the GitHub origin**

```bash
curl -sI https://lunstb.github.io/blog/ | head -1
```

Expected: `HTTP/2 301` (redirect toward the custom domain — proves content is live and the domain is registered). No commit this task; it's all remote state.

---

### Task 3: Cloudflare DNS + HTTPS

**Files:**
- Modify: none (Cloudflare dashboard + `gh api` operations)

**Interfaces:**
- Consumes: live Pages site with custom domain set (Task 2).
- Produces: `https://blog.berkelunstad.com` resolving with valid TLS and HTTPS enforced. Nothing downstream.

- [ ] **Step 1: User adds the DNS record in Cloudflare (manual)**

Ask the user to add, in the Cloudflare dashboard for `berkelunstad.com` → DNS:

- Type: `CNAME`, Name: `blog`, Target: `lunstb.github.io`, Proxy status: **DNS only** (grey cloud), TTL: Auto.

Grey cloud is required until GitHub provisions the certificate; it can stay grey permanently (GitHub serves HTTPS itself).

- [ ] **Step 2: Verify DNS resolves**

```bash
dig +short blog.berkelunstad.com CNAME @1.1.1.1
```

Expected: `lunstb.github.io.` — retry after a minute if empty (propagation).

- [ ] **Step 3: Wait for GitHub's TLS certificate**

```bash
for i in $(seq 1 30); do
  state=$(gh api repos/lunstb/blog/pages --jq .https_certificate.state)
  echo "$state"; [ "$state" = "approved" ] && break; sleep 30
done
```

Expected: reaches `approved` (can take up to ~15 min; states progress `new` → `authorization_created` → … → `approved`). If stuck >30 min, re-trigger by clearing and re-setting the domain: `gh api -X PUT repos/lunstb/blog/pages -f cname=blog.berkelunstad.com`.

- [ ] **Step 4: Enforce HTTPS**

```bash
gh api -X PUT repos/lunstb/blog/pages -F https_enforced=true
gh api repos/lunstb/blog/pages --jq .https_enforced
```

Expected: `true`.

- [ ] **Step 5: End-to-end verification**

```bash
curl -sI https://blog.berkelunstad.com/ | head -1          # expect HTTP/2 200
curl -s  https://blog.berkelunstad.com/ | grep -c "Hello, world"   # expect >= 1
curl -sI https://blog.berkelunstad.com/hello-world/ | head -1      # expect HTTP/2 200
curl -s  https://blog.berkelunstad.com/feed.xml | head -2          # expect XML feed header
curl -sI http://blog.berkelunstad.com/ | head -1           # expect 301 (HTTPS enforced)
```

All five must match expectations. Done.
