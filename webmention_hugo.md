# Complete Guide: Adding Webmentions to Hugo with GitLab CI/CD

**Version:** 1.0
**Last Updated:** 2025-12-20
**Estimated Implementation Time:** 4-6 hours
**Difficulty:** Intermediate

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Architecture Overview](#architecture-overview)
4. [Phase 1: Webmention.io Setup](#phase-1-webmentionio-setup)
5. [Phase 2: Add Microformats to Hugo](#phase-2-add-microformats-to-hugo)
6. [Phase 3: Display Webmentions](#phase-3-display-webmentions)
7. [Phase 4: GitLab CI/CD Integration](#phase-4-gitlab-cicd-integration)
8. [Phase 5: Bridgy Fed Integration](#phase-5-bridgy-fed-integration)
9. [Phase 6: Styling and Polish](#phase-6-styling-and-polish)
10. [Testing and Validation](#testing-and-validation)
11. [Troubleshooting](#troubleshooting)
12. [Optional Enhancements](#optional-enhancements)
13. [Maintenance](#maintenance)

---

## Introduction

This guide will help you add **Webmentions** to your existing Hugo blog, enabling:
- ✅ Comments from other blogs that support webmentions
- ✅ Mastodon/Fediverse comments (via Bridgy Fed)
- ✅ Twitter mentions (via Brid.gy)
- ✅ Reddit mentions (via Brid.gy)
- ✅ Likes, reposts, and bookmarks
- ✅ Build-time rendering (SEO-friendly)
- ✅ Zero cost forever

### What You'll Build

```
Your Hugo Blog
    ↓
Add Webmention Endpoint (webmention.io)
    ↓
Add Microformats (h-entry, h-card)
    ↓
GitLab CI/CD Fetches Webmentions Daily
    ↓
Hugo Renders Comments as Static HTML
    ↓
Deploy with Comments Included
```

---

## Prerequisites

### Required
- [ ] Existing Hugo blog in GitLab repository
- [ ] Hugo version 0.100.0+ (for resources.GetRemote)
- [ ] GitLab CI/CD pipeline configured
- [ ] Custom domain (e.g., yourdomain.com)
- [ ] Basic knowledge of Hugo templates
- [ ] Basic Git/GitLab knowledge

### Recommended
- [ ] Code editor (VS Code, Sublime, etc.)
- [ ] Local Hugo development environment
- [ ] Understanding of HTML/CSS

### Verify Your Setup

```bash
# Check Hugo version
hugo version
# Should be 0.100.0 or higher

# Test local Hugo server
hugo server -D
# Should start without errors

# Check GitLab CI/CD
# Visit: https://gitlab.com/your-username/your-repo/-/pipelines
# Ensure pipeline exists and runs
```

---

## Architecture Overview

### How It Works

```
┌─────────────────────────────────────────────────────┐
│  1. PUBLISH POST                                     │
│     Your Hugo blog has h-entry markup                │
│     <link rel="webmention"> points to endpoint       │
└───────────────┬─────────────────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────────────────┐
│  2. SOMEONE MENTIONS YOUR POST                       │
│     ├─ Mastodon user replies to URL (via Bridgy Fed)│
│     ├─ Another blog sends webmention                │
│     ├─ Twitter mention (via Brid.gy)                │
│     └─ Reddit link (via Brid.gy)                    │
└───────────────┬─────────────────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────────────────┐
│  3. WEBMENTION RECEIVED                              │
│     webmention.io endpoint receives POST             │
│     Verifies source links to target                  │
│     Parses microformats from source                  │
│     Stores mention data                              │
└───────────────┬─────────────────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────────────────┐
│  4. GITLAB CI/CD SCHEDULED JOB                       │
│     Runs daily (or on-demand)                        │
│     Fetches all webmentions from API                 │
│     Saves to data/webmentions.json                   │
│     Commits back to repository                       │
└───────────────┬─────────────────────────────────────┘
                │
                ↓
┌─────────────────────────────────────────────────────┐
│  5. HUGO BUILD                                       │
│     Reads data/webmentions.json                      │
│     Matches mentions to posts by URL                 │
│     Renders as static HTML                           │
│     Deploys with comments included                   │
└─────────────────────────────────────────────────────┘
```

### Data Flow

```json
// webmention.io stores this:
{
  "type": "entry",
  "author": {
    "name": "Alice",
    "photo": "https://example.com/avatar.jpg",
    "url": "https://example.com/alice"
  },
  "url": "https://example.com/alice/reply",
  "published": "2025-12-20T10:00:00Z",
  "wm-received": "2025-12-20T10:05:00Z",
  "wm-id": 123456,
  "wm-source": "https://example.com/alice/reply",
  "wm-target": "https://yourdomain.com/posts/my-post/",
  "wm-property": "in-reply-to",
  "content": {
    "html": "<p>Great post!</p>",
    "text": "Great post!"
  }
}
```

---

## Phase 1: Webmention.io Setup

### Step 1.1: Create webmention.io Account

1. **Visit webmention.io**
   ```
   https://webmention.io/
   ```

2. **Sign in with your domain**
   - Click "Sign in with your domain"
   - Enter your domain: `https://yourdomain.com`
   - You'll be redirected to authenticate using IndieAuth

3. **Set up IndieAuth**

   You need ONE of these on your homepage:

   **Option A: GitHub (Recommended)**

   Add to your Hugo site's `config.toml` or `config.yaml`:

   ```toml
   # config.toml
   [params]
     github = "yourusername"
   ```

   Then in `layouts/partials/head.html` or homepage:
   ```html
   <link rel="me" href="https://github.com/yourusername" />
   ```

   **Option B: Email**
   ```html
   <link rel="me" href="mailto:you@yourdomain.com" />
   ```

   **Option C: PGP Key**
   ```html
   <link rel="me" href="https://keybase.io/yourusername" />
   ```

4. **Deploy and Verify**
   ```bash
   # Commit the rel="me" link
   git add layouts/partials/head.html config.toml
   git commit -m "Add IndieAuth rel=me link"
   git push

   # Wait for GitLab CI/CD to deploy
   ```

5. **Complete Authentication**
   - Return to webmention.io
   - Enter your domain again
   - Click "Sign in"
   - Authorize via GitHub (or your chosen method)

6. **Get Your Endpoints**

   After authentication, you'll see:
   ```html
   <link rel="webmention" href="https://webmention.io/yourdomain.com/webmention" />
   <link rel="pingback" href="https://webmention.io/yourdomain.com/xmlrpc" />
   ```

   **Save these!** You'll need them in the next step.

### Step 1.2: Get API Token

1. **Visit your webmention.io settings**
   ```
   https://webmention.io/settings
   ```

2. **Find your API Key**
   - Look for "API Key" section
   - Copy your token (looks like: `abc123def456...`)

3. **Save as GitLab CI/CD Variable**
   - Go to your GitLab project
   - Settings → CI/CD → Variables
   - Click "Add Variable"
   - Key: `WEBMENTION_TOKEN`
   - Value: `your-api-token-here`
   - Check: "Mask variable" ✓
   - Check: "Protect variable" (optional)
   - Click "Add variable"

---

## Phase 2: Add Microformats to Hugo

### Step 2.1: Update Site Configuration

Edit your `config.toml` or `config.yaml`:

**config.toml:**
```toml
[params]
  domain = "yourdomain.com"
  author = "Your Name"
  author_url = "https://yourdomain.com/about/"
  author_photo = "https://yourdomain.com/images/avatar.jpg"
  author_email = "you@yourdomain.com"
  webmention_endpoint = "https://webmention.io/yourdomain.com/webmention"
  pingback_endpoint = "https://webmention.io/yourdomain.com/xmlrpc"
```

**config.yaml:**
```yaml
params:
  domain: yourdomain.com
  author: Your Name
  author_url: https://yourdomain.com/about/
  author_photo: https://yourdomain.com/images/avatar.jpg
  author_email: you@yourdomain.com
  webmention_endpoint: https://webmention.io/yourdomain.com/webmention
  pingback_endpoint: https://webmention.io/yourdomain.com/xmlrpc
```

### Step 2.2: Add Webmention Links to Head

**Option A: If you have `layouts/partials/head.html`**

Edit `layouts/partials/head.html` and add:

```html
{{/* Webmention endpoints */}}
{{ with .Site.Params.webmention_endpoint }}
<link rel="webmention" href="{{ . }}" />
{{ end }}
{{ with .Site.Params.pingback_endpoint }}
<link rel="pingback" href="{{ . }}" />
{{ end }}

{{/* IndieAuth */}}
{{ with .Site.Params.github }}
<link rel="me" href="https://github.com/{{ . }}" />
{{ end }}
{{ with .Site.Params.author_email }}
<link rel="me" href="mailto:{{ . }}" />
{{ end }}
```

**Option B: If you need to create the partial**

1. Create `layouts/partials/head.html`:

```html
{{/* Webmention and IndieAuth links */}}
<link rel="webmention" href="{{ .Site.Params.webmention_endpoint }}" />
<link rel="pingback" href="{{ .Site.Params.pingback_endpoint }}" />
<link rel="me" href="https://github.com/{{ .Site.Params.github }}" />
```

2. Include it in your base template `layouts/_default/baseof.html`:

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ .Title }}</title>

  {{/* Add this line */}}
  {{ partial "head.html" . }}
</head>
```

### Step 2.3: Add h-card (Author Info)

Create or edit `layouts/partials/author-hcard.html`:

```html
<div class="h-card" style="display:none;">
  <a class="u-url u-uid" href="{{ .Site.Params.author_url | default .Site.BaseURL }}">
    <img class="u-photo" src="{{ .Site.Params.author_photo }}" alt="{{ .Site.Params.author }}" />
    <span class="p-name">{{ .Site.Params.author }}</span>
  </a>
  {{ with .Site.Params.author_email }}
    <a class="u-email" href="mailto:{{ . }}">{{ . }}</a>
  {{ end }}
  {{ with .Site.Params.bio }}
    <span class="p-note">{{ . }}</span>
  {{ end }}
</div>
```

Include in your homepage or footer:

```html
{{/* In layouts/index.html or layouts/partials/footer.html */}}
{{ partial "author-hcard.html" . }}
```

### Step 2.4: Add h-entry to Blog Posts

Edit `layouts/_default/single.html` (or your post template):

**Before:**
```html
<article>
  <h1>{{ .Title }}</h1>
  <time>{{ .Date.Format "January 2, 2006" }}</time>
  {{ .Content }}
</article>
```

**After:**
```html
<article class="h-entry">
  {{/* Post Title */}}
  <h1 class="p-name">{{ .Title }}</h1>

  {{/* Post Metadata */}}
  <div class="post-meta">
    <a class="u-url" href="{{ .Permalink }}">
      <time class="dt-published" datetime="{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}">
        {{ .Date.Format "January 2, 2006" }}
      </time>
    </a>

    {{/* Author */}}
    {{ with .Site.Params.author }}
      by
      <a class="p-author h-card" href="{{ $.Site.Params.author_url | default $.Site.BaseURL }}">
        {{ . }}
      </a>
    {{ end }}

    {{/* Categories/Tags */}}
    {{ with .Params.categories }}
      <span class="post-categories">
        {{ range . }}
          <a class="p-category" href="{{ "/categories/" | relURL }}{{ . | urlize }}">{{ . }}</a>
        {{ end }}
      </span>
    {{ end }}
  </div>

  {{/* Post Content */}}
  <div class="e-content">
    {{ .Content }}
  </div>

  {{/* Summary for syndication */}}
  {{ with .Summary }}
    <div class="p-summary" style="display:none;">{{ . }}</div>
  {{ end }}
</article>
```

### Step 2.5: Add h-entry to List Pages (Optional)

Edit `layouts/_default/list.html` or `layouts/index.html`:

```html
{{ range .Pages }}
<article class="h-entry">
  <h2 class="p-name">
    <a class="u-url" href="{{ .Permalink }}">{{ .Title }}</a>
  </h2>

  <time class="dt-published" datetime="{{ .Date.Format "2006-01-02T15:04:05Z07:00" }}">
    {{ .Date.Format "January 2, 2006" }}
  </time>

  {{ with .Summary }}
    <div class="p-summary">{{ . }}</div>
  {{ end }}

  <a href="{{ .Permalink }}">Read more...</a>
</article>
{{ end }}
```

### Step 2.6: Verify Microformats

1. **Build and deploy**
   ```bash
   hugo
   git add .
   git commit -m "Add microformats support"
   git push
   ```

2. **Wait for deployment**

3. **Validate at microformats.io**
   - Visit: https://microformats.io/
   - Enter a blog post URL
   - Should see h-entry, h-card parsed correctly

4. **Validate at IndieWebify.me**
   - Visit: https://indiewebify.me/validate-h-entry/
   - Enter a blog post URL
   - Check for errors

---

## Phase 3: Display Webmentions

### Step 3.1: Create Webmentions Partial

Create `layouts/partials/webmentions.html`:

```html
{{/*
  Webmentions Partial
  Displays webmentions (comments, likes, reposts) for the current page
*/}}

{{ $permalink := .Permalink }}

{{/* Check if we have webmentions data file */}}
{{ if fileExists "data/webmentions.json" }}
  {{ $allMentions := index site.Data "webmentions" }}

  {{/* Filter mentions for this page */}}
  {{ $pageMentions := slice }}
  {{ with $allMentions.children }}
    {{ range . }}
      {{ if eq (strings.TrimSuffix "/" .wm-target) (strings.TrimSuffix "/" $permalink) }}
        {{ $pageMentions = $pageMentions | append . }}
      {{ end }}
    {{ end }}
  {{ end }}

  {{ if gt (len $pageMentions) 0 }}
    {{/* Categorize mentions by type */}}
    {{ $replies := slice }}
    {{ $likes := slice }}
    {{ $reposts := slice }}
    {{ $bookmarks := slice }}
    {{ $mentions := slice }}

    {{ range $pageMentions }}
      {{ if eq .wm-property "in-reply-to" }}
        {{ $replies = $replies | append . }}
      {{ else if eq .wm-property "like-of" }}
        {{ $likes = $likes | append . }}
      {{ else if eq .wm-property "repost-of" }}
        {{ $reposts = $reposts | append . }}
      {{ else if eq .wm-property "bookmark-of" }}
        {{ $bookmarks = $bookmarks | append . }}
      {{ else }}
        {{ $mentions = $mentions | append . }}
      {{ end }}
    {{ end }}

    <section class="webmentions" id="webmentions">
      <h2>Webmentions</h2>
      <p class="webmentions-info">
        <a href="https://indieweb.org/Webmention">What's this?</a>
      </p>

      {{/* Display Likes */}}
      {{ if gt (len $likes) 0 }}
        <div class="webmention-likes">
          <h3>❤️ Liked by {{ len $likes }} {{ if eq (len $likes) 1 }}person{{ else }}people{{ end }}</h3>
          <div class="like-avatars">
            {{ range first 20 $likes }}
              {{ with .author }}
                <a href="{{ .url }}" class="like-avatar" title="{{ .name }}" rel="nofollow ugc">
                  {{ if .photo }}
                    <img src="{{ .photo }}" alt="{{ .name }}" loading="lazy" />
                  {{ else }}
                    <span class="avatar-placeholder">{{ substr .name 0 1 }}</span>
                  {{ end }}
                </a>
              {{ end }}
            {{ end }}
            {{ if gt (len $likes) 20 }}
              <span class="more-count">+{{ sub (len $likes) 20 }} more</span>
            {{ end }}
          </div>
        </div>
      {{ end }}

      {{/* Display Reposts */}}
      {{ if gt (len $reposts) 0 }}
        <div class="webmention-reposts">
          <h3>🔁 Reposted by {{ len $reposts }} {{ if eq (len $reposts) 1 }}person{{ else }}people{{ end }}</h3>
          <div class="repost-avatars">
            {{ range first 20 $reposts }}
              {{ with .author }}
                <a href="{{ .url }}" class="repost-avatar" title="{{ .name }}" rel="nofollow ugc">
                  {{ if .photo }}
                    <img src="{{ .photo }}" alt="{{ .name }}" loading="lazy" />
                  {{ else }}
                    <span class="avatar-placeholder">{{ substr .name 0 1 }}</span>
                  {{ end }}
                </a>
              {{ end }}
            {{ end }}
            {{ if gt (len $reposts) 20 }}
              <span class="more-count">+{{ sub (len $reposts) 20 }} more</span>
            {{ end }}
          </div>
        </div>
      {{ end }}

      {{/* Display Bookmarks */}}
      {{ if gt (len $bookmarks) 0 }}
        <div class="webmention-bookmarks">
          <h3>🔖 Bookmarked by {{ len $bookmarks }} {{ if eq (len $bookmarks) 1 }}person{{ else }}people{{ end }}</h3>
          <div class="bookmark-avatars">
            {{ range first 20 $bookmarks }}
              {{ with .author }}
                <a href="{{ .url }}" class="bookmark-avatar" title="{{ .name }}" rel="nofollow ugc">
                  {{ if .photo }}
                    <img src="{{ .photo }}" alt="{{ .name }}" loading="lazy" />
                  {{ else }}
                    <span class="avatar-placeholder">{{ substr .name 0 1 }}</span>
                  {{ end }}
                </a>
              {{ end }}
            {{ end }}
            {{ if gt (len $bookmarks) 20 }}
              <span class="more-count">+{{ sub (len $bookmarks) 20 }} more</span>
            {{ end }}
          </div>
        </div>
      {{ end }}

      {{/* Display Replies (Comments) */}}
      {{ if gt (len $replies) 0 }}
        <div class="webmention-replies">
          <h3>💬 {{ len $replies }} {{ if eq (len $replies) 1 }}Reply{{ else }}Replies{{ end }}</h3>

          {{ range $replies }}
            <article class="webmention h-cite" id="webmention-{{ .wm-id }}">
              <div class="webmention-header">
                {{ with .author }}
                  <a href="{{ .url }}" class="webmention-author" rel="nofollow ugc">
                    {{ if .photo }}
                      <img src="{{ .photo }}" alt="{{ .name }}" class="webmention-avatar" loading="lazy" />
                    {{ else }}
                      <span class="avatar-placeholder large">{{ substr .name 0 1 }}</span>
                    {{ end }}
                    <span class="webmention-author-name">{{ .name }}</span>
                  </a>
                {{ end }}

                <a href="{{ .url }}" class="webmention-date" rel="nofollow ugc">
                  <time datetime="{{ .published }}">
                    {{ if .published }}
                      {{ dateFormat "January 2, 2006 at 3:04 PM" .published }}
                    {{ else }}
                      {{ dateFormat "January 2, 2006 at 3:04 PM" .wm-received }}
                    {{ end }}
                  </time>
                </a>
              </div>

              <div class="webmention-content">
                {{ if .content.html }}
                  {{ .content.html | safeHTML }}
                {{ else if .content.text }}
                  <p>{{ .content.text }}</p>
                {{ else }}
                  <p><em>mentioned this post</em></p>
                {{ end }}
              </div>

              <div class="webmention-footer">
                <a href="{{ .url }}" class="webmention-source" rel="nofollow ugc">
                  View original →
                </a>
              </div>
            </article>
          {{ end }}
        </div>
      {{ end }}

      {{/* Display Other Mentions */}}
      {{ if gt (len $mentions) 0 }}
        <div class="webmention-mentions">
          <h3>🔗 Mentioned in</h3>
          {{ range $mentions }}
            <div class="webmention-mention">
              {{ with .author }}
                <a href="{{ .url }}" rel="nofollow ugc">{{ .name }}</a>
              {{ else }}
                <a href="{{ .url }}" rel="nofollow ugc">Someone</a>
              {{ end }}
              mentioned this in
              <a href="{{ .url }}" rel="nofollow ugc">
                {{ if .name }}{{ .name }}{{ else }}a post{{ end }}
              </a>
            </div>
          {{ end }}
        </div>
      {{ end }}

      {{/* Call to Action */}}
      <div class="webmention-cta">
        <p>
          <strong>Want to comment?</strong> Reply to this post from your Mastodon/Fediverse account,
          or <a href="https://{{ .Site.Params.domain }}" target="_blank" rel="noopener">mention this post's URL</a> in your reply.
          Your comment will appear here automatically!
        </p>
        <p style="font-size: 0.875rem; color: #6b7280; margin-top: 0.5rem;">
          Have your own blog? Send a <a href="https://indieweb.org/Webmention" target="_blank" rel="noopener">webmention</a>
          to <code>{{ .Site.Params.webmention_endpoint }}</code>
        </p>
      </div>
    </section>

  {{ else }}
    {{/* No webmentions yet */}}
    <section class="webmentions-empty" id="webmentions">
      <h2>Comments</h2>
      <p>
        <strong>Be the first to comment!</strong> Reply to this post from your Mastodon/Fediverse account,
        or mention this post's URL in your reply. Your comment will appear here automatically via
        <a href="https://indieweb.org/Webmention">webmention</a>.
      </p>
      <p class="webmentions-info">
        Don't have a Mastodon account? <a href="https://joinmastodon.org/" target="_blank" rel="noopener">Join Mastodon</a>
        or follow this blog at <strong>@{{ .Site.Params.domain }}@web.brid.gy</strong>
      </p>
      <p class="webmentions-info">
        <a href="https://indieweb.org/Webmention">What's this?</a>
      </p>
    </section>
  {{ end }}

{{ else }}
  {{/* Data file doesn't exist yet */}}
  <section class="webmentions-loading" id="webmentions">
    <h2>Webmentions</h2>
    <p>Webmentions will appear here once the site has been built.</p>
  </section>
{{ end }}
```

### Step 3.2: Include Webmentions in Post Layout

Edit `layouts/_default/single.html` and add at the bottom:

```html
<article class="h-entry">
  {{/* Your existing post content */}}
  <h1 class="p-name">{{ .Title }}</h1>
  <div class="e-content">{{ .Content }}</div>

  {{/* Add webmentions section */}}
  {{ partial "webmentions.html" . }}
</article>
```

### Step 3.3: Create Initial Empty Data File

For local testing, create `data/webmentions.json`:

```json
{
  "type": "feed",
  "name": "Webmentions",
  "children": []
}
```

Commit this:
```bash
git add data/webmentions.json
git commit -m "Add empty webmentions data file"
git push
```

### Step 3.4: Understanding Comment Options for Users

**Important**: Webmentions are designed for site-to-site communication, not as a traditional comment form. The implementation above directs users to comment via Mastodon/Fediverse (through Bridgy Fed), which provides the best user experience.

**For users without websites:**
- ✅ **Recommended**: Comment via Mastodon/Fediverse (Bridgy Fed - covered in Phase 5)
- ⚠️ Alternative: Use [Comment Parade](https://commentpara.de/) for anonymous webmentions
- ⚠️ Alternative: Add traditional comment systems (Disqus, Commento, utterances, giscus)

**Why not use webmention forms?**
- Traditional webmention forms require users to provide a "source URL" (their website)
- Most blog readers don't have their own websites
- Mastodon/Fediverse provides a much better UX: users just reply normally

**If you need traditional comments:**
Consider these alternatives alongside webmentions:
- **utterances**: GitHub Issues-based comments (free, open source)
- **giscus**: GitHub Discussions-based comments (free, open source)
- **Commento**: Privacy-focused ($5-10/month)
- **Disqus**: Popular but privacy concerns (free/paid tiers)

---

## Phase 4: GitLab CI/CD Integration

### Step 4.1: Create Fetch Script

Create `.gitlab/scripts/fetch-webmentions.sh`:

```bash
#!/bin/bash
set -e

echo "Fetching webmentions..."

# Configuration
DOMAIN="${SITE_DOMAIN}"
TOKEN="${WEBMENTION_TOKEN}"
OUTPUT_FILE="data/webmentions.json"

# Validate required variables
if [ -z "$DOMAIN" ]; then
  echo "Error: SITE_DOMAIN environment variable not set"
  exit 1
fi

if [ -z "$TOKEN" ]; then
  echo "Error: WEBMENTION_TOKEN environment variable not set"
  exit 1
fi

# Create data directory if it doesn't exist
mkdir -p data

# Fetch webmentions from webmention.io API
# Using per-page=9999 to get all mentions
# You can add &since= parameter for incremental updates
API_URL="https://webmention.io/api/mentions.jf2?domain=${DOMAIN}&token=${TOKEN}&per-page=9999"

echo "Fetching from: ${API_URL}"

# Fetch and save
HTTP_CODE=$(curl -s -w "%{http_code}" -o "${OUTPUT_FILE}" "${API_URL}")

if [ "$HTTP_CODE" -eq 200 ]; then
  echo "✓ Successfully fetched webmentions"

  # Count mentions
  MENTION_COUNT=$(jq '.children | length' "${OUTPUT_FILE}" 2>/dev/null || echo "0")
  echo "  Total webmentions: ${MENTION_COUNT}"

  # Show breakdown by type
  echo "  Breakdown:"
  jq -r '.children | group_by(."wm-property") | .[] | "\(.length) \(.[0]."wm-property")"' "${OUTPUT_FILE}" 2>/dev/null || echo "  (unable to parse)"

  exit 0
else
  echo "✗ Failed to fetch webmentions (HTTP ${HTTP_CODE})"

  # If file exists, keep the old one
  if [ -f "${OUTPUT_FILE}.backup" ]; then
    echo "  Restoring previous version"
    mv "${OUTPUT_FILE}.backup" "${OUTPUT_FILE}"
  fi

  exit 1
fi
```

Make it executable:
```bash
chmod +x .gitlab/scripts/fetch-webmentions.sh
```

### Step 4.2: Update GitLab CI Configuration

Edit `.gitlab-ci.yml`:

**Option A: Add to existing pipeline**

```yaml
# Existing stages
stages:
  - build
  - deploy
  - fetch  # Add this new stage

# Your existing build/deploy jobs...

# Add this new job
fetch-webmentions:
  stage: fetch
  image: alpine:latest
  before_script:
    - apk add --no-cache curl jq git
  script:
    - git config user.name "GitLab CI"
    - git config user.email "ci@gitlab.com"

    # Fetch latest changes
    - git pull origin main

    # Backup existing file
    - if [ -f data/webmentions.json ]; then cp data/webmentions.json data/webmentions.json.backup; fi

    # Run fetch script
    - .gitlab/scripts/fetch-webmentions.sh

    # Check if file changed
    - |
      if git diff --quiet data/webmentions.json; then
        echo "No new webmentions"
      else
        echo "New webmentions found, committing..."
        git add data/webmentions.json
        git commit -m "Update webmentions [skip ci]"

        # Push using CI token
        git push "https://gitlab-ci-token:${CI_JOB_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.git" HEAD:${CI_COMMIT_REF_NAME}
      fi

  rules:
    # Run on schedule
    - if: $CI_PIPELINE_SOURCE == "schedule"
    # Run on manual trigger
    - if: $CI_PIPELINE_SOURCE == "web"
      when: manual

  variables:
    SITE_DOMAIN: "yourdomain.com"  # Change this!

  only:
    - main
```

**Option B: Complete example with Hugo build**

```yaml
stages:
  - fetch
  - build
  - deploy

variables:
  HUGO_VERSION: "0.121.0"
  GIT_SUBMODULE_STRATEGY: recursive

# Fetch webmentions first
fetch-webmentions:
  stage: fetch
  image: alpine:latest
  before_script:
    - apk add --no-cache curl jq git
  script:
    - git config user.name "GitLab CI"
    - git config user.email "ci@gitlab.com"
    - git pull origin main || true

    - if [ -f data/webmentions.json ]; then cp data/webmentions.json data/webmentions.json.backup; fi
    - .gitlab/scripts/fetch-webmentions.sh

    - |
      if ! git diff --quiet data/webmentions.json 2>/dev/null; then
        echo "Committing new webmentions..."
        git add data/webmentions.json
        git commit -m "Update webmentions [skip ci]"
        git push "https://gitlab-ci-token:${CI_JOB_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.git" HEAD:${CI_COMMIT_REF_NAME}
      else
        echo "No changes to webmentions"
      fi

  variables:
    SITE_DOMAIN: "yourdomain.com"

  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_PIPELINE_SOURCE == "web"
      when: manual
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
      allow_failure: true

# Build Hugo site
build:
  stage: build
  image: registry.gitlab.com/pages/hugo/hugo_extended:${HUGO_VERSION}
  script:
    - hugo --minify
  artifacts:
    paths:
      - public
  only:
    - main

# Deploy to GitLab Pages (or your hosting)
pages:
  stage: deploy
  script:
    - echo "Deploying to GitLab Pages..."
  artifacts:
    paths:
      - public
  only:
    - main
```

### Step 4.3: Add GitLab CI/CD Variables

1. Go to your GitLab project
2. Settings → CI/CD → Variables → Expand
3. Add these variables:

| Key | Value | Protected | Masked |
|-----|-------|-----------|--------|
| `WEBMENTION_TOKEN` | Your webmention.io API token | ✓ | ✓ |
| `SITE_DOMAIN` | yourdomain.com | ✗ | ✗ |

### Step 4.4: Set Up Scheduled Pipeline

1. **Go to CI/CD → Schedules**
   - Navigate to: `https://gitlab.com/username/project/-/pipeline_schedules`

2. **Click "New schedule"**

3. **Configure schedule:**
   - **Description:** "Fetch webmentions daily"
   - **Interval Pattern:** Custom
   - **Cron notation:** `0 2 * * *` (runs at 2 AM daily)
     - Or use: `0 */6 * * *` (every 6 hours)
     - Or use: `0 0,12 * * *` (twice daily at midnight and noon)
   - **Target branch:** `main`
   - **Activated:** ✓

4. **Save pipeline schedule**

### Step 4.5: Test the Pipeline

**Manual test:**

1. Go to CI/CD → Pipelines
2. Click "Run pipeline"
3. Select branch: `main`
4. Click "Run pipeline"
5. Watch the "fetch-webmentions" job run

**Check results:**

```bash
# Pull latest changes
git pull

# Check the webmentions file
cat data/webmentions.json | jq '.children | length'
# Should show number of webmentions
```

---

## Phase 5: Bridgy Fed Integration

### Step 5.1: Sign Up for Bridgy Fed

1. **Visit Bridgy Fed**
   ```
   https://fed.brid.gy/
   ```

2. **Enter your domain**
   - Enter: `https://yourdomain.com`
   - Click "Sign up"

3. **Verify your domain**
   - Bridgy Fed will check for:
     - rel="me" link (already added in Phase 1)
     - h-card with author info (already added in Phase 2)
   - Should verify automatically

4. **Get your Fediverse handle**
   - You'll get: `@yourdomain.com@web.brid.gy`
   - This is your site's Fediverse identity!

### Step 5.2: Add Bridgy Fed Info to Your Site

Create or edit `layouts/partials/follow.html`:

```html
<section class="follow-section">
  <h3>Follow this blog</h3>
  <p>
    You can follow this blog on the Fediverse (Mastodon, Pleroma, etc.) by searching for:
  </p>
  <p class="fediverse-handle">
    <strong>
      @{{ .Site.Params.domain }}@web.brid.gy
    </strong>
  </p>
  <p class="follow-instructions">
    Search for this handle in your Fediverse app to follow and see new posts.
    Your replies will appear as comments here!
    <a href="https://fed.brid.gy/web/{{ .Site.Params.domain }}"
       target="_blank" rel="noopener">
      View profile →
    </a>
  </p>
</section>
```

Include in your sidebar or footer:
```html
{{ partial "follow.html" . }}
```

### Step 5.3: Test Bridgy Fed

1. **From your Mastodon account:**
   - Search for: `@yourdomain.com@web.brid.gy`
   - Click "Follow"

2. **Publish a test post:**
   ```bash
   hugo new posts/test-bridgy-fed.md
   ```

   Edit the post:
   ```markdown
   ---
   title: "Testing Bridgy Fed"
   date: 2025-12-20
   ---

   This is a test post to verify Bridgy Fed integration.
   If you're reading this on Mastodon, try replying!
   ```

3. **Deploy:**
   ```bash
   git add .
   git commit -m "Add test post for Bridgy Fed"
   git push
   ```

4. **Reply from Mastodon:**
   - Copy the post URL
   - Create a new post on Mastodon with the URL
   - Reply to that post
   - Wait 5-10 minutes

5. **Fetch webmentions:**
   - Trigger the GitLab pipeline manually
   - Or run: `.gitlab/scripts/fetch-webmentions.sh` locally
   - Check `data/webmentions.json` for new entries

### Step 5.4: Set Up Brid.gy (Optional - for Twitter/Reddit)

1. **Visit Brid.gy**
   ```
   https://brid.gy/
   ```

2. **Connect accounts:**
   - Click "Twitter" (or other platforms)
   - Authorize the connection
   - Enable "Publish" to send webmentions

3. **Configure:**
   - Choose "Send webmentions for:"
     - ✓ Mentions
     - ✓ Replies
     - ✓ Likes (optional)
   - Target: Your webmention endpoint (already set)

---

## Phase 6: Styling and Polish

### Step 6.1: Create Webmentions CSS

Create `assets/css/webmentions.css`:

```css
/* ============================================
   Webmentions Styling
   ============================================ */

/* Main webmentions section */
.webmentions {
  margin: 4rem 0 2rem;
  padding: 2rem 0 0;
  border-top: 2px solid #e5e7eb;
}

.webmentions h2 {
  font-size: 1.875rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
  color: #1f2937;
}

.webmentions h3 {
  font-size: 1.25rem;
  font-weight: 600;
  margin: 2rem 0 1rem;
  color: #374151;
}

.webmentions-info {
  font-size: 0.875rem;
  color: #6b7280;
  margin-bottom: 2rem;
}

.webmentions-info a {
  color: #3b82f6;
  text-decoration: underline;
}

/* Empty state */
.webmentions-empty,
.webmentions-loading {
  margin: 4rem 0 2rem;
  padding: 2rem;
  background: #f9fafb;
  border-radius: 8px;
  text-align: center;
}

/* Likes section */
.webmention-likes,
.webmention-reposts,
.webmention-bookmarks {
  margin: 2rem 0;
  padding: 1.5rem;
  background: #f9fafb;
  border-radius: 8px;
}

.like-avatars,
.repost-avatars,
.bookmark-avatars {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 1rem;
}

.like-avatar,
.repost-avatar,
.bookmark-avatar {
  position: relative;
  display: block;
  transition: transform 0.2s;
}

.like-avatar:hover,
.repost-avatar:hover,
.bookmark-avatar:hover {
  transform: translateY(-2px);
  z-index: 10;
}

.like-avatar img,
.repost-avatar img,
.bookmark-avatar img {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  border: 2px solid white;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.avatar-placeholder {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #e5e7eb;
  color: #6b7280;
  font-weight: 600;
  font-size: 1rem;
  text-transform: uppercase;
  border: 2px solid white;
}

.avatar-placeholder.large {
  width: 48px;
  height: 48px;
  font-size: 1.25rem;
}

.more-count {
  display: inline-flex;
  align-items: center;
  padding: 0 0.75rem;
  height: 40px;
  background: #e5e7eb;
  border-radius: 20px;
  font-size: 0.875rem;
  color: #6b7280;
  font-weight: 500;
}

/* Replies (Comments) section */
.webmention-replies {
  margin: 2rem 0;
}

.webmention {
  margin: 1.5rem 0;
  padding: 1.5rem;
  background: white;
  border: 1px solid #e5e7eb;
  border-left: 3px solid #3b82f6;
  border-radius: 8px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.webmention-header {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.webmention-author {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  text-decoration: none;
  color: #1f2937;
  flex: 1;
}

.webmention-author:hover {
  color: #3b82f6;
}

.webmention-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: 2px solid #e5e7eb;
}

.webmention-author-name {
  font-weight: 600;
  font-size: 1rem;
}

.webmention-date {
  font-size: 0.875rem;
  color: #6b7280;
  text-decoration: none;
  white-space: nowrap;
}

.webmention-date:hover {
  color: #3b82f6;
}

.webmention-content {
  margin: 1rem 0;
  line-height: 1.7;
  color: #374151;
}

.webmention-content p {
  margin: 0.75rem 0;
}

.webmention-content a {
  color: #3b82f6;
  text-decoration: underline;
}

.webmention-content img {
  max-width: 100%;
  height: auto;
  border-radius: 4px;
  margin: 0.5rem 0;
}

.webmention-footer {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid #f3f4f6;
}

.webmention-source {
  font-size: 0.875rem;
  color: #6b7280;
  text-decoration: none;
}

.webmention-source:hover {
  color: #3b82f6;
  text-decoration: underline;
}

/* Other mentions */
.webmention-mentions {
  margin: 2rem 0;
}

.webmention-mention {
  padding: 0.75rem;
  margin: 0.5rem 0;
  background: #f9fafb;
  border-radius: 4px;
  font-size: 0.9375rem;
}

.webmention-mention a {
  color: #3b82f6;
  text-decoration: none;
}

.webmention-mention a:hover {
  text-decoration: underline;
}

/* Call to action */
.webmention-cta {
  margin: 3rem 0 0;
  padding: 1.5rem;
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  border-radius: 8px;
  text-align: center;
}

.webmention-cta p {
  margin: 0;
  color: #1e40af;
}

.webmention-cta a {
  color: #1e40af;
  font-weight: 600;
  text-decoration: underline;
}

/* Responsive design */
@media (max-width: 640px) {
  .webmentions {
    margin: 2rem 0 1rem;
  }

  .webmention {
    padding: 1rem;
  }

  .webmention-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .webmention-author {
    width: 100%;
  }

  .webmention-date {
    margin-left: 60px;
  }

  .like-avatar img,
  .repost-avatar img,
  .bookmark-avatar img {
    width: 32px;
    height: 32px;
  }

  .avatar-placeholder {
    width: 32px;
    height: 32px;
    font-size: 0.875rem;
  }
}

/* Dark mode support (optional) */
@media (prefers-color-scheme: dark) {
  .webmentions {
    border-top-color: #374151;
  }

  .webmentions h2,
  .webmentions h3 {
    color: #f9fafb;
  }

  .webmentions-info {
    color: #9ca3af;
  }

  .webmentions-empty,
  .webmentions-loading {
    background: #1f2937;
    color: #e5e7eb;
  }

  .webmention-likes,
  .webmention-reposts,
  .webmention-bookmarks {
    background: #1f2937;
  }

  .webmention {
    background: #1f2937;
    border-color: #374151;
    color: #e5e7eb;
  }

  .webmention-author {
    color: #f9fafb;
  }

  .webmention-content {
    color: #d1d5db;
  }

  .webmention-mention {
    background: #1f2937;
  }

  .webmention-cta {
    background: #1e3a5f;
    border-color: #1e40af;
    color: #bfdbfe;
  }

  .webmention-cta p {
    color: #bfdbfe;
  }
}
```

### Step 6.2: Include CSS in Hugo

**Option A: If using asset pipeline**

In your `layouts/partials/head.html`:

```html
{{ $webmentionsCSS := resources.Get "css/webmentions.css" }}
{{ $webmentionsCSS := $webmentionsCSS | resources.Minify }}
<link rel="stylesheet" href="{{ $webmentionsCSS.Permalink }}">
```

**Option B: If using static CSS**

Move to `static/css/webmentions.css` and add to head:

```html
<link rel="stylesheet" href="{{ "css/webmentions.css" | relURL }}">
```

### Step 6.3: Test Locally

```bash
# Build and serve locally
hugo server -D

# Open browser to a post
open http://localhost:1313/posts/your-post/

# Check webmentions section appears
# (Will be empty if no data yet)
```

---

## Testing and Validation

### Step 1: Validate Microformats

**Test with microformats.io:**
```
1. Visit: https://microformats.io/
2. Enter your blog post URL
3. Verify h-entry is detected:
   - p-name (post title)
   - e-content (post content)
   - dt-published (date)
   - p-author (author)
   - u-url (permalink)
```

**Test with IndieWebify.me:**
```
1. Visit: https://indiewebify.me/
2. Test each level:
   - Level 1: rel=me links
   - Level 2: h-entry markup
   - Level 3: webmention endpoint
```

### Step 2: Test Webmention Endpoint

**Use Telegraph to send test webmention:**

1. Visit: https://telegraph.p3k.io/send-a-webmention
2. Enter source URL (any page linking to your post)
3. Enter target URL (your blog post)
4. Click "Send Webmention"
5. Should see success message

**Verify received:**

```bash
# Check webmention.io dashboard
open https://webmention.io/dashboard

# Or fetch directly
curl "https://webmention.io/api/mentions.jf2?domain=yourdomain.com&token=YOUR_TOKEN" | jq
```

### Step 3: Test GitLab CI/CD

**Trigger manual pipeline:**

```bash
# Via GitLab UI:
# 1. Go to CI/CD → Pipelines
# 2. Click "Run pipeline"
# 3. Select branch: main
# 4. Click "Run pipeline"

# Via command line (requires gitlab CLI):
glab ci run
```

**Verify results:**

```bash
# Pull latest
git pull

# Check webmentions file
cat data/webmentions.json | jq '.children | length'

# View by type
cat data/webmentions.json | jq '.children | group_by(."wm-property") | map({type: .[0]."wm-property", count: length})'
```

### Step 4: Test Bridgy Fed

**From Mastodon:**

1. Search for: `@yourdomain.com@web.brid.gy`
2. Follow the account
3. You should see your blog posts appear in your feed
4. Reply to a post
5. Wait 10-15 minutes
6. Trigger webmention fetch
7. Check if reply appears on your blog

**Verify Bridgy Fed is working:**

```
Visit: https://fed.brid.gy/web/yourdomain.com
Should show:
- Your domain verification status
- Recent activities
- Any errors
```

### Step 5: End-to-End Test

**Complete flow test:**

1. **Publish a new post**
   ```bash
   hugo new posts/webmention-test.md
   # Edit and add content
   git add .
   git commit -m "Add webmention test post"
   git push
   ```

2. **Send a webmention**
   - Use Telegraph: https://telegraph.p3k.io/send-a-webmention
   - Or reply from Mastodon

3. **Wait and fetch**
   - Wait 5-10 minutes
   - Trigger GitLab pipeline (or wait for scheduled run)

4. **Verify display**
   - Pull latest changes
   - Build locally: `hugo server`
   - Check post shows webmention

---

## Troubleshooting

### Issue: Webmention.io authentication fails

**Symptoms:**
- Can't sign in to webmention.io
- "Unable to verify your domain"

**Solutions:**

1. **Check rel=me links**
   ```bash
   # View your homepage source
   curl -s https://yourdomain.com | grep 'rel="me"'
   # Should show at least one rel=me link
   ```

2. **Verify GitHub link**
   ```html
   <!-- Must be exact format -->
   <link rel="me" href="https://github.com/yourusername" />
   ```

3. **Check domain is deployed**
   - Ensure changes are live on production
   - Clear CDN cache if applicable

### Issue: Webmentions not appearing on site

**Symptoms:**
- webmentions.json has data
- But webmentions don't display on posts

**Solutions:**

1. **Check file location**
   ```bash
   # File must be in data/ directory
   ls -la data/webmentions.json
   ```

2. **Verify JSON format**
   ```bash
   # Validate JSON
   cat data/webmentions.json | jq
   # Should parse without errors
   ```

3. **Check URL matching**
   ```bash
   # URLs must match exactly
   # Check your post permalink:
   hugo list all | grep "your-post"

   # Compare with webmention target:
   cat data/webmentions.json | jq '.children[0]."wm-target"'
   ```

4. **Debug in template**
   ```html
   <!-- Add to webmentions.html for debugging -->
   <pre>
   Permalink: {{ .Permalink }}
   Webmentions file exists: {{ fileExists "data/webmentions.json" }}
   Total mentions: {{ len (index site.Data "webmentions").children }}
   </pre>
   ```

### Issue: GitLab CI/CD fetch job fails

**Symptoms:**
- Pipeline fails at fetch-webmentions stage
- Error in job logs

**Solutions:**

1. **Check environment variables**
   ```bash
   # In GitLab: Settings → CI/CD → Variables
   # Verify these exist:
   # - WEBMENTION_TOKEN
   # - SITE_DOMAIN (in .gitlab-ci.yml)
   ```

2. **Test script locally**
   ```bash
   # Set environment variables
   export SITE_DOMAIN="yourdomain.com"
   export WEBMENTION_TOKEN="your-token"

   # Run script
   .gitlab/scripts/fetch-webmentions.sh
   ```

3. **Check API token**
   ```bash
   # Test token manually
   curl "https://webmention.io/api/mentions.jf2?domain=yourdomain.com&token=YOUR_TOKEN"
   # Should return JSON, not error
   ```

4. **Verify git push permissions**
   - CI/CD jobs need write access
   - Check: Settings → Repository → Protected branches
   - Ensure "Allowed to push" includes "Maintainers + Developers"

### Issue: Bridgy Fed not working

**Symptoms:**
- Mastodon replies don't appear as webmentions
- Can't find domain on Bridgy Fed

**Solutions:**

1. **Verify domain on Bridgy Fed**
   ```
   Visit: https://fed.brid.gy/web/yourdomain.com
   Check for errors
   ```

2. **Check h-card exists**
   ```
   Visit: https://microformats.io/
   Enter your homepage URL
   Verify h-card is detected with:
   - p-name (your name)
   - u-url (your site URL)
   - u-photo (avatar - optional)
   ```

3. **Wait longer**
   - Bridgy Fed can take 10-30 minutes
   - Check Bridgy Fed logs for your domain

4. **Manually trigger Bridgy Fed**
   ```
   Visit: https://fed.brid.gy/
   Enter your domain again
   Click "Refresh"
   ```

### Issue: Scheduled pipeline not running

**Symptoms:**
- GitLab schedule exists but doesn't trigger

**Solutions:**

1. **Check schedule is active**
   - CI/CD → Schedules
   - Verify toggle is "Active"

2. **Verify cron syntax**
   ```
   0 2 * * *     # Correct: runs at 2 AM daily
   0 */6 * * *   # Correct: runs every 6 hours
   * * * * *     # Wrong: runs every minute (too frequent)
   ```

3. **Check last run**
   - Click on schedule
   - View "Last pipeline"
   - Check for errors

4. **Trigger manually**
   - Click "Play" button on schedule
   - Verify job runs successfully

### Issue: Microformats not validating

**Symptoms:**
- Microformats validators show no h-entry
- IndieWebify.me fails validation

**Solutions:**

1. **Check class names**
   ```html
   <!-- Correct -->
   <article class="h-entry">
     <h1 class="p-name">Title</h1>
     <div class="e-content">Content</div>
   </article>

   <!-- Wrong -->
   <article class="h entry">  <!-- Space instead of dash -->
   <article class="hentry">    <!-- Missing dash -->
   ```

2. **Verify HTML structure**
   ```bash
   # View rendered HTML
   curl https://yourdomain.com/posts/your-post/ | grep -A 20 "h-entry"
   ```

3. **Check template overrides**
   ```bash
   # Find which template is actually used
   hugo --templateMetrics
   # Look for single.html or post.html
   ```

### Issue: Comments from wrong post appearing

**Symptoms:**
- Post A shows comments from Post B

**Solutions:**

1. **Check URL trailing slashes**
   ```go
   // In webmentions.html, use consistent comparison:
   {{ if eq (strings.TrimSuffix "/" .wm-target) (strings.TrimSuffix "/" $permalink) }}
   ```

2. **Verify permalinks**
   ```toml
   # In config.toml, ensure consistent permalink structure
   [permalinks]
     posts = "/posts/:slug/"  # or "/:year/:month/:slug/"
   ```

3. **Check baseURL**
   ```toml
   # In config.toml
   baseURL = "https://yourdomain.com/"  # Include trailing slash
   ```

---

## Optional Enhancements

### Enhancement 1: Comment Count in Post List

Show comment counts in your post listings.

Edit `layouts/_default/list.html`:

```html
{{ range .Pages }}
<article class="post-summary">
  <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
  <time>{{ .Date.Format "January 2, 2006" }}</time>

  {{/* Add comment count */}}
  {{ if fileExists "data/webmentions.json" }}
    {{ $allMentions := index site.Data "webmentions" }}
    {{ $postURL := .Permalink }}
    {{ $commentCount := 0 }}

    {{ range $allMentions.children }}
      {{ if and (eq .wm-property "in-reply-to") (eq (strings.TrimSuffix "/" .wm-target) (strings.TrimSuffix "/" $postURL)) }}
        {{ $commentCount = add $commentCount 1 }}
      {{ end }}
    {{ end }}

    {{ if gt $commentCount 0 }}
      <span class="comment-count">
        💬 {{ $commentCount }} {{ if eq $commentCount 1 }}comment{{ else }}comments{{ end }}
      </span>
    {{ end }}
  {{ end }}

  {{ .Summary }}
  <a href="{{ .Permalink }}">Read more...</a>
</article>
{{ end }}
```

### Enhancement 2: RSS Feed with Webmentions

Include comment counts in your RSS feed.

Create `layouts/_default/rss.xml`:

```xml
{{ printf "<?xml version=\"1.0\" encoding=\"utf-8\" standalone=\"yes\"?>" | safeHTML }}
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ .Site.Title }}</title>
    <link>{{ .Permalink }}</link>
    <description>{{ .Site.Params.description }}</description>
    <language>{{ .Site.LanguageCode }}</language>
    <lastBuildDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" }}</lastBuildDate>
    <atom:link href="{{ .Permalink }}" rel="self" type="application/rss+xml" />

    {{ range .Pages }}
    {{ $commentCount := 0 }}
    {{ if fileExists "data/webmentions.json" }}
      {{ $allMentions := index site.Data "webmentions" }}
      {{ $postURL := .Permalink }}
      {{ range $allMentions.children }}
        {{ if and (eq .wm-property "in-reply-to") (eq (strings.TrimSuffix "/" .wm-target) (strings.TrimSuffix "/" $postURL)) }}
          {{ $commentCount = add $commentCount 1 }}
        {{ end }}
      {{ end }}
    {{ end }}

    <item>
      <title>{{ .Title }}</title>
      <link>{{ .Permalink }}</link>
      <pubDate>{{ .Date.Format "Mon, 02 Jan 2006 15:04:05 -0700" }}</pubDate>
      <guid>{{ .Permalink }}</guid>
      <description>{{ .Summary | html }}{{ if gt $commentCount 0 }} ({{ $commentCount }} comments){{ end }}</description>
    </item>
    {{ end }}
  </channel>
</rss>
```

### Enhancement 3: Webmention Sending

Automatically send webmentions when you link to other blogs.

Create `.gitlab/scripts/send-webmentions.sh`:

```bash
#!/bin/bash
# Script to send webmentions for links in new posts

echo "Sending webmentions..."

# Find all URLs in your content
URLS=$(grep -Phorw 'https?://[^\s"\)]+' content/posts/*.md | sort -u)

for TARGET_URL in $URLS; do
  # Skip internal links
  if [[ $TARGET_URL == *"yourdomain.com"* ]]; then
    continue
  fi

  echo "Checking $TARGET_URL for webmention endpoint..."

  # Send via Telegraph (or implement webmention discovery)
  # This is a simplified version
  # For production, use: https://github.com/nekr0z/static-webmentions
done

echo "Done sending webmentions"
```

Add to `.gitlab-ci.yml`:

```yaml
send-webmentions:
  stage: deploy
  script:
    - .gitlab/scripts/send-webmentions.sh
  only:
    - main
  when: on_success
```

### Enhancement 4: Moderation System

Add ability to hide specific webmentions.

Create `data/webmentions-blocked.json`:

```json
{
  "blocked_ids": [123456, 789012],
  "blocked_domains": ["spam.example.com"]
}
```

Update `layouts/partials/webmentions.html`:

```html
{{ $blocked := slice }}
{{ if fileExists "data/webmentions-blocked.json" }}
  {{ $blockedData := index site.Data "webmentions-blocked" }}
  {{ $blocked = $blockedData.blocked_ids }}
{{ end }}

{{ range $pageMentions }}
  {{/* Check if blocked */}}
  {{ if not (in $blocked .wm-id) }}
    {{/* Display webmention */}}
  {{ end }}
{{ end }}
```

### Enhancement 5: Webmention Analytics

Track webmention statistics.

Create `layouts/shortcodes/webmention-stats.html`:

```html
{{ if fileExists "data/webmentions.json" }}
  {{ $allMentions := index site.Data "webmentions" }}
  {{ $total := len $allMentions.children }}
  {{ $replies := 0 }}
  {{ $likes := 0 }}
  {{ $reposts := 0 }}

  {{ range $allMentions.children }}
    {{ if eq .wm-property "in-reply-to" }}
      {{ $replies = add $replies 1 }}
    {{ else if eq .wm-property "like-of" }}
      {{ $likes = add $likes 1 }}
    {{ else if eq .wm-property "repost-of" }}
      {{ $reposts = add $reposts 1 }}
    {{ end }}
  {{ end }}

  <div class="webmention-stats">
    <h3>Webmention Statistics</h3>
    <ul>
      <li>Total webmentions: {{ $total }}</li>
      <li>Comments: {{ $replies }}</li>
      <li>Likes: {{ $likes }}</li>
      <li>Reposts: {{ $reposts }}</li>
    </ul>
  </div>
{{ end }}
```

Use in any page:
```markdown
{{< webmention-stats >}}
```

---

## Maintenance

### Daily Tasks

**None!** - The GitLab scheduled pipeline handles everything automatically.

### Weekly Tasks

1. **Check pipeline success**
   - Visit: GitLab → CI/CD → Schedules
   - Verify last run was successful

2. **Monitor Bridgy Fed**
   - Visit: https://fed.brid.gy/web/yourdomain.com
   - Check for any errors

### Monthly Tasks

1. **Review webmentions**
   ```bash
   # Check for spam or unwanted mentions
   cat data/webmentions.json | jq '.children[] | select(.wm-property == "in-reply-to") | {author: .author.name, content: .content.text}'
   ```

2. **Update blocked list if needed**
   ```bash
   # Add to data/webmentions-blocked.json
   ```

3. **Check API token validity**
   - Visit: https://webmention.io/settings
   - Verify token hasn't expired

### Troubleshooting Commands

```bash
# Test webmention endpoint
curl -X POST \
  -d "source=https://example.com/post" \
  -d "target=https://yourdomain.com/posts/test/" \
  https://webmention.io/yourdomain.com/webmention

# Fetch latest webmentions manually
curl "https://webmention.io/api/mentions.jf2?domain=yourdomain.com&token=YOUR_TOKEN" | jq > data/webmentions.json

# Validate microformats
curl https://yourdomain.com/posts/test/ | grep -o 'class="[^"]*h-[^"]*"'

# Check git status in CI
git log --oneline -5 --grep="webmentions"

# Test Hugo build locally
hugo --gc --minify

# Validate data file
cat data/webmentions.json | jq 'if .children then "Valid" else "Invalid" end'
```

---

## Summary

You now have a complete webmentions implementation for Hugo with:

✅ Webmention.io endpoint for receiving mentions
✅ Microformats (h-entry, h-card) for semantic markup
✅ GitLab CI/CD automation for fetching webmentions daily
✅ Bridgy Fed integration for Fediverse comments
✅ Beautiful display of comments, likes, and reposts
✅ Build-time rendering for SEO
✅ Zero ongoing costs

### What's Next?

1. **Promote your webmention support:**
   - Add to your about page
   - Mention in blog posts
   - Share Fediverse handle

2. **Engage with commenters:**
   - Reply to comments on their platforms
   - Thank people for interactions

3. **Monitor and optimize:**
   - Check analytics
   - Improve styling
   - Add enhancements as needed

### Resources

- **Webmention.io:** https://webmention.io/
- **Bridgy Fed:** https://fed.brid.gy/
- **IndieWeb Wiki:** https://indieweb.org/
- **Microformats Reference:** https://microformats.org/
- **Hugo Documentation:** https://gohugo.io/

---

**Congratulations!** Your Hugo blog now supports webmentions with full GitLab CI/CD automation. 🎉

---

## Phase 7: POSSE Auto-Publishing to Mastodon & Bluesky

### Overview

This section covers configuring your Hugo site to automatically publish new blog posts to **Mastodon** and **Bluesky** using Bridgy Fed's POSSE (Publish Own Site, Syndicate Elsewhere) functionality.

**Key Features:**
- ✅ Only posts from **December 2025 onwards** are auto-published
- ✅ Historic posts (before December 2025) won't be syndicated
- ✅ All posts still accept webmentions from readers
- ✅ GitLab CI automatically notifies Bridgy Fed after deployment
- ✅ Your website remains the canonical source of truth

### Step 7.1: Understanding POSSE

**POSSE** (Publish Own Site, Syndicate Elsewhere) means:
1. You publish content on your own website first
2. Your content is automatically cross-posted to social networks
3. Your website remains the canonical source
4. Interactions (replies, likes) flow back as webmentions

**How it works with Bridgy Fed:**
- You add hidden `u-syndication` links to your posts
- After deployment, GitLab CI sends webmentions to Bridgy Fed for each post
- Bridgy Fed fetches each post's HTML and parses microformats
- Bridgy Fed finds posts with `u-syndication` links
- Bridgy Fed publishes to Mastodon and/or Bluesky
- Interactions flow back as webmentions

**CRITICAL: RSS Polling vs Webmentions**

Bridgy Fed has two modes of operation:

1. **RSS Polling Mode** (initial behavior):
   - When you first connect, Bridgy Fed polls your RSS feed periodically
   - Automatically discovers and syndicates new posts
   - No manual intervention needed

2. **Webmention Mode** (after sending first webmention):
   - **Once you send ANY webmention** to Bridgy Fed, it STOPS polling your RSS feed
   - From that point forward, you MUST send a webmention for each post
   - This is permanent - Bridgy Fed will not return to RSS polling
   - This is why the GitLab CI automation is essential

**Why this matters:**
- If you set up POSSE without CI automation, posts won't be syndicated
- You must send webmentions for INDIVIDUAL post URLs, not just your homepage
- The correct endpoint is `https://fed.brid.gy/webmention`
- Each webmention must include: `source` (post URL) and `target` (https://fed.brid.gy/)

### Step 7.2: Add POSSE Syndication Markup to Posts

Edit `layouts/_default/single.html` and add syndication links in the post metadata section.

**Find this section** (around line 38-45):

```html
{{ with .Params.categories }}
  <span class="post-categories">
    {{ range . }}
      <a class="p-category" href="{{ "/categories/" | relURL }}{{ . | urlize }}">{{ . }}</a>
    {{ end }}
  </span>
{{ end }}
</div>
```

**Add after it:**

```html
{{ with .Params.categories }}
  <span class="post-categories">
    {{ range . }}
      <a class="p-category" href="{{ "/categories/" | relURL }}{{ . | urlize }}">{{ . }}</a>
    {{ end }}
  </span>
{{ end }}

{{/* POSSE syndication links - only for posts from December 2025 onwards */}}
{{ $cutoffDate := time.AsTime "2025-12-01T00:00:00Z" }}
{{ if ge .Date.Unix $cutoffDate.Unix }}
  {{/* Hidden Bridgy Fed link to trigger federation to Fediverse/Mastodon */}}
  {{/* CRITICAL: Use u-bridgy-fed class, NOT u-syndication for Fediverse */}}
  <a class="u-bridgy-fed" href="https://fed.brid.gy/" hidden></a>
  {{/* Bluesky syndication via bsky.brid.gy - uses u-syndication */}}
  <a class="u-syndication" href="https://bsky.brid.gy/" rel="syndication" style="display:none;">Bluesky</a>
{{ end }}
</div>
```

**What this does:**
- Checks if post date ≥ December 1, 2025
- If yes, adds hidden `u-syndication` links that tell Bridgy Fed to publish to Mastodon and Bluesky
- If no, skips syndication (historic posts protected)
- Links are hidden with `display:none` but readable by Bridgy Fed

**CRITICAL: Bridgy Fed vs Bluesky Syntax**

Bridgy Fed uses DIFFERENT markup for Fediverse vs Bluesky:

**For Fediverse/Mastodon (Bridgy Fed):**
- **Use:** `<a class="u-bridgy-fed" href="https://fed.brid.gy/" hidden></a>`
- **NOT u-syndication** - Bridgy Fed uses its own `u-bridgy-fed` class
- The `hidden` attribute makes it invisible to users but readable by Bridgy Fed
- Must be inside or adjacent to the h-entry element

**For Bluesky (bsky.brid.gy):**
- **Use:** `<a class="u-syndication" href="https://bsky.brid.gy/" rel="syndication">`
- Bluesky uses traditional u-syndication microformat

**Why the difference:**
- Bridgy Fed federates web content to the Fediverse (different from POSSE)
- Bluesky bridge uses traditional POSSE with u-syndication tracking
- Using wrong class/format will prevent posts from appearing

### Step 7.3: Add rel="me" Link for Your Personal Mastodon

Edit `layouts/_partials/head-additions.html`:

**Find this section:**

```html
{{/* IndieAuth */}}
{{ with .Site.Params.github }}
<link rel="me" href="https://github.com/{{ . }}" />
{{ end }}
{{ with .Site.Params.author_email }}
<link rel="me" href="mailto:{{ . }}" />
{{ end }}
```

**Add the Mastodon rel="me" link:**

```html
{{/* IndieAuth */}}
{{ with .Site.Params.github }}
<link rel="me" href="https://github.com/{{ . }}" />
{{ end }}
{{ with .Site.Params.author_email }}
<link rel="me" href="mailto:{{ . }}" />
{{ end }}
{{ with .Site.Params.ananke.social.networks.mastodon.profile }}
<link rel="me" href="{{ . }}" />
{{ end }}
```

**Note:** This adds your **personal** Mastodon account as a rel="me" link for IndieAuth purposes. Your blog's Fediverse identity (e.g., `@gaggl.com@gaggl.com`) is handled separately via your custom webfinger redirect.

### Step 7.4: Update GitLab CI to Notify Bridgy Fed

Edit `.gitlab-ci.yml` to ping Bridgy Fed after deployment.

**Find the deploy stage:**

```yaml
# Deploy the built site
deploy:
  stage: deploy
  timeout: 5 minutes
  script:
    - apk add rsync openssh-client
    # ... SSH setup ...
    # Sync the built site
    - rsync -crtvz --delete ./public/ gitlabci@server:/path/
  only:
    - main
  dependencies:
    - build
```

**Update to:**

```yaml
# Deploy the built site
deploy:
  stage: deploy
  timeout: 5 minutes
  script:
    # Need SSH client, curl, and libxml2-utils for deployment and Bridgy notification
    - apk add rsync openssh-client curl libxml2-utils
    # ... SSH setup ...
    # Sync the built site
    - rsync -crtvz --delete ./public/ gitlabci@server:/path/
    # Notify Bridgy Fed to check for new posts and syndicate to Mastodon/Bluesky
    - echo "Notifying Bridgy Fed of new content..."
    # Send webmentions for recent posts (top 10 from RSS feed)
    - |
      echo "Fetching recent posts from RSS feed..."
      curl -s https://yourdomain.com/index.xml | xmllint --xpath "//item[position() <= 10]/link/text()" - 2>/dev/null | while read -r POST_URL; do
        echo "Sending webmention for: $POST_URL"
        RESPONSE=$(curl -s -X POST https://fed.brid.gy/webmention \
          --data-urlencode "source=${POST_URL}" \
          --data-urlencode "target=https://fed.brid.gy/" \
          2>&1) || true
        echo "  Response: $RESPONSE"
        sleep 2  # Rate limiting - be nice to Bridgy Fed
      done
      echo "Webmention notifications completed"
  only:
    - main
  dependencies:
    - build
```

**Important changes:**
1. Added `curl` and `libxml2-utils` to the `apk add` line
2. **CRITICAL**: Changed endpoint from `/publish/webmention` to `/webmention` (correct endpoint)
3. **CRITICAL**: Sends webmentions for **individual post URLs**, not just homepage
4. Parses RSS feed to get the 10 most recent posts
5. Uses `--data-urlencode` for proper URL encoding
6. Includes 2-second delay between requests (rate limiting)
7. Captures and logs responses for debugging
8. Made non-fatal so deployment succeeds even if Bridgy is down

**Why individual post URLs?**
- Bridgy Fed requires a webmention for each post URL
- Once you've sent webmentions, Bridgy Fed stops polling your RSS feed
- Future posts need individual webmentions to trigger syndication
- Sending homepage URL only doesn't work

**Example for gaggl.com:**

```yaml
- |
  echo "Fetching recent posts from RSS feed..."
  curl -s https://gaggl.com/index.xml | xmllint --xpath "//item[position() <= 10]/link/text()" - 2>/dev/null | while read -r POST_URL; do
    echo "Sending webmention for: $POST_URL"
    RESPONSE=$(curl -s -X POST https://fed.brid.gy/webmention \
      --data-urlencode "source=${POST_URL}" \
      --data-urlencode "target=https://fed.brid.gy/" \
      2>&1) || true
    echo "  Response: $RESPONSE"
    sleep 2
  done
  echo "Webmention notifications completed"
```

### Step 7.5: Understanding the Workflow

**When you publish a new post (e.g., dated April 6, 2025):**

1. **GitLab CI Build Stage:**
   - Hugo builds your site
   - Post is rendered with h-entry microformats
   - Because date ≥ December 1, 2025, hidden `u-syndication` links are included in HTML

2. **GitLab CI Deploy Stage:**
   - Site deployed via rsync
   - After deployment, script fetches recent posts from RSS feed
   - Sends individual webmentions to `https://fed.brid.gy/webmention` for each post
   - Each webmention includes `source` (post URL) and `target` (https://fed.brid.gy/)

3. **Bridgy Fed Processing** (5-10 minutes):
   - Bridgy Fed receives webmention for each post URL
   - Fetches the post HTML and parses microformats
   - Finds posts with `u-syndication` links pointing to fed.brid.gy
   - Creates Mastodon post at your Fediverse identity
   - Creates Bluesky post via bsky.brid.gy
   - Both posts link back to your canonical URL

4. **Result:**
   - Post appears on Mastodon
   - Post appears on Bluesky
   - Replies/likes/boosts flow back as webmentions
   - Displayed on your blog via webmentions partial

**For historic posts (before December 2025):**
- No `u-syndication` links in HTML
- Bridgy Fed ignores them
- They don't get published to social networks
- But still accept webmentions if someone manually shares them

### Step 7.6: Testing POSSE Setup

**Test 1: Verify Syndication Links in HTML**

1. Build locally:
   ```bash
   hugo server
   ```

2. Visit a NEW post (dated ≥ December 2025)

3. View page source, look for:
   ```html
   <div style="display:none;">
     <a class="u-syndication" href="https://fed.brid.gy/" rel="syndication">Mastodon</a>
     <a class="u-syndication" href="https://bsky.brid.gy/" rel="syndication">Bluesky</a>
   </div>
   ```

4. Visit an OLD post (before December 2025)

5. Verify the syndication div is **NOT** present

**Test 2: Validate Microformats**

Visit https://microformats.io/ and enter your post URL:
- Should detect h-entry
- Should show u-syndication properties for new posts

**Test 3: Manual Webmention Test (Before CI Setup)**

Before deploying, test the webmention endpoint manually:

```bash
# Test sending a webmention for your latest post
curl -v -X POST https://fed.brid.gy/webmention \
  --data-urlencode "source=https://yourdomain.com/posts/your-post/" \
  --data-urlencode "target=https://fed.brid.gy/"

# Expected successful response:
# "Added webmention [large-number-ID] now"

# Wrong responses indicate problems:
# "404 Not Found" = wrong endpoint
# "405 Method Not Allowed" = wrong HTTP method or endpoint
# HTML error page = missing parameters or malformed request
```

If successful, check https://fed.brid.gy/web/yourdomain.com for activity.

**Test 4: Check GitLab CI Logs**

After pushing:
1. Go to GitLab → CI/CD → Pipelines
2. Open the latest pipeline → deploy job
3. Look for:
   ```
   Notifying Bridgy Fed of new content...
   Fetching recent posts from RSS feed...
   Sending webmention for: https://yourdomain.com/posts/...
   Response: Added webmention [ID] now
   ```
4. Verify:
   - Multiple posts listed (not just one)
   - Each shows "Added webmention [ID] now"
   - No "404 Not Found" errors
   - No "405 Method Not Allowed" errors

**Test 5: Monitor Bridgy Fed**

After deployment:
1. Wait 5-10 minutes
2. Visit https://fed.brid.gy/web/yourdomain.com
3. Check activity log for recent publications
4. Verify posts appeared on Mastodon and Bluesky

### Step 7.7: Adjusting the Date Cutoff

The cutoff is set to **December 1, 2025** in `single.html`:

```go
{{ $cutoffDate := time.AsTime "2025-12-01T00:00:00Z" }}
```

**To change the cutoff date:**
- Edit `layouts/_default/single.html`
- Update the date string (format: `YYYY-MM-DDTHH:MM:SSZ`)
- Example for January 1, 2026: `"2026-01-01T00:00:00Z"`

**To disable date checking (syndicate all posts):**

Replace:
```go
{{ $cutoffDate := time.AsTime "2025-12-01T00:00:00Z" }}
{{ if ge .Date.Unix $cutoffDate.Unix }}
```

With:
```go
{{ if true }}
```

**Warning:** This will syndicate ALL posts, including historic ones, when Bridgy Fed next crawls your site.

### Step 7.8: Per-Post Syndication Control (Optional)

To allow manual control via front matter:

**1. Update `single.html`:**

Replace:
```go
{{ $cutoffDate := time.AsTime "2025-12-01T00:00:00Z" }}
{{ if ge .Date.Unix $cutoffDate.Unix }}
```

With:
```go
{{ $cutoffDate := time.AsTime "2025-12-01T00:00:00Z" }}
{{ $shouldSyndicate := true }}
{{ if isset .Params "syndicate" }}
  {{ $shouldSyndicate = .Params.syndicate }}
{{ end }}
{{ if and (ge .Date.Unix $cutoffDate.Unix) $shouldSyndicate }}
```

**2. In post front matter:**

```yaml
---
title: "My Post Title"
date: 2026-01-15
syndicate: false  # Don't auto-publish this specific post
---
```

Or:
```yaml
syndicate: true   # Force syndication even if before cutoff date
```

### Step 7.9: Troubleshooting POSSE

**Issue: Posts not appearing on Mastodon/Bluesky**

**Check 1: Post date**
- View HTML source of the post
- Look for `<div style="display:none;">` with `u-syndication` links
- If missing, post is before December 2025 cutoff

**Check 2: GitLab CI notification**
- Check CI logs for "Notifying Bridgy Fed of new content..."
- If missing, verify curl is in apk install line
- Check for curl errors in output
- **CRITICAL**: Look for "404 Not Found" or "405 Method Not Allowed" errors
  - These indicate you're using the wrong endpoint
  - Correct endpoint: `https://fed.brid.gy/webmention`
  - Wrong endpoints: `/publish/webmention`, `/` without path, or just homepage POST
- **CRITICAL**: Verify webmentions are sent for individual post URLs
  - Look for log lines like "Sending webmention for: https://yourdomain.com/posts/..."
  - Should see one webmention per post, not just one for homepage
  - Response should be "Added webmention [ID] now"
  - If you see "404 Not Found", you're using the wrong endpoint

**Check 3: Bridgy Fed status**
- Visit https://fed.brid.gy/web/yourdomain.com
- Check for errors or warnings
- Verify domain is connected and authorized

**Check 4: Microformats**
- Validate at https://microformats.io/
- Ensure h-entry, p-name, e-content, dt-published are present
- Verify u-syndication links are inside the h-entry article tag

**Check 5: Bridgy Fed authorization**
- Visit https://fed.brid.gy/
- Re-authorize Mastodon and Bluesky if needed
- Check that publish permissions are granted

**Issue: Posts stopped syndicating after initial setup worked**

This is the most common issue. If posts initially syndicated but stopped working:

**Diagnosis:**
1. Visit https://fed.brid.gy/web/yourdomain.com
2. Check the activity log - are new posts showing up?
3. Check if Bridgy Fed switched from RSS polling to webmention mode

**Root cause:**
- You sent a webmention at some point (manually or via CI)
- Bridgy Fed switched from RSS polling to webmention-only mode
- Your CI/CD stopped sending webmentions (or never was set up correctly)
- New posts are published but Bridgy Fed never knows about them

**Solution:**
1. **Verify CI/CD is sending webmentions:**
   ```bash
   # Check latest pipeline logs
   # Look for: "Sending webmention for: https://yourdomain.com/posts/..."
   # Should see response: "Added webmention [ID] now"
   ```

2. **Manually send webmentions for missed posts:**
   ```bash
   # For each post that was missed:
   curl -X POST https://fed.brid.gy/webmention \
     --data-urlencode "source=https://yourdomain.com/posts/my-post/" \
     --data-urlencode "target=https://fed.brid.gy/"
   ```

3. **Fix your CI/CD configuration:**
   - Ensure you're using the correct endpoint: `https://fed.brid.gy/webmention`
   - Ensure you're sending individual post URLs (not just homepage)
   - Ensure `libxml2-utils` is installed for RSS parsing
   - Check for 404 or 405 errors in logs

**Prevention:**
- Always include the webmention automation in CI/CD from the start
- Test webmentions in CI logs after every deployment
- Monitor Bridgy Fed activity log weekly

**Issue: Historic posts accidentally syndicated**

If you accidentally syndicated old posts:
- Delete them from Mastodon/Bluesky manually
- They won't re-appear unless you explicitly re-ping Bridgy with those URLs

To prevent future accidents:
- Test locally first (`hugo server`)
- Check HTML source before pushing
- Use per-post `syndicate: false` flag for sensitive posts

**Issue: Only syndicated to one network**

If posts appear on Mastodon but not Bluesky (or vice versa):
- Check Bridgy Fed authorization for both networks
- Verify both u-syndication links are in HTML
- Check https://fed.brid.gy/ logs for errors
- Bluesky requires a valid DID and handle setup

### Step 7.10: Advanced Configuration

**Syndicate only to Mastodon:**

Remove the Bluesky link in `single.html`:
```html
<div style="display:none;">
  <a class="u-syndication" href="https://fed.brid.gy/" rel="syndication">Mastodon</a>
  {{/* Bluesky removed */}}
</div>
```

**Syndicate only to Bluesky:**

Remove the Mastodon link:
```html
<div style="display:none;">
  {{/* Mastodon removed */}}
  <a class="u-syndication" href="https://bsky.brid.gy/" rel="syndication">Bluesky</a>
</div>
```

**Add more networks:**

Bridgy Fed supports other networks. Add additional links:
```html
<a class="u-syndication" href="https://fed.brid.gy/" rel="syndication">Mastodon</a>
<a class="u-syndication" href="https://bsky.brid.gy/" rel="syndication">Bluesky</a>
<a class="u-syndication" href="https://pixelfed.brid.gy/" rel="syndication">Pixelfed</a>
```

**Customize syndicated content:**

To customize what appears on social networks, Bridgy Fed uses:
- Post title (p-name) → becomes social post title
- Post summary (p-summary) → becomes excerpt if available
- Post content (e-content) → fallback content
- Featured image → attached if present

Add to front matter:
```yaml
summary: "This custom summary will appear on Mastodon/Bluesky"
```

---

## Phase 7 Summary

You've now configured POSSE auto-publishing:

✅ **Date-based syndication** protects historic content
✅ **GitLab CI automation** handles notifications
✅ **Bridgy Fed integration** publishes to Mastodon & Bluesky
✅ **Webmentions flow back** from social interactions
✅ **Your site remains canonical** source of truth

### Files Modified:

1. **`layouts/_default/single.html`**
   - Added date-checking logic
   - Added hidden u-syndication links for new posts

2. **`.gitlab-ci.yml`**
   - Added curl to dependencies
   - Added Bridgy Fed notification after deployment

3. **`layouts/_partials/head-additions.html`**
   - Added rel="me" link for personal Mastodon

### Next Steps:

1. **Commit and push your changes**
2. **Publish a new test post** (dated December 2025 or later)
3. **Monitor GitLab CI** for successful notification
4. **Check Bridgy Fed** for syndication activity
5. **Verify posts appear** on Mastodon and Bluesky

---

## Complete Implementation Checklist

Use this checklist to verify your complete webmention + POSSE setup:

### Webmention Setup
- [x] Webmention.io account configured
- [x] Webmention endpoint in `<head>`
- [x] Microformats (h-entry, h-card) on all posts
- [x] Webmentions partial displays interactions
- [x] GitLab CI or local cron fetches webmentions
- [x] CSS styling for webmentions

### Bridgy Fed Setup
- [x] Domain verified at https://fed.brid.gy/
- [x] Mastodon authorized
- [x] Bluesky authorized
- [x] Fediverse identity confirmed (e.g., @yourdomain.com@yourdomain.com)

### POSSE Setup
- [x] u-syndication links in post template
- [x] Date-based syndication control (December 2025+)
- [x] GitLab CI pings Bridgy Fed after deployment
- [x] rel="me" links for IndieAuth
- [x] Tested with new post

### Testing & Validation
- [ ] Microformats validate at microformats.io
- [ ] IndieAuth works at webmention.io
- [ ] GitLab CI notification succeeds
- [ ] New post appears on Mastodon
- [ ] New post appears on Bluesky
- [ ] Historic posts don't get syndicated
- [ ] Webmentions display correctly

---

**Your Hugo blog is now a fully IndieWeb-enabled, POSSE-capable website!** 🎉🚀
