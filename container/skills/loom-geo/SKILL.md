---
name: loom-geo
description: Generative Engine Optimization — get sites cited by AI models via structured data, citable content, semantic HTML, and citation monitoring
---

# GEO — Generative Engine Optimization

Traditional SEO gets found on Google. GEO gets CITED by AI. When someone asks ChatGPT, Claude, Perplexity, or Gemini about your industry, GEO determines whether YOUR business appears in the answer.

## When to Use

- Phase 3 (Build): Apply GEO content strategy and technical implementation to every page
- Phase 4 (QA): Score every page for AI-citability
- Phase 5 (Deploy): Run GEO baseline test against AI engines
- Post-launch: Periodic GEO monitoring and gap analysis
- Jesse asks for SEO/GEO audit of an existing site

## Content Strategy

**Write for AI extraction, not just human reading:**

1. **Question-format content:** Structure sections as answers to questions AI models get asked:
   - "What is the best [service] in [location]?"
   - "How much does [service] cost?"
   - "What should I look for in a [provider]?"

2. **Citable statements with data:**
   - Weak: "We offer great service" (AI ignores this)
   - Strong: "Archon replaces $85,000-$180,000/yr in staff costs for a typical 5-15 person service business" (AI cites this)
   - Include specific numbers, percentages, price ranges, timeframes, locations

3. **Entity association:** Use the business name naturally and frequently. AI needs clear entity→fact binding.

4. **Content organization optimized for extraction:**
   - Key facts in the FIRST paragraph of each section (AI reads top-heavy)
   - Bullet points for feature/service/benefit lists (AI extracts lists well)
   - Comparison tables with clear headers (AI loves structured data)
   - Pull quotes / callout boxes for the most citable statements

5. **FAQ sections on every service page:** 5-8 questions per page, marked up with FAQPage schema. Questions should match what people actually ask AI assistants.

## Technical Implementation

### JSON-LD Structured Data

Every page gets relevant structured data. Validate against Google Rich Results Test.

**Organization (site-wide, in layout head):**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Business Name",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "contactPoint": { "@type": "ContactPoint", "telephone": "+1-XXX-XXX-XXXX", "contactType": "customer service" },
  "sameAs": ["https://facebook.com/...", "https://instagram.com/...", "https://linkedin.com/..."]
}
```

**LocalBusiness (if physical location):**
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Business Name",
  "address": { "@type": "PostalAddress", "streetAddress": "...", "addressLocality": "...", "addressRegion": "...", "postalCode": "..." },
  "geo": { "@type": "GeoCoordinates", "latitude": "...", "longitude": "..." },
  "openingHoursSpecification": [{ "@type": "OpeningHoursSpecification", "dayOfWeek": ["Monday", "Tuesday"], "opens": "09:00", "closes": "17:00" }],
  "priceRange": "$$",
  "areaServed": { "@type": "GeoCircle", "geoMidpoint": { "@type": "GeoCoordinates", "latitude": "...", "longitude": "..." }, "geoRadius": "30mi" }
}
```

**Product/Service (per service page):**
```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "Service Name",
  "description": "One-sentence factual description",
  "provider": { "@type": "Organization", "name": "Business Name" },
  "areaServed": "City, State",
  "offers": { "@type": "Offer", "priceRange": "$X-$Y" }
}
```

