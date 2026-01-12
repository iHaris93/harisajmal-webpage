# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project overview

This repository contains a fully static personal/portfolio site for **harisajmal.com**:
- Main landing page: `index.html`
- Privacy policy: `privacy.html`
- SEO/robots config: `robots.txt`, `sitemap.xml`
- Static assets: `assets/profile/` (profile image), `assets/icons/` (brand icons, favicons referenced from HTML)

There is **no build system, package manager configuration, or test framework** defined. The HTML files are production-ready documents that can be served directly by any static web host.

## Architecture & structure

### HTML entrypoints

- **`index.html`**
  - Single-page layout built with **Tailwind CSS via CDN** (`https://cdn.tailwindcss.com`), configured with `darkMode: 'class'`.
  - Sections:
    - Hero with profile image, title, role, and primary CTAs (LinkedIn, Buy Me a Coffee)
    - About section describing experience and responsibilities
    - Skills & expertise presented as Tailwind-styled lists
    - Certifications grid with links and icons
    - Contact CTA section re-using the main CTAs
    - Footer with dynamic copyright year
  - Accessibility and UX:
    - `lang="en"` on `<html>` and a **skip-to-content** link targeting `#about-me`
    - Custom hero gradient background and reduced-motion styles in an inline `<style>` block

- **`privacy.html`**
  - Standalone privacy policy page using Tailwind via CDN and a simple typographic layout.
  - Explains GA4 usage, cookie behavior, user rights (GDPR-style), and contact email.
  - The cookie banner in `index.html` links here via `href="/privacy.html"`, so keep this filename and route stable.

### Styling setup

- Styling is entirely done via **Tailwind utility classes** loaded from the CDN in each HTML document; there is **no local CSS build pipeline**.
- A small inline `<style>` block in `index.html` configures base fonts, transition behavior, and reduced-motion overrides.
- If you introduce new styling, prefer Tailwind utility classes to stay consistent with the current approach.

### JavaScript behavior

All behavior is implemented via inline `<script>` tags in `index.html`:

1. **Dark mode system + toggle**
   - Uses the `dark` class on `<html>` (`document.documentElement`) combined with Tailwind's `dark:` variants.
   - Persists the theme in `localStorage` under the key `theme` (`'dark'` or `'light'`).
   - On initial load, it checks `localStorage` first, and falls back to the OS-level `prefers-color-scheme: dark` media query.
   - The toggle button text is updated between `"🌙 Dark Mode"` and `"☀️ Light Mode"` to reflect the current state.
   - When changing how dark mode works, ensure you:
     - Continue to respect system preferences on first visit, and
     - Preserve or carefully migrate the existing `localStorage` contract if possible.

2. **Dynamic year in footer**
   - Simple DOM update setting `#year` to `new Date().getFullYear()`.
   - If you modify the footer markup, keep this element ID or update the script accordingly.

3. **Analytics click tracking helper**
   - Delegated click listener on `document.body` that looks for the closest `<a>` with `data-gtag-category` and optional `data-gtag-label` attributes.
   - On click, it pushes an event into `window.dataLayer` if present (for Google Tag Manager), otherwise falls back to calling `gtag('event', 'click', ...)` if `gtag` exists.
   - This script is tightly coupled to anchor elements annotated with these data attributes (used on CTAs and certification links). When adding new tracked links, use the same `data-gtag-*` pattern so they are automatically tracked.

4. **Cookie consent banner**
   - Uses **CookieConsent v3** via CSS and JS loaded from the jsDelivr CDN.
   - Initialized on `window.load` with a simple dark theme and a message that links to `/privacy.html`.
   - The banner explains that only basic analytics cookies are used.
   - If you change cookie behavior or analytics stack, update both this initialization and the text in `privacy.html` to stay consistent.

### SEO, metadata & structured data

- `index.html` defines comprehensive metadata:
  - Standard `<meta>` tags for description, keywords, author, and responsive viewport.
  - Canonical URL (`https://harisajmal.com/`).
  - Theme color for both light and dark schemes.
  - Open Graph tags (title, description, URL, type, image) used by LinkedIn and other social platforms.
- Favicons and touch icons are served from `assets/icons/` and wired up in `<head>`.
- A JSON-LD **EducationalOccupationalCredential** block describes the HubSpot certification.
  - If you add or change structured data, ensure the JSON-LD stays valid against Schema.org and that URLs remain correct.

### Robots & sitemap

- **`robots.txt`**:
  - Allows all crawlers except for a few disallowed paths (`/admin/`, `/404.html`).
  - Points to the sitemap at `https://harisajmal.com/sitemap.xml`.
- **`sitemap.xml`**:
  - XML sitemap with two URLs: the homepage and the privacy policy.
  - Each entry declares `lastmod`, `changefreq`, and `priority`.

If you add new top-level pages, remember to:
- Update `sitemap.xml` with additional `<url>` entries.
- Consider whether `robots.txt` needs changes for new paths.

### Third-party integrations

- **Tailwind CSS CDN**: Primary styling mechanism; no local PostCSS or Tailwind config files.
- **Google Tag Manager (GTM)**:
  - Standard GTM snippet in `<head>` and `<noscript>` iframe in `<body>`.
  - Uses a specific container ID; avoid changing it unintentionally.
- **Google Analytics (GA4)**:
  - Not directly included via a GA script in this repo; events are sent via GTM using `dataLayer` when available.
  - The privacy policy assumes GA4-style anonymized analytics; if you switch analytics providers, update the policy.
- **CookieConsent**: Provides the cookie banner and basic consent handling, especially for EU/UK visitors.

## Development & preview commands

There are no language runtimes or tooling required beyond a web browser. The project is static HTML/asset files.

From the project root (`/Users/harisajmal/harisajmal-webpage`):

### Run a local development server

Use any simple static file server. On macOS with Python installed, a common option is:

```bash
cd /Users/harisajmal/harisajmal-webpage
python3 -m http.server 8000
```

Then open `http://localhost:8000/` in a browser to preview `index.html`. `privacy.html`, `robots.txt`, and `sitemap.xml` will also be served at their corresponding paths.

### Direct file open

You can also open the HTML files directly in a browser without a server (for quick checks):

```bash
open index.html
open privacy.html
```

Note: Some analytics or cookie/consent behavior may not work exactly the same without an HTTP server (e.g., origin-dependent logic), so prefer using a local server when debugging integrations.

### Build, lint, and tests

- **Build:** No build step is defined; deployment is expected to serve the HTML and assets as-is.
- **Lint:** There are no HTML/CSS/JS linters configured in this repo (no `package.json`, Taskfile, or config files). If you introduce tooling (e.g., HTMLHint, ESLint, or a formatting step), document the commands here.
- **Tests:** No automated tests are present. If you add tests (e.g., Playwright, Cypress, or Lighthouse checks), add the relevant commands and any single-test invocation patterns to this section.

## Things to keep in mind when editing

- Maintain the existing **SEO and social metadata** (titles, descriptions, canonical link, OG image) when refactoring `<head>`; changes can affect search and sharing.
- Be careful when modifying the **GTM snippet** or the **dataLayer click tracking** helper—breaking these can silently disable analytics.
- If you change cookies or analytics behavior, update **both** `privacy.html` and the cookie consent initialization so the implementation and policy remain aligned.
- Preserve accessibility affordances: the skip link, `lang` attributes, reasonable color contrast for new elements, and reduced-motion safeguards.
- There is no bundler; all JS is inline. Keep scripts small, self-contained, and compatible with modern evergreen browsers.
