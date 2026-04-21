---
name: loom-brand-generation
description: Generate complete brand systems from brief, existing site, or reference URLs — colors, typography, voice, tokens, Tailwind config
---

# Brand Generation Engine

Create complete, production-ready brand systems. Three input modes, same deliverables every time.

## When to Use

- Jesse provides a brand brief (business name + adjectives + audience)
- Jesse says "migrate [URL]" or "clone [URL]" (extract brand from existing site)
- Jesse provides reference URLs ("I like the feel of these sites")
- Starting a new project that doesn't have an existing brand
- Refreshing or evolving an existing brand

## Mode 1 — Brand from Brief

**Input:** Business name, industry, target audience, 3-5 adjectives (e.g. "bold, minimal, techy, trustworthy"), color preferences or avoidances.

**Process:**

1. **Color palette generation:**
   - Primary color: anchors the brand, used for CTAs and key UI
   - Secondary color: complementary, used for supporting elements
   - Accent color: high-contrast pop for highlights, badges, alerts
   - Semantic colors: success (#green), warning (#amber), error (#red), info (#blue)
   - Background scale: page bg, card bg, elevated bg (3 levels)
   - Border/divider: subtle separation colors
   - Text scale: heading, body, muted, disabled (4 levels)
   - Generate BOTH light mode AND dark mode variants
   - Every color pair must pass WCAG AA contrast (4.5:1 body text, 3:1 large text)

2. **Color psychology rationale:** For each primary/secondary/accent choice, explain WHY this color works for this audience and industry. Reference color associations (trust, energy, luxury, approachability).

3. **Typography system:**
   - Heading font: personality carrier (display or strong sans/serif)
   - Body font: readability workhorse (neutral sans or readable serif)
   - Mono font: code, data, technical content
   - Prefer Google Fonts for web performance (free, CDN-cached)
   - Size scale: xs (12), sm (14), base (16), lg (18), xl (20), 2xl (24), 3xl (30), 4xl (36), 5xl (48), 6xl (60) — in px, convert to rem
   - Weight usage: which weights for headings, body, emphasis, captions
   - Letter-spacing rules: tight for large headings, normal for body, wide for small caps/labels
   - Line-height: tight (1.2) for headings, relaxed (1.6-1.75) for body

4. **Voice and tone guidelines:**
   - How the brand speaks in headlines (punchy? authoritative? warm?)
   - How the brand speaks in body copy (conversational? formal? technical?)
   - How the brand speaks in CTAs (urgent? inviting? confident?)
   - How the brand handles error messages (apologetic? matter-of-fact? humorous?)
   - 3 example sentences in each register
   - Words to use / words to avoid

5. **Component aesthetic direction:**
   - Border radius: sharp (0-2px), soft (4-8px), rounded (12-16px), pill (9999px)
   - Shadow depth: flat (none), subtle (sm), layered (md+lg), dramatic (xl+2xl)
   - Spacing scale: tight (4-8px base), standard (8-16px base), generous (16-24px base)
   - Density: compact (high info density), balanced, spacious (breathing room)
   - Animation preference: none, subtle (fade/slide), expressive (spring/bounce)

6. **Logo brief:** Describe what the logo mark should communicate, geometric direction (angular/organic/geometric/abstract), color usage rules (which brand colors, minimum size, clear space). This is a creative brief for Jesse to use with AI image tools or a designer — Loom does NOT generate the logo itself.

## Mode 2 — Brand from Existing Site

**Input:** A URL to crawl.

**Process:**

1. **Crawl with `agent-browser`:**
   ```bash
   agent-browser open {url}
   agent-browser screenshot --full-page -o brand/screenshots/homepage-desktop.png
   agent-browser viewport 768 1024
   agent-browser screenshot --full-page -o brand/screenshots/homepage-tablet.png
   agent-browser viewport 375 812
   agent-browser screenshot --full-page -o brand/screenshots/homepage-mobile.png
   ```
   Repeat for every major page (about, services, contact, pricing).

2. **Extract colors:** Use `agent-browser` to read computed styles from key elements:
   ```bash
   agent-browser eval "getComputedStyle(document.querySelector('header')).backgroundColor"
   agent-browser eval "getComputedStyle(document.querySelector('a.cta, .btn-primary')).backgroundColor"
   agent-browser eval "getComputedStyle(document.body).color"
   agent-browser eval "getComputedStyle(document.body).backgroundColor"
   ```
   Sample 10-15 key elements. Build palette from dominant colors.

3. **Extract typography:**
   ```bash
   agent-browser eval "getComputedStyle(document.querySelector('h1')).fontFamily"
   agent-browser eval "getComputedStyle(document.querySelector('p')).fontFamily"
   agent-browser eval "getComputedStyle(document.querySelector('h1')).fontSize"
   ```
   Identify heading font, body font, size scale.

4. **Extract layout patterns:** Grid structure, spacing rhythm, component patterns, nav style.

5. **Download assets:** Save all accessible images, logos, and icons to `brand/logos/` and `brand/photos/`.

6. **Capture content:** Extract all text content page by page. Map full site structure.

7. **Generate migration report:** What they have, what to keep, what to improve. Save to `design/migration-report.md`.

8. **Generate brand system** from extracted tokens — not a pixel-perfect clone, a faithful evolution.

## Mode 3 — Brand from Reference

**Input:** 2-3 reference URLs + notes from Jesse on what he likes about each.

**Process:**

1. Crawl all references using Mode 2 extraction process.
2. For each reference, document: color approach, typography feel, layout density, animation style, component patterns.
3. Identify the common design DNA across references.
4. Synthesize a NEW brand that captures the energy without copying any single reference.
5. Generate full brand system with rationale explaining which reference inspired which choices.

## Deliverables (Every Mode)

Save all files to the project's `brand/` directory:

| File | Content |
|------|---------|
| `brand.config.json` | Machine-readable tokens — all colors (hex + HSL), fonts, sizes, spacing, radii, shadows |
| `tailwind.tokens.js` | Export that extends Tailwind config with brand tokens |
| `palette.md` | Color system documentation with swatches, contrast ratios, usage rules, light/dark |
| `typography.md` | Font stack, size scale table, weight usage, letter-spacing, line-height |
| `voice.md` | Brand voice guide with examples per register |
| `logo-brief.md` | Logo creative brief for Jesse (Mode 1 only) |

**brand.config.json structure:**
```json
{
  "name": "Project Name",
  "colors": {
    "light": {
      "primary": "#hex", "secondary": "#hex", "accent": "#hex",
      "background": { "page": "#hex", "card": "#hex", "elevated": "#hex" },
      "text": { "heading": "#hex", "body": "#hex", "muted": "#hex" },
      "border": "#hex",
      "success": "#hex", "warning": "#hex", "error": "#hex", "info": "#hex"
    },
    "dark": { }
  },
  "typography": {
    "heading": { "family": "Font Name", "weights": [600, 700] },
    "body": { "family": "Font Name", "weights": [400, 500] },
    "mono": { "family": "Font Name", "weights": [400] },
    "scale": { "xs": "0.75rem", "sm": "0.875rem", "base": "1rem" }
  },
  "spacing": { "base": "1rem", "scale": [0.25, 0.5, 0.75, 1, 1.5, 2, 3, 4, 6, 8] },
  "radius": { "sm": "4px", "md": "8px", "lg": "16px", "full": "9999px" },
  "shadows": { "sm": "...", "md": "...", "lg": "..." }
}
```

**tailwind.tokens.js structure:**
```js
// Generated from brand.config.json — do not edit manually
export default {
  theme: {
    extend: {
      colors: { /* from brand.config.json colors */ },
      fontFamily: { /* from brand.config.json typography */ },
      borderRadius: { /* from brand.config.json radius */ },
      boxShadow: { /* from brand.config.json shadows */ },
    }
  }
}
```

## Quality Checklist

- [ ] Every color pair passes WCAG AA contrast (4.5:1 body, 3:1 large text) — verify with contrast checker
- [ ] Both light AND dark mode palettes generated
- [ ] Font choices are Google Fonts or system fonts (no licensing issues)
- [ ] brand.config.json is valid JSON and parseable
- [ ] tailwind.tokens.js imports cleanly into a Tailwind config
- [ ] palette.md includes usage rules (when to use primary vs secondary vs accent)
- [ ] typography.md includes the full size scale with rem values
- [ ] voice.md includes concrete examples, not just adjectives
- [ ] Logo brief describes direction without prescribing a specific design
- [ ] No colors or typography borrowed from a different project's brand system