**FAQPage (per FAQ section):**
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "How much does [service] cost?",
      "acceptedAnswer": { "@type": "Answer", "text": "Specific factual answer with numbers." }
    }
  ]
}
```

**HowTo (process/methodology pages):**
```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How [Process] Works",
  "step": [
    { "@type": "HowToStep", "name": "Step 1", "text": "Description" }
  ]
}
```

**AggregateRating (if testimonials exist):**
```json
{
  "@context": "https://schema.org",
  "@type": "AggregateRating",
  "ratingValue": "4.9",
  "reviewCount": "47",
  "bestRating": "5"
}
```

### Semantic HTML

Every page must use:
- Single `<h1>`, logical `<h2>`/`<h3>` nesting (never skip levels)
- Landmark elements: `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`
- `<dl>` definition lists for key term definitions
- `<figure>` + `<figcaption>` for images with context
- `<blockquote>` + `<cite>` for testimonials
- `<address>` for contact information
- `<time datetime="...">` for dates

### Meta Layer

Every page must have:
- `<title>` — factual, under 60 chars, includes business name
- `<meta name="description">` — factual summary (not marketing fluff), under 155 chars
- `<meta property="og:title">`, `og:description`, `og:image`, `og:url`, `og:type`
- `<meta name="twitter:card" content="summary_large_image">`
- `<link rel="canonical" href="...">` — prevent duplicate content
- `hreflang` tags if multi-language

## Authority Signals

Content patterns that build citation authority:

- **About page**: Founder credentials, years of experience, specific qualifications, certifications
- **Service pages**: Specific pricing or price ranges (AI cites prices)
- **Location pages**: Exact address, service radius, local landmarks/context
- **Case studies**: Specific outcomes with numbers ("reduced costs by 40%", "increased revenue by $12K/mo")
- **Comparison content**: "X vs Y" pages with structured tables
- **Regular updates**: Blog/insights answering industry questions (signals freshness to AI)

## GEO Monitoring

**Baseline test (Phase 5, after deploy):**

Using `agent-browser`, query AI search engines and record whether the site appears:

```bash
# Test with Perplexity (has web search)
agent-browser open "https://www.perplexity.ai"
agent-browser snapshot -i
# Type query: "best [service] in [location]"
# Screenshot the response
agent-browser screenshot -o .lighthouse/geo-baseline-perplexity.png
```

**Test queries to run:**
1. "best [service] in [location]"
2. "[service] near [location]"
3. "how much does [service] cost in [location]"
4. "[business name] reviews"
5. "what should I look for in a [provider]"

**Record results** in `.lighthouse/geo-baseline.md`:
```markdown
# GEO Baseline — {date}

| Query | Engine | Cited? | Position | Notes |
|-------|--------|--------|----------|-------|
| best tattoo in Charlotte NC | Perplexity | No | — | Competitor X cited instead |
| best tattoo in Charlotte NC | ChatGPT | No | — | Generic answer |
```

**Periodic monitoring:** Re-run baseline queries monthly. Track changes. Identify content gaps.

**Competitor GEO audit:** Run the same queries substituting competitor names. Who does the AI cite? What content do they have that we don't?

## GEO Content Scoring

Score each page 1-10 for AI-citability during QA:

| Criterion | Weight | Check |
|-----------|--------|-------|
| Specific data points | 2 | Numbers, prices, percentages, timeframes |
| Q&A format content | 2 | FAQ sections, question-format headers |
| Structured data | 2 | JSON-LD present and valid |
| Semantic HTML | 1 | Proper heading hierarchy, landmarks |
| Entity association | 1 | Business name appears in context 3+ times |
| Meta quality | 1 | Factual title/description, OG tags present |
| Freshness signals | 1 | Date published, last updated |

Score 7+ = good. Score 4-6 = needs improvement. Score 1-3 = rewrite.

## GEO + Traditional SEO Combined

GEO layers ON TOP of traditional SEO — never a replacement:
- XML sitemap auto-generated from routes
- `robots.txt` with sitemap reference
- Search Console submission after deploy
- Core Web Vitals (Google uses them; AI models factor in site quality)
- Backlink profile still matters (AI trusts well-linked domains)

## Quality Checklist

- [ ] JSON-LD structured data on every page — validated via Rich Results Test
- [ ] FAQPage schema on every page with an FAQ section
- [ ] Organization schema in site-wide layout
- [ ] LocalBusiness schema if physical location
- [ ] Every page has factual `<title>` + `<meta description>` + OG tags + canonical
- [ ] Semantic HTML: single h1, logical hierarchy, landmarks, figure/figcaption
- [ ] At least 3 citable statements with specific data per key page
- [ ] GEO content score 7+ on every key page
- [ ] XML sitemap generated and valid
- [ ] robots.txt present with sitemap reference
- [ ] GEO baseline test completed and recorded
