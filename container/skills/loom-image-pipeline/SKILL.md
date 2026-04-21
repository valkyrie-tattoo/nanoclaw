---
name: loom-image-pipeline
description: Process, optimize, and generate images — Sharp resizing, WebP/AVIF conversion, srcset, favicons, OG images, SVGO, generation prompts
---

# Image Pipeline

Every image that enters a project — uploaded by Jesse, downloaded during migration, or generated — goes through this pipeline before serving.

## When to Use

- Jesse sends photos via Telegram for a project
- Processing images during site migration (downloaded assets)
- Generating images for a project (hero graphics, backgrounds, icons)
- Building favicon sets from a logo
- Creating OG images for pages
- Optimizing SVGs

## Setup (Per-Project, First Time)

```bash
cd /workspace/group/projects/{slug}
npm init -y 2>/dev/null
npm install sharp @vercel/og svgo --save
```

Check before installing: `test -d node_modules/sharp && echo "already installed"`. Sharp uses prebuilt binaries for linux-x64 — no build tools needed.

## Photo Management

**Receiving photos from Jesse:**
1. Save to `brand/photos/YYYY-MM-DD-{description}.{ext}`
2. Log to `brand/photos/index.md` with description, date, intended use
3. If Jesse specifies a project: save to that project's `brand/photos/`
4. If general (no project): save to `/workspace/group/photo-library/`

**Before requesting new images for a page:**
1. Check project `brand/photos/` for relevant assets
2. Check `/workspace/group/photo-library/` for general assets
3. Only request new images if nothing suitable exists

## Image Processing

**Core optimization (every image):**

```bash
# Resize + convert to WebP with quality optimization
node -e "
const sharp = require('sharp');
sharp('input.jpg')
  .resize(1280, null, { withoutEnlargement: true })
  .webp({ quality: 80, effort: 6 })
  .toFile('output.webp');
"
```

**Responsive srcset generation:**

```bash
node -e "
const sharp = require('sharp');
const widths = [640, 768, 1024, 1280, 1536];
const input = 'brand/photos/hero.jpg';
for (const w of widths) {
  sharp(input)
    .resize(w, null, { withoutEnlargement: true })
    .webp({ quality: 80 })
    .toFile(\`public/images/hero-\${w}w.webp\`);
}
"
```

**LQIP (Low Quality Image Placeholder):**

```bash
node -e "
const sharp = require('sharp');
sharp('input.jpg')
  .resize(20)
  .blur()
  .toBuffer()
  .then(buf => console.log(\`data:image/webp;base64,\${buf.toString('base64')}\`));
"
```

**EXIF stripping (privacy):**

```bash
node -e "
const sharp = require('sharp');
sharp('input.jpg')
  .rotate() // auto-rotate based on EXIF, then strip
  .withMetadata({ orientation: undefined })
  .toFile('output.jpg');
"
```

**Processing rules:**
- Target: no image over 200KB served without explicit justification
- Always generate WebP. Generate AVIF for hero/above-fold images (slower encode, better compression)
- Lazy loading attribute on all below-fold images
- All images must have alt text written before placement

## Favicon Generation

From a logo source file, generate the full favicon set:

```bash
node -e "
const sharp = require('sharp');
const sizes = [16, 32, 48, 180, 192, 512];
const input = 'brand/logos/logo.png';
for (const s of sizes) {
  sharp(input)
    .resize(s, s, { fit: 'contain', background: { r: 0, g: 0, b: 0, alpha: 0 } })
    .png()
    .toFile(\`brand/icons/favicon-\${s}x\${s}.png\`);
}
// Also generate ICO-compatible 32x32
sharp(input).resize(32, 32).toFile('public/favicon.ico');
"
```

**Generate web manifest:**
```json
{
  "name": "Project Name",
  "short_name": "Project",
  "icons": [
    { "src": "/favicon-192x192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/favicon-512x512.png", "sizes": "512x512", "type": "image/png" }
  ],
  "theme_color": "#primary-color",
  "background_color": "#bg-color",
  "display": "standalone"
}
```

**Generate browserconfig.xml:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<browserconfig>
  <msapplication>
    <tile>
      <square150x150logo src="/favicon-192x192.png"/>
      <TileColor>#primary-color</TileColor>
    </tile>
  </msapplication>
</browserconfig>
```

Place favicons in `public/` and `brand/icons/`.

## OG Image Generation

Per-page OG images using @vercel/og or Satori:

```bash
node -e "
const { ImageResponse } = require('@vercel/og');
const fs = require('fs');

const html = {
  type: 'div',
  props: {
    style: {
      width: '1200px', height: '630px',
      display: 'flex', flexDirection: 'column',
      justifyContent: 'center', alignItems: 'center',
      backgroundColor: '#bg-color', color: '#text-color',
      fontFamily: 'Inter', fontSize: '48px', fontWeight: 700,
      padding: '60px',
    },
    children: 'Page Title — Brand Name'
  }
};

new ImageResponse(html, { width: 1200, height: 630 })
  .arrayBuffer()
  .then(buf => fs.writeFileSync('public/og/page-name.png', Buffer.from(buf)));
"
```

Generate one OG image per page. Use brand colors and fonts. Save to `public/og/{page-slug}.png`.

## SVG Optimization

```bash
npx svgo input.svg -o output.svg --multipass --pretty
```

For batch processing:
```bash
npx svgo -f brand/logos/ -o public/logos/ --multipass
```

## Image Generation Prompts

When a project needs generated images (hero graphics, backgrounds, illustrations):

**For each needed image, produce:**
1. **Prompt** — detailed generation prompt with style, composition, lighting, color palette constraints matching the project brand
2. **Aspect ratio** — 16:9 for hero, 1:1 for thumbnails, 3:2 for cards
3. **Resolution** — minimum 2x display size (e.g. 2560x1440 for a 1280px hero)
4. **Alt text** — pre-written before generation
5. **Brand constraints** — specific hex colors to emphasize, mood keywords

**If `$REPLICATE_TOKEN` is set (Flux/SDXL):**
```bash
curl -s -X POST https://api.replicate.com/v1/predictions \
  -H "Authorization: Bearer $REPLICATE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"version": "model-version-id", "input": {"prompt": "...", "width": 1280, "height": 720}}'
```

**If `$OPENAI_API_KEY` is set (DALL-E):**
```bash
curl -s -X POST https://api.openai.com/v1/images/generations \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "dall-e-3", "prompt": "...", "size": "1792x1024", "quality": "hd"}'
```

After generation: download, run through full processing pipeline (resize, WebP, srcset, LQIP), place in project.

**If no image API configured:** Output the prompt with all specs. Jesse generates externally and sends back.

## Quality Checklist

- [ ] Every served image is under 200KB (or has documented justification)
- [ ] WebP versions generated for all raster images
- [ ] Responsive srcset generated for images wider than 640px
- [ ] LQIP placeholder generated for above-fold hero images
- [ ] EXIF data stripped from all photos
- [ ] All images have alt text written
- [ ] Below-fold images have `loading="lazy"` attribute
- [ ] Favicon set complete: 16, 32, 48, 180, 192, 512 + .ico + manifest + browserconfig
- [ ] OG images generated for every page (1200x630)
- [ ] SVGs optimized via SVGO
- [ ] No unprocessed raw images in `public/`
