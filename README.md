# BL⚡TZ · Portfolio

A single-file, zero-dependency personal portfolio for **Blitz (A. Wahab)**, software developer and founder of BlitzCore, Blitz Tech and Neon Ways. Python first.

The site pulls projects, languages and push dates from the GitHub API in the visitor's browser, so new repos appear without editing or redeploying anything.

- **Stack:** one `index.html`, vanilla JavaScript, no framework, no build step, no bundler
- **Data:** GitHub REST API (public, unauthenticated)
- **Tracking:** none. No cookies, no analytics
- **Hosting:** any static host

---

## Contents

1. [Features](#features)
2. [Folder contents](#folder-contents)
3. [Quick start](#quick-start)
4. [Before you publish: set your domain](#before-you-publish-set-your-domain)
5. [Configuration](#configuration)
6. [Customising content](#customising-content)
7. [Design system](#design-system)
8. [How it works](#how-it-works)
9. [SEO and social sharing](#seo-and-social-sharing)
10. [Logos and icons](#logos-and-icons)
11. [Deployment](#deployment)
12. [Accessibility and performance](#accessibility-and-performance)
13. [Troubleshooting](#troubleshooting)
14. [Known limitations](#known-limitations)
15. [License](#license)

---

## Features

**Pages** (hash-routed, no server config needed)

| Route | What it shows |
|---|---|
| `#/` | Hero, About, Stack, Selected work, Language load, GitHub card, Ventures, Contact |
| `#/projects` | Full repo index with language filter chips, text search and sorting |
| `#/projects/<repo>` | Detail page: overview, spec sheet, language split, topics, prev/next |
| `#/about` | Longer about page with principles |
| `#/contact` | Telegram, X, Instagram, TikTok, GitHub |

**Data-driven**
- Repo list, stars, top languages, push dates and the GitHub profile card come from the API
- Forks are excluded by default; archived repos are tagged
- "Selected work" ranks repos by stars, live demo link, description, topics and recent activity
- Clicking a language row on Home opens the project index pre-filtered to that language
- Detail pages lazily load the per-repo language breakdown and approximate commit count

**UX**
- Sticky filter bar on mobile with horizontally scrolling chips, full-width search, 16px inputs (no iOS zoom)
- Press `/` on the project index to jump to search
- Back-to-top button, scroll progress "volt gauge" in the header, animated full-screen mobile menu
- Error state with a Retry button if GitHub can't be reached; loading shimmer while data loads
- Responses cached in `sessionStorage` for 30 minutes to stay under GitHub's rate limit

**Motion**
- Letter-by-letter hero reveal with flickering bolt, canvas lightning arcs in the hero (follows the pointer)
- Rotating role line with a blinking cursor
- Scroll-triggered reveals with stagger, count-up stats, marquee strip
- Pointer-following spotlight on project rows, hover wobble on icons, shine sweep on primary buttons
- All motion is disabled under `prefers-reduced-motion`

---

## Folder contents

```
site/
├── index.html              The whole website (HTML + CSS + JS)
├── README.md               This file
├── bolt.svg                The ⚡ logo, standalone (yellow, transparent)
├── blitz-logo.svg          The BL⚡TZ wordmark (light letters, transparent)
├── favicon.svg             Modern browsers' tab icon
├── favicon.ico             Legacy fallback (16, 32, 48 px)
├── favicon-32.png          32×32 PNG
├── apple-touch-icon.png    180×180, iOS home screen
├── icon-192.png            Android / PWA
├── icon-512.png            Android / PWA (maskable)
├── og-image.png            1200×630 link preview thumbnail
├── site.webmanifest        Web app manifest
├── sitemap.xml             Sitemap (home page)
└── robots.txt              Crawler rules + sitemap pointer
```

Keep every file in the **same folder** as `index.html`. The page links to them with relative paths.

---

## Quick start

**Preview locally**

Open `index.html` in a browser. It works from `file://`.

For a more realistic test (service workers aside, mostly relevant for the preview image and icons), serve the folder:

```bash
cd site
python3 -m http.server 8080
# open http://localhost:8080
```

You need an internet connection. The page calls `api.github.com`, Google Fonts and the Devicon CDN (stack icons).

---

## Before you publish: set your domain

Open Graph, the canonical link, structured data, the sitemap and `robots.txt` all need an **absolute URL**. They currently contain the placeholder:

```
https://YOUR-DOMAIN.com/
```

Replace it everywhere in one go (include the trailing slash):

```bash
# macOS / Linux
sed -i 's#https://YOUR-DOMAIN.com/#https://yourname.dev/#g' index.html sitemap.xml robots.txt

# macOS (BSD sed)
sed -i '' 's#https://YOUR-DOMAIN.com/#https://yourname.dev/#g' index.html sitemap.xml robots.txt
```

On Windows, use your editor's *Find in Files* on those three files.

Files and places affected:

| Where | What |
|---|---|
| `index.html` `<link rel="canonical">` | Canonical URL |
| `index.html` `og:url`, `og:image`, `twitter:image` | Share card URLs |
| `index.html` JSON-LD block | `url`, `image`, `@id` |
| `sitemap.xml` | `<loc>` |
| `robots.txt` | `Sitemap:` line |

If you deploy under a sub-path (for example `https://user.github.io/portfolio/`), use that full path as the domain.

---

## Configuration

All site behaviour you're likely to touch sits at the top of the main `<script>` in `index.html`.

```js
const GH_USER = 'blitzlabx';       // GitHub username the whole site reads from
const CONFIG = {
  includeForks: false,             // true = show forked repos too
  hide:  [],                       // repo names to keep off the site
  pin:   [],                       // repo names forced to the top of Selected work
  notes: {}                        // per-repo hand-written content (see below)
};
const SOCIALS = [ /* label, handle, note, url */ ];
```

### Change the handle

Set `GH_USER` to your GitHub username. Then update `SOCIALS`, and search the file for `blitzlabx`. A few links (hero buttons, contact strip, structured data, the About page link) are built from it or typed directly.

### Hide or pin repos

```js
hide: ['dotfiles', 'test-repo'],
pin:  ['devpilot', 'ghostlink'],
```

Pinned repos jump to the top of **Selected work**. The home page shows the top 4.

### Hand-written project pages

Repo descriptions are often terse. Add a human write-up per repo:

```js
notes: {
  'devpilot': {
    tagline:   'Developer toolbox inside Telegram.',
    overview:  'A longer paragraph shown on the project detail page.',
    highlights:['Feature one', 'Feature two', 'Feature three'],
    image:     'https://example.com/screenshot.png'
  }
}
```

- `tagline` replaces the repo description in lists
- `overview` shows on the detail page
- `highlights` renders as the "What it does" list
- `image` shows as a screenshot at the top of the detail page

The key must match the repo name exactly.

---

## Customising content

### Role line in the hero

The rotating words (`Software developer`, `Founder`, `Python first`, `I build · I ship`) live in the enhancement block at the bottom of `index.html`, inside `<script id="enh-js">`, in the array named `w`. The static first word is in the `pgHome()` function (`id="rot"`).

### Tech stack

Edit the `STACK` array in the main script:

```js
const STACK = [
  ['Python', 'python'],            // first entry = the big "Main language" card
  ['JavaScript', 'javascript'],
  ['Next.js', 'nextjs', 'inv'],    // 'inv' inverts a dark icon so it shows on a dark background
  ...
];
```

The second value is the [Devicon](https://devicon.dev) icon slug (`https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/<slug>/<slug>-original.svg`). If an icon fails to load, it is removed and the text label remains.

### About copy and ventures

Search `index.html` for the text you want to change. The home About block and the About page are in `pgHome()` and `pgAbout()`. The Ventures cards (BlitzCore, Blitz Tech, Neon Ways) are in the `vgrid` markup in `pgHome()`.

### Contact links

Edit `SOCIALS` for the Contact page and mobile menu. The five icon buttons on the home Contact strip are written out in `pgHome()`.

---

## Design system

Colours, fonts and easing are CSS variables at the top of the `<style>` block:

```css
:root{
  --bg:#0B0B0C;  --bg2:#111113;  --line:#232327;  --line2:#333338;
  --txt:#F4F3EF; --mut:#A8A8A0;  --dim:#6F6F69;
  --volt:#F5E003;                /* accent */
  --fd:'Space Grotesk',sans-serif;
  --fm:'JetBrains Mono',monospace;
}
```

Change `--volt` to re-theme the whole site. The bolt colour in the SVG files and the manifest `theme_color` are separate; update those too if you change the accent.

The code is organised as: tokens → base → loader → header → mobile menu → hero → sections → pages → footer → responsive → reduced motion. Mobile-specific overrides and the newer components (stack, ventures, role row, back-to-top, extra animations) are in the second `<style id="enh-css">` block at the end of the file, which loads after the first and wins on conflicts.

**Breakpoints:** 880px (header switches to burger menu), 860px (two-column layouts collapse), 720px (sticky mobile filter bar), 560px (small phones).

---

## How it works

1. **Boot.** A loader shows while the page requests `/users/<GH_USER>` and `/users/<GH_USER>/repos?per_page=100&sort=updated`.
2. **Cache.** The result is stored in `sessionStorage` under `blitz.gh.v1` for 30 minutes. Reloads and navigation within that time make no new API calls.
3. **Timeouts.** Requests abort after 9 seconds. The loader never blocks longer than ~4.5 seconds, and after ~5.2 seconds without data the page shows an error state with a Retry button.
4. **Derive.** Forks and hidden repos are filtered out, repos are scored for "Selected work", languages are counted per repo, and stars are summed.
5. **Router.** A tiny hash router renders each page into `#view` with a short fade transition. No server rewrites are required, which is why it works on any static host.
6. **Detail pages.** Opening a project fetches `/repos/<owner>/<repo>/languages` and the commits endpoint (to estimate commit count from the pagination header).
7. **Reveals.** An `IntersectionObserver` adds an `in` class to `[data-reveal]` elements as they scroll into view.

**Rate limits.** GitHub allows 60 unauthenticated requests per hour per IP. Typical visits use 2 requests (plus 2 per project detail page opened), and the cache keeps repeat views free. If a visitor hits the limit, they see a friendly message and a Retry button.

---

## SEO and social sharing

What's included in `<head>`:

- `<title>` and meta description
- `robots` meta, canonical link, author and keywords
- Open Graph tags (`og:title`, `og:description`, `og:image` with size and alt text, `og:url`, `og:type`)
- Twitter/X `summary_large_image` card
- JSON-LD structured data: a `WebSite` and a `Person` (name "Blitz", alternate name "A. Wahab", job title, the three organisations, `sameAs` social profiles)
- Favicons (SVG, ICO, PNG), Apple touch icon and web manifest
- `sitemap.xml` and `robots.txt`
- An `<h1>` with an accessible name for screen readers and crawlers, plus `<noscript>` fallback text

**After deploying**

1. Submit `https://yourdomain/sitemap.xml` in [Google Search Console](https://search.google.com/search-console) and Bing Webmaster Tools, and request indexing for the home page.
2. Check the preview card with the platform debuggers:
   - Facebook Sharing Debugger
   - LinkedIn Post Inspector
   - X Card Validator (or just paste the link into a draft post)
3. Social platforms cache previews. If you change `og-image.png`, rename the file (e.g. `og-image-v2.png`) and update the three meta tags.

**Expectations.** Because routes use `#/…`, search engines treat the whole site as one page and index the home page only. If you want individual project pages indexed, you would need a server-rendered or pre-rendered version. The site is built for a personal portfolio, where one strong home page is usually what matters.

---

## Logos and icons

| File | Use |
|---|---|
| `bolt.svg` | The ⚡ mark on its own. Yellow `#F5E003`, transparent background, 24×24 grid |
| `blitz-logo.svg` | BL⚡TZ wordmark. Light letters (`#F4F3EF`) + yellow bolt, transparent. Made for dark backgrounds |
| `favicon.svg` | Bolt on a rounded dark tile |
| `og-image.png` | 1200×630 share preview |

The wordmark letters are drawn as vector paths, not text, so it renders identically everywhere with no font needed. For a light-background version, change `stroke="#F4F3EF"` in `blitz-logo.svg` to a dark colour.

---

## Deployment

The site is static, so any host works. Upload the **contents of the `site/` folder** (with `index.html` at the root).

**GitHub Pages**
1. Push the files to a repo (for a user site, name it `<username>.github.io`)
2. Settings → Pages → deploy from the `main` branch, root folder
3. Optional custom domain: add it in the Pages settings and create a `CNAME` file containing the domain

**Netlify / Cloudflare Pages / Vercel**
- Drag and drop the folder, or connect the repo
- Build command: none. Output/publish directory: the folder containing `index.html`

**Any web server (Nginx, Apache, shared hosting)**
- Copy the files to the web root. No rewrite rules needed

Recommended: serve over HTTPS and enable gzip/brotli. The page is about 100 KB uncompressed.

---

## Accessibility and performance

- Skip-to-content link, visible focus rings, semantic landmarks and ARIA labels on icon-only controls
- Closed mobile menu is hidden from keyboard and screen readers; `Esc` closes it
- Respects `prefers-reduced-motion` and uses `color-scheme: dark`
- Tap targets of at least 40–48px on mobile
- No layout libraries, no images in the critical path, lazy-loaded avatars, canvas animation pauses when the hero is off-screen or the tab is hidden
- Fonts load from Google Fonts with fallbacks, so text renders immediately

---

## Troubleshooting

| Symptom | Likely cause and fix |
|---|---|
| Stuck on the loader or an error box | GitHub unreachable or rate-limited. Wait a few minutes and press Retry |
| No projects listed | The account has no public, non-forked repos, or they're all in `hide`. Check `GH_USER` spelling |
| Stack icons missing | The Devicon CDN is blocked or offline. Labels still show. Self-host the SVGs and change the `dv()` function if needed |
| Link preview shows no image | Domain placeholder not replaced, or the platform cached an older preview. Use the debuggers listed above |
| Fonts look different | Google Fonts blocked. The fallback fonts are used |
| Old data after pushing a repo | Session cache (30 min). Open a new tab or clear site data |
| Page is blank with JS disabled | Expected. A `<noscript>` message is shown; the site needs JavaScript |

---

## Known limitations

- Only the first 100 repos (sorted by recent update) are read
- Unauthenticated GitHub API: 60 requests per hour per IP
- Hash routes are not indexed individually by search engines (see [SEO](#seo-and-social-sharing))
- Commit counts on detail pages are approximate (derived from pagination)
- Stack icons depend on a third-party CDN
- Requires a modern browser (CSS `clip`, `svh`, `IntersectionObserver`, `backdrop-filter`). Current Chrome, Edge, Safari and Firefox are fine
- The `og-image.png` is static; it won't update itself

---

## License

All rights reserved © Blitz (A. Wahab), unless you choose otherwise. Add a `LICENSE` file if you want to open-source the site.

Third-party: [Space Grotesk](https://fonts.google.com/specimen/Space+Grotesk) and [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) (SIL Open Font License), [Devicon](https://devicon.dev) (MIT), data from the [GitHub REST API](https://docs.github.com/en/rest).

<p align="center">I build. I experiment. I ship. ⚡</p>
