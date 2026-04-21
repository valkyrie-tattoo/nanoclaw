---
name: loom-site-migration
description: Migrate existing websites — crawl, extract content and brand, plan restructure, rebuild on modern stack
---

# Site Migration Engine

Full pipeline for migrating an existing site to a modern, optimized rebuild.

## When to Use

- Jesse says "migrate [URL]", "clone [URL]", or "rebuild [URL]"
- Starting a project where an existing site needs replacement
- Extracting content from a site Jesse is leaving (Wix, Squarespace, Vagaro, etc.)

## Step 1 — Crawl

Spider the site with `agent-browser`. Capture everything.

**Homepage crawl:**
```bash
agent-browser open {url}
agent-browser wait --load networkidle

# Desktop screenshot
agent-browser screenshot --full-page -o design/migration/screenshots/home-1440.png

# Tablet
agent-browser viewport 768 1024
agent-browser screenshot --full-page -o design/migration/screenshots/home-768.png

# Mobile
agent-browser viewport 375 812
agent-browser screenshot --full-page -o design/migration/screenshots/home-375.png

# Reset
agent-browser viewport 1440 900
```

**Extract navigation structure:**
```bash
agent-browser snapshot -i
# Identify all nav links, build site map
agent-browser eval "JSON.stringify([...document.querySelectorAll('nav a')].map(a => ({text: a.textContent.trim(), href: a.href})))"
```

**Extract page content:**
```bash
# Get main content text
agent-browser eval "document.querySelector('main, article, .content, #content, body').innerText"
```

**Extract computed styles:**
```bash
agent-browser eval "JSON.stringify({
  bgColor: getComputedStyle(document.body).backgroundColor,
  textColor: getComputedStyle(document.body).color,
  h1Font: getComputedStyle(document.querySelector('h1')).fontFamily,
  h1Size: getComputedStyle(document.querySelector('h1')).fontSize,
  bodyFont: getComputedStyle(document.querySelector('p')).fontFamily,
  linkColor: getComputedStyle(document.querySelector('a')).color
})"
```

**Crawl depth:** Follow every internal link from homepage. For large sites (50+ pages), stop at depth 3 and flag remaining pages.

**Repeat for every major page:** About, Services (each), Contact, Pricing, Gallery/Portfolio, Blog (index + sample posts), any other linked pages.

**Save all crawl data** to `design/migration/`:
```
design/migration/
  screenshots/              — all pages at all breakpoints
  content/                  — text content per page (page-slug.md)
  assets/                   — downloaded images, logos, PDFs
  site-map.md               — URL → title → content type
  extracted-styles.json     — computed CSS values
```

## Step 2 — Extract & Catalog

1. **Organize assets:** Move images to `brand/photos/`, logos to `brand/logos/`, icons to `brand/icons/`.

2. **Generate brand system:** Feed extracted styles into `loom-brand-generation` Mode 2. Produce `brand.config.json` and all brand deliverables.

3. **Create content inventory:** Page-by-page list of all text content:

```markdown
# Content Inventory

| Page | URL | Content Type | Word Count | Status | Notes |
|------|-----|-------------|------------|--------|-------|
| Home | / | Landing | 450 | Migrate as-is | Hero needs rewrite |
| About | /about | Bio | 320 | Migrate + update | Outdated headshot |
| Services | /services | Listing | 280 | Rewrite | Missing pricing |
```

4. **Flag content quality:**
   - **Migrate as-is**: Content is good, just needs formatting
   - **Migrate + update**: Content is mostly good, needs minor edits
   - **Rewrite**: Content is outdated, vague, or off-brand — flag for Shield or Jesse
   - **Drop**: Redundant pages, placeholder content, outdated promos

## Step 3 — Plan

Generate a migration plan document for Jesse's approval before building.

**Migration plan template (`design/migration/plan.md`):**

```markdown
# Migration Plan — {Project Name}

## Source: {original URL}
## Target: {new framework} on {deploy target}

## Site Structure

### Keeping (mapped)
| Original Page | New Page | Notes |
|--------------|----------|-------|
| /about | /about | Content updated |
| /services | /services | Split into individual pages |

### Adding (new pages)
| New Page | Reason |
|----------|--------|
| /faq | GEO — AI citation opportunity |
| /pricing | GEO — AI cites prices |

### Dropping
| Original Page | Reason |
|--------------|--------|
| /blog/2019-sale | Outdated promo |

## URL Redirects (SEO preservation)
| Old URL | New URL | Type |
|---------|---------|------|
| /old-path | /new-path | 301 |

## Design Changes
- [list improvements over original]

## GEO Opportunities
- [content gaps the original missed]

## Content Needs
- [pages that need copy from Shield or Jesse]

## Timeline Estimate
- Phase 2 (Design System): X days
- Phase 3 (Build): X days
- Phase 4 (QA): X days
```

**Jesse must approve this plan before Phase 4 (Build) begins.**

## Step 4 — Build

Execute via standard 6-phase workflow starting at Phase 2 (Design System). The migration plan informs what to build.

**Critical migration rules:**
- Preserve URL structure where possible for SEO (301 redirects for any changed URLs)
- All original content must land somewhere — nothing silently dropped
- Every image from the original goes through `loom-image-pipeline`
- All new pages get `loom-geo` treatment (structured data, semantic HTML, meta)

## Quality Checklist

- [ ] Every page from the original site is accounted for (migrated, merged, or dropped with reason)
- [ ] Content inventory complete with status for every page
- [ ] Screenshots captured at all breakpoints for all original pages
- [ ] All accessible images and media downloaded
- [ ] Brand system extracted and documented (brand.config.json generated)
- [ ] Migration plan approved by Jesse before build begins
- [ ] URL redirects mapped for any changed paths
- [ ] No content silently dropped — every piece has a destination or documented removal reason
- [ ] All migrated images processed through image pipeline
