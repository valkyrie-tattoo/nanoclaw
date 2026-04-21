---
name: loom-qa
description: Full QA pipeline — Lighthouse audits, accessibility testing, visual regression, link/form validation, structured data checks, GEO scoring
---

# QA Pipeline

Comprehensive quality assurance for every project before production deploy. Run during Phase 4 of the workflow.

## When to Use

- Phase 4 (QA) of any project
- After significant changes to a deployed site
- Jesse requests a site audit
- Pre-deploy verification after staging build

## Performance Audit — Lighthouse

Run Lighthouse via `agent-browser` on every page:

```bash
# Navigate to the page
agent-browser open {url}
agent-browser wait --load networkidle

# Run Lighthouse (if available in environment)
# Alternative: use Chrome DevTools Protocol via agent-browser
agent-browser eval "
  const resp = await fetch('{url}');
  const headers = Object.fromEntries(resp.headers.entries());
  JSON.stringify(headers);
"
```

**For local dev server audits, use npx:**
```bash
npx lighthouse {url} --output=json --output-path=.lighthouse/report-{page}.json --chrome-flags="--headless --no-sandbox"
```

**For full-site crawl audit:**
```bash
npx unlighthouse --site {url} --output-path .lighthouse/full-site/
```

**Thresholds (non-negotiable):**

| Category | Minimum | Target |
|----------|---------|--------|
| Performance | 90 | 95+ |
| Accessibility | 95 | 100 |
| Best Practices | 95 | 100 |
| SEO | 95 | 100 |

**Record scores** in `.lighthouse/scores.md`:
```markdown
# Lighthouse Scores — {date}

| Page | Perf | A11y | BP | SEO | Notes |
|------|------|------|----|-----|-------|
| / | 97 | 100 | 100 | 100 | — |
| /about | 94 | 100 | 100 | 100 | Large hero image |
```

**Common performance fixes:**
- Image not in WebP/AVIF → run through `loom-image-pipeline`
- Missing lazy loading → add `loading="lazy"` to below-fold images
- Render-blocking CSS → inline critical CSS, async the rest
- Large JS bundle → code-split, lazy-load components
- Missing font-display → add `font-display: swap` to @font-face
- Layout shift → set explicit width/height on images and embeds

## Accessibility Audit

**Automated (axe-core):**
```bash
cd /workspace/group/projects/{slug}
npm install @axe-core/cli --save-dev 2>/dev/null
npx axe {url} --save .lighthouse/axe-report.json
```

**Manual checks via `agent-browser`:**

```bash
# Keyboard navigation test
agent-browser open {url}
agent-browser keyboard Tab
agent-browser snapshot -i  # Check focus indicator visible
agent-browser keyboard Tab  # Repeat through all interactive elements

# Check heading hierarchy
agent-browser eval "JSON.stringify([...document.querySelectorAll('h1,h2,h3,h4,h5,h6')].map(h => ({level: h.tagName, text: h.textContent.trim()})))"

# Check all images have alt text
agent-browser eval "[...document.querySelectorAll('img')].filter(i => !i.alt).map(i => i.src)"

# Check skip-to-content link
agent-browser eval "document.querySelector('a[href=\"#main-content\"], a[href=\"#content\"], .skip-link')?.textContent"

# Check color contrast (sample key elements)
agent-browser eval "getComputedStyle(document.querySelector('p')).color + ' on ' + getComputedStyle(document.querySelector('p')).backgroundColor"
```

**Accessibility requirements (non-negotiable):**
- WCAG 2.1 AA minimum on every project
- Zero axe-core violations (critical and serious)
- Full keyboard navigation — every interactive element reachable via Tab
- Focus indicators visible and branded (never `outline: none` without replacement)
- Skip-to-content link on every page
- All images have descriptive alt text
- All form inputs have associated labels
- ARIA attributes on custom interactive elements
- Color contrast 4.5:1 minimum for body text, 3:1 for large text
- `prefers-reduced-motion` respected — no animation without media query guard

## Visual Regression Testing

