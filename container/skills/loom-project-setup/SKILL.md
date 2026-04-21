---
name: loom-project-setup
description: Initialize new web projects — framework selection, directory scaffolding, Tailwind config, Git init, starter kit setup
---

# Project Setup

Initialize new project workspaces with the right framework, directory structure, and tooling.

## When to Use

- Starting any new project
- Jesse assigns a new site to build
- After brand system is approved (Phase 0 → Phase 1 transition)
- Setting up a project workspace for migration

## Framework Decision Tree

| Signal | Framework | Why |
|--------|-----------|-----|
| Content-heavy, blog, portfolio, marketing site | **Astro** (DEFAULT) | Static by default, best Lighthouse scores, islands for interactivity |
| Dynamic: auth, dashboard, real-time, heavy API | **Next.js 14+** (App Router) | React ecosystem, server components, API routes |
| Single landing page, event page, coming-soon | **Static HTML** | Zero build overhead, maximum simplicity |

**Default is Astro** unless the project clearly needs Next.js features. GEO advantage: static HTML is most easily parsed by AI crawlers.

Always use **Tailwind CSS 4**. Always use **TypeScript**.

## Scaffolding — Astro (Default)

```bash
PROJECT=/workspace/group/projects/{slug}
mkdir -p $PROJECT/{brand/{logos,photos,generated,icons},design/{style-guide,mockups,competitors},public,.lighthouse}

cd $PROJECT
npm create astro@latest src -- --template minimal --typescript strict --install --no-git 2>&1

cd src
npx astro add tailwind --yes 2>&1

# Install common dependencies
npm install @astrojs/sitemap sharp --save 2>&1
```

**Configure Tailwind with brand tokens:**

Create `src/tailwind.config.mjs`:
```js
import tokens from '../brand/tailwind.tokens.js';
export default {
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}'],
  theme: { extend: { ...tokens.theme.extend } },
  plugins: [],
}
```

**Configure Astro with sitemap:**

Add to `src/astro.config.mjs`:
```js
import sitemap from '@astrojs/sitemap';
export default defineConfig({
  site: 'https://example.com', // update with real URL
  integrations: [sitemap()],
});
```

**Create base layout** with:
- JSON-LD structured data slot
- Meta tags (title, description, OG, canonical)
- Favicon links
- Analytics script (Plausible/Fathom)
- Skip-to-content link
- Font loading with `font-display: swap`

## Scaffolding — Next.js

```bash
PROJECT=/workspace/group/projects/{slug}
mkdir -p $PROJECT/{brand/{logos,photos,generated,icons},design/{style-guide,mockups,competitors},public,.lighthouse}

cd $PROJECT
npx create-next-app@latest src --typescript --tailwind --eslint --app --src-dir --no-import-alias 2>&1

cd src
npm install sharp next-sitemap --save 2>&1
```

**Configure next-sitemap:**

Create `src/next-sitemap.config.js`:
```js
module.exports = {
  siteUrl: 'https://example.com',
  generateRobotsTxt: true,
  robotsTxtOptions: { policies: [{ userAgent: '*', allow: '/' }] },
}
```

## Scaffolding — Static HTML

```bash
PROJECT=/workspace/group/projects/{slug}
mkdir -p $PROJECT/{brand/{logos,photos,generated,icons},design,src/css,src/js,public,.lighthouse}

cd $PROJECT/src
cat > index.html << 'HTMLEOF'
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Project Name</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
  <a href="#main-content" class="sr-only focus:not-sr-only">Skip to content</a>
  <main id="main-content"></main>
</body>
</html>
HTMLEOF
```

## Git Initialization

```bash
cd /workspace/group/projects/{slug}/src

git init
git checkout -b main

# Create .gitignore
cat > .gitignore << 'EOF'
node_modules/
dist/
.vercel/
.env
.env.local
.DS_Store
*.log
EOF

git add -A
git commit -m "Initial project setup — {framework}"
```

**Branch strategy:**
- `main` — production (deployed)
- `staging` — preview URL (Jesse reviews here)
- Feature branches for specific pages/components

**If GitHub repo needed:**
```bash
gh repo create {org}/{slug} --private --source=. --remote=origin --push
```

## Project Index Update

After creating a new project, update `/workspace/group/projects/index.md`:

```markdown
| Slug | Name | Framework | Phase | Status | Deploy URL |
|------|------|-----------|-------|--------|------------|
| {slug} | {name} | Astro | 0 - Brand | In progress | — |
```

## Project README

Create `README.md` in project root:

```markdown
# {Project Name}

## Brief
{one-paragraph project description}

## Brand
{adjectives / source URL / reference URLs}

## Framework
{Astro / Next.js / Static} — {rationale}

## Deploy
- Preview: {url when available}
- Production: {url when available}

## Status
Phase 0 — Brand (in progress)
```

## Quality Checklist

- [ ] Framework selected with documented rationale
- [ ] Directory structure created (brand/, design/, src/, public/, .lighthouse/)
- [ ] Framework scaffolded and builds successfully (`npm run build`)
- [ ] Tailwind configured with brand tokens (or ready to receive them)
- [ ] Sitemap integration installed
- [ ] Git initialized with .gitignore
- [ ] Project added to `projects/index.md`
- [ ] README.md created with brief, framework choice, and status
- [ ] Base layout includes: meta tags, OG slots, skip-to-content link, font loading
