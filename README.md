# @valesordev/ui

Shared UI primitives for Valesor web properties. Astro components and base styles used across all sites in the `valesordev` GitHub org.

Published to GitHub Packages: `https://npm.pkg.github.com`

## Installation

Add to your site's `.npmrc`:

```
@valesordev:registry=https://npm.pkg.github.com
```

Install the package:

```bash
npm install @valesordev/ui
```

**Local dev auth:** GitHub Packages requires authentication even for public packages. Add a PAT with `read:packages` to your global `~/.npmrc`:

```
//npm.pkg.github.com/:_authToken=YOUR_PAT
```

In CI, pass `secrets.GITHUB_TOKEN` as `NODE_AUTH_TOKEN` — see the reusable deploy workflow below.

## Peer dependencies

```json
{
  "astro": ">=5.0.0",
  "tailwindcss": ">=4.0.0"
}
```

---

## Components

### `BaseLayout.astro`

HTML shell. Every site wraps its pages in this.

```astro
import BaseLayout from '@valesordev/ui/BaseLayout.astro';
```

#### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `title` | `string` | required | `<title>` and OG title |
| `description` | `string` | — | `<meta name="description">` (omitted if not provided) |
| `favicon` | `string` | `'/favicon.svg'` | Path to favicon |

#### Slots

| Slot | Description |
|---|---|
| `head` | Additional `<head>` content (styles, meta tags, scripts) |
| *(default)* | Page body content |

#### Example

```astro
---
import BaseLayout from '@valesordev/ui/BaseLayout.astro';
---
<BaseLayout title="My Site" description="A Valesor web property" favicon="/logo.svg">
  <slot name="head">
    <!-- extra head content here -->
  </slot>
  <main>Page content</main>
</BaseLayout>
```

---

### `SiteHeader.astro`

Fixed top header with logo slot and nav slot. Applies `transition-all duration-300` so sites can animate background/opacity on scroll via JS.

```astro
import SiteHeader from '@valesordev/ui/SiteHeader.astro';
```

#### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `id` | `string` | — | Element ID (useful for scroll behavior targeting) |
| `class` | `string` | `''` | Additional classes (appended; use for bg color, initial state) |

#### Slots

| Slot | Description |
|---|---|
| `logo` | Left side — brand mark, wordmark, or logo image |
| `nav` | Right side — navigation links, wrapped in `<nav>` |

#### Example

```astro
<SiteHeader id="site-header" class="bg-transparent">
  <div slot="logo">
    <a href="/">Valesor</a>
  </div>
  <ul slot="nav" class="flex space-x-6">
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</SiteHeader>
```

#### Scroll behavior pattern

Target the header by `id` in a client script to change appearance on scroll:

```html
<script>
  const header = document.getElementById('site-header');
  window.addEventListener('scroll', () => {
    header.classList.toggle('bg-black/80', window.scrollY > 50);
    header.classList.toggle('backdrop-blur-md', window.scrollY > 50);
  });
</script>
```

---

### `SiteFooter.astro`

Full-width footer wrapper with centered container.

```astro
import SiteFooter from '@valesordev/ui/SiteFooter.astro';
```

#### Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `class` | `string` | `''` | Additional classes (bg color, text color, etc.) |

#### Slots

| Slot | Description |
|---|---|
| *(default)* | Footer content — copyright, links, columns |

#### Example

```astro
<SiteFooter class="bg-neutral-900 text-white">
  <p>&copy; {new Date().getFullYear()} Valesor Development</p>
</SiteFooter>
```

---

## Styles

### `styles/base.css`

Base stylesheet. Import in your site's global CSS or directly in a layout.

```css
/* in your global.css */
@import '@valesordev/ui/styles/base.css';
```

Includes:
- `@import "tailwindcss"` — Tailwind 4 base
- CSS custom properties for theming:

| Variable | Default | Purpose |
|---|---|---|
| `--color-bg` | `#0a0a0a` | Page background |
| `--color-text` | `#ededed` | Body text |

Override in your site's CSS after the import:

```css
@import '@valesordev/ui/styles/base.css';

:root {
  --color-bg: #1a1a1a;
  --color-text: #f5f5f5;
}
```

---

## CI/CD

### Reusable deploy workflow

Sites call the shared workflow from this repo. Add `.github/workflows/deploy.yml` to your site repo:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    uses: valesordev/ui/.github/workflows/astro-pages.yml@main
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The reusable workflow handles: checkout → Node setup → `npm ci` (with GitHub Packages auth) → `npm run build` → upload artifact → deploy to Pages.

### Publishing

A new version is published to GitHub Packages when a GitHub Release is created, or manually via `workflow_dispatch` on `publish.yml`.

```bash
# bump version in package.json, commit, push, then create a release on GitHub
```