Screenshot every page at all breakpoints. Compare before/after on changes.

```bash
# Capture baseline screenshots
for page in / /about /services /contact /pricing; do
  agent-browser open "{base_url}${page}"
  agent-browser wait --load networkidle

  for viewport in "1440 900" "1024 768" "768 1024" "375 812"; do
    w=$(echo $viewport | cut -d' ' -f1)
    h=$(echo $viewport | cut -d' ' -f2)
    agent-browser viewport $w $h
    slug=$(echo $page | tr '/' '-' | sed 's/^-/home/')
    agent-browser screenshot --full-page -o .lighthouse/visual/${slug}-${w}.png
  done
done
```

**Review:** Open each screenshot. Check for:
- Layout breaks at any viewport
- Text overflow or truncation
- Images not loading or wrong aspect ratio
- Navigation collapse working correctly on mobile
- Footer alignment and content
- Form layout and label placement

## Link and Form Testing

**Link testing:**
```bash
agent-browser open {url}
agent-browser eval "JSON.stringify([...document.querySelectorAll('a')].map(a => ({text: a.textContent.trim(), href: a.href, target: a.target})))"
```

Test every internal link — navigate, verify page loads, check for 404s.
Test every external link — verify it doesn't point to dead URLs.

**Form testing:**
```bash
# Find all forms
agent-browser eval "[...document.querySelectorAll('form')].map(f => ({action: f.action, method: f.method, inputs: [...f.querySelectorAll('input,textarea,select')].map(i => i.name)}))"

# Test form submission
agent-browser snapshot -i
agent-browser type @e1 "Test Name"
agent-browser type @e2 "test@example.com"
agent-browser type @e3 "Test message"
agent-browser click @submit-button
# Verify success state
agent-browser snapshot -i
```

Test both valid submissions and validation states (empty required fields, invalid email, etc.).

## Structured Data Validation

```bash
# Extract all JSON-LD from page
agent-browser eval "JSON.stringify([...document.querySelectorAll('script[type=\"application/ld+json\"]')].map(s => JSON.parse(s.textContent)))"
```

For each page, verify:
- JSON-LD is valid JSON (no parse errors)
- `@type` matches page content (FAQPage on FAQ sections, LocalBusiness on location pages)
- Required fields populated (no empty strings, no placeholder values)
- URLs are absolute (not relative)
- Images referenced in schema actually exist

## GEO Content Scoring

Run `loom-geo` content scoring checklist on every key page. Score must be 7+ before deploy.

## QA Report

Generate a summary report at `.lighthouse/qa-report.md`:

```markdown
# QA Report — {Project Name} — {date}

## Summary
| Check | Status | Notes |
|-------|--------|-------|
| Lighthouse Performance | PASS (97 avg) | All pages 95+ |
| Lighthouse Accessibility | PASS (100) | All pages 100 |
| Lighthouse Best Practices | PASS (100) | — |
| Lighthouse SEO | PASS (100) | — |
| axe-core | PASS | Zero violations |
| Keyboard navigation | PASS | All elements reachable |
| Visual regression | PASS | 4 viewports × N pages |
| Links | PASS | 0 broken |
| Forms | PASS | Validation + submission working |
| Structured data | PASS | Valid on all pages |
| GEO scoring | PASS | All key pages 7+ |

## Issues Found
[list any issues and their resolution]

## Screenshots
[reference .lighthouse/visual/ directory]
```

## Quality Checklist

- [ ] Lighthouse scores meet thresholds on every page
- [ ] axe-core reports zero critical/serious violations
- [ ] Full keyboard navigation verified
- [ ] Focus indicators visible on all interactive elements
- [ ] Skip-to-content link present and functional
- [ ] All images have alt text
- [ ] Visual regression screenshots captured at 4 viewports for all pages
- [ ] All internal links working (no 404s)
- [ ] All forms submit correctly with proper validation
- [ ] JSON-LD structured data valid on every page
- [ ] GEO content score 7+ on every key page
- [ ] QA report generated and saved to `.lighthouse/qa-report.md`
