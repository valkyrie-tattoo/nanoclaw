---
name: loom-brand-systems
description: Reference brand tokens for known brands (ARCHON, Valkyrie Tattoo) — load on demand when working on these projects
---

# Known Brand Systems

Reference data for existing brands. Load ONLY when Jesse assigns a project for that brand. Never assume as default context.

## When to Use

- Jesse says "work on the ARCHON site" or references Valkyrie Systems web properties
- Jesse says "work on the Valkyrie Tattoo site" or references the studio's web presence
- Building or updating a site that uses one of these established brands
- Need to verify brand token accuracy before applying to a project

Do NOT load for new/unrelated projects — those get a fresh brand via `loom-brand-generation`.

## ARCHON (Valkyrie Systems)

AI operations platform brand. Sells to SMB service businesses.

**Light mode (primary — PDFs, proposals, daytime UI):**

| Token | Hex | Usage |
|-------|-----|-------|
| Canvas | #FFFFFF | Page background |
| Authority (primary) | #7C3AED | CTAs, primary actions, key UI |
| Accent | #A855F7 | Highlights, badges, hover states |
| Text | #111111 | Headings, body |
| Secondary | #3D3852 | Supporting text, subheadings |
| Muted | #6B6080 | Captions, placeholders |
| Money | #0A7B4F | Revenue, savings, positive metrics |
| Tech | #00C2FF | Technical callouts, data viz |
| Border | #E0DBE8 | Dividers, card borders |
| Card | #F8F6FC | Card backgrounds, elevated surfaces |

**Dark mode (secondary — website, boot screen, dashboards):**

| Token | Hex | Usage |
|-------|-----|-------|
| Void | #08060F | Page background |
| Surface | #110E1A | Cards, elevated surfaces |
| Authority | #A855F7 | Primary actions (same violet) |
| Text | #E8E4F0 | Headings, body |
| Secondary | #9B8FB8 | Supporting text |
| Money | #00D68F | Revenue, positive metrics |
| Border | #2A2438 | Dividers, card borders |

**Typography:**
- Heading: Inter (geometric sans-serif), weights 600-700
- Body: Inter, weights 400-500
- Mono: JetBrains Mono or similar geometric mono
- ARCHON wordmark: uppercase, letter-spacing 0.15em minimum (10px+ at display sizes)
- Numbers: tabular (monospace) alignment in tables

**Design rules:**
- 3px violet top-border on cards
- One dark closing card per document (void background, light text — gut-punch contrast)
- Generous whitespace — never cramped
- Border radius: 8px default

**Voice:** Authoritative, precise, data-driven, restrained confidence. Never hype. Let numbers speak.

**Logo:** Geometric angular crown/chevron mark. Standalone from text. Use only provided logo files — do not recreate.

**Taglines (use contextually):**
- "It never satisfies. It never sleeps. It never stops getting better."
- "Your AI operations platform"
- "Built different. Runs relentless."

## VALKYRIE TATTOO & PIERCING

Charlotte NC tattoo and piercing studio. Owner: Jesse Chatfield.

**Palette:**

| Token | Hex | Usage |
|-------|-----|-------|
| Gold (primary) | #B8944E | Accents, CTAs, brand mark |
| Dark bg | #0F0E0C | Page background |
| Warm gray | #2A2418 | Card backgrounds, surfaces |
| Text | #C8B8A0 | Body text, headings |
| Light accent | #E8D5B0 | Highlights, hover states |

Dark-first design — this brand lives on dark backgrounds. Gold is the accent, not the background.

**Typography:**
- Heading: Georgia (serif), weight 700
- Body: Calibri or system sans-serif, weight 400
- Norse/runic aesthetic — angular, crafted feel
- Rune mark: ᛡ (used as brand icon, section dividers)

**Voice:** Warm, crafted, artisanal, personal. Jesse's own voice — direct, real, no corporate polish. Speaks like a skilled tradesperson who cares about the work.

**Design rules:**
- Dark backgrounds, warm tones
- Gold used sparingly — accents and highlights, never overwhelming
- Photography-forward — the work speaks for itself
- Subtle texture (leather, metal, parchment) where appropriate
- Booking/CTA prominence — the site drives appointments

## CRITICAL RULES

1. These two brands are completely separate. NEVER mix tokens, voice, or design elements between them.
2. These are reference data — not defaults. New projects start fresh unless Jesse explicitly assigns one of these brands.
3. If a brand's tokens have been updated in its actual `brand.config.json` project file, that project file is the source of truth — this skill is the canonical reference but project overrides take precedence.
4. When applying these tokens, generate the full `brand.config.json` and `tailwind.tokens.js` per the `loom-brand-generation` deliverables format — don't shortcut.
