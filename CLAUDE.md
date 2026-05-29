# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **Shopify Online Store 2.0 theme** based on **Dawn v15.4.1**, customized into a branded
single-product landing site for **D'Cal** — a zero-electricity hard-water softener sold in the
Indian market. The Dawn base is untouched; all custom work lives in a **"premium" layer** layered
on top. Product copy, pricing, and form fields are India-specific (PIN codes, Indian states, ₹).

There is **no local build tooling** — no `package.json`, bundler, or compile step. Liquid/CSS/JS
are served as-is. Editing a file changes the theme directly.

## Commands

Work is done with the **Shopify CLI** (the theme has no repo-specific scripts):

```bash
shopify theme dev      # local dev server with hot reload against a dev store
shopify theme push     # upload local files to a store theme
shopify theme pull      # download theme files from a store (also pulls JSON edited in the admin)
shopify theme check    # Theme Check linter (Liquid/JSON/schema validation)
```

There is no test suite. "Testing" a change means previewing it via `shopify theme dev` or the
theme editor.

## ⚠️ GOLDEN RULE — never overwrite live customizer state (NEVER VIOLATE)

The merchant edits **product selections, settings, and content in the Shopify customizer**, which live
in these files on the live theme. Pushing local copies over them has **wiped the homepage Premium
Products selections 4+ times**. These files are the danger zone:

- `templates/*.json`
- `sections/*-group.json` (header-group, footer-group)
- `config/settings_data.json`

**They are listed in `.shopifyignore`, which the Shopify CLI honours on every push/pull — even with
`--only` (verified empirically: `--only` does NOT override `.shopifyignore`).** So a normal `push`
(or even a careless bare `shopify theme push`) physically cannot touch them.

### Workflow

- **Normal work** — `sections/*.liquid`, `snippets/`, `assets/`, `layout/theme.liquid`:
  ```
  push
  ```
  (fish alias: `shopify theme push --theme=140491882599 --allow-live` + `--ignore` for all settings.)

- **Work that MUST change a settings file** (template/section-group/settings_data):
  ```
  sync-from-live      # 1. pull current live customizer state into local
  …edit locally…      # 2. make the change
  deploy-settings     # 3. push ONLY templates/index.json + sections/footer-group.json
  ```
  `sync-from-live` and `deploy-settings` are fish functions that **temporarily move `.shopifyignore`
  aside** (then restore it), because the CLI otherwise refuses to touch ignored files. They're the
  ONLY sanctioned way to read/write settings via CLI.

### Checklist before pushing (future Claude sessions MUST run this mentally)

> **"Am I about to push `templates/*.json`, `sections/*-group.json`, or `config/settings_data.json`?
> If YES, STOP.** Have the user run `sync-from-live` first to back up live customizer state, make the
> change, then deploy with `deploy-settings`. Never push these files with a plain/full push."

If you ever edit `index.json` or a `*-group.json` locally, assume the live version is **newer** (the
merchant may have changed it in the customizer) — `sync-from-live` before touching it.

## Architecture

### Two layers: Dawn base + premium overlay

- **Dawn base** — the standard sections/snippets/templates (`main-product`, `cart-drawer`,
  `card-product`, `base.css`, `global.js`, etc.). Generally leave these alone.
- **Premium overlay** — everything prefixed `premium-`, plus two global assets:
  - `assets/premium-custom.css`
  - `assets/premium-custom.js`

Both global assets are loaded **once, at the end of `layout/theme.liquid` (lines ~299–300)**, after
Dawn's `base.css`/`global.js`. They therefore apply site-wide and override Dawn.

### The premium design system (read these two files first)

`assets/premium-custom.css` defines the shared visual language:
- **Design tokens** in `:root` (`--brand:#00C7B7`, `--brand-2`, `--ink`, `--glass`, `--shadow`, …)
- **Utility classes** reused across all premium sections: `.container`, `.eyebrow`, `.btn` /
  `.btn-primary` / `.btn-ghost`, `.glass`, `.grad-text`, `.blob`, `.tilt`, and `[data-reveal]`.
- Imports Google Fonts (Inter + Space Grotesk).

`assets/premium-custom.js` wires interactive behavior **by selector / data-attribute**, so simply
adding the right class or attribute in any section activates it globally — no per-section JS needed:
- `[data-reveal]` (+ `data-reveal-delay="1..4"`) → fade/slide-in on scroll (adds `.in`)
- `[data-counter]` → animated number count-up
- `.tilt` → 3D mouse-tilt; `.magnetic` → magnetic-pull buttons
- `[data-parallax]` → scroll parallax; plus a custom cursor (`.cursor-dot`/`.cursor-ring`)
- `.compare` (with child `.after` img + `.handle`) → draggable before/after slider (mouse + touch)
- `[data-multi-form]` → multi-step form (`[data-step]`, `[data-next]`, `[data-prev]`), with
  per-step required-field validation and a `[data-honeypot]` anti-spam guard
- `[data-pp-thumb]` (with `data-full`) + `[data-pp-main]` → product gallery click/keyboard image
  swap, with an opacity fade (the main img also transitions `opacity`)
- `[data-pp-form]` + a `[data-pp-variants]` JSON `<script>` → reactive variant selection
  (updates `[data-pp-id]`, `[data-pp-price]`, `[data-pp-stock*]`, `[data-pp-atc]`, `[data-pp-buynow]`);
  also **strikes-through/dims unavailable option pills** (`[data-pp-opt-pill]`, radios carry
  `data-opt-pos`), and AJAX add-to-cart (updates `#cart-icon-bubble`, shows `.pp-toast`, native fallback)
- `[data-pp-qty]` (wrapper with `[data-qty="minus|plus"]` buttons around a number input) → quantity
  stepper. **Use this instead of inline `onclick`** — the premium layer has no inline JS handlers.
- `#pp-back-to-top` → global scroll-to-top button (appears after 500px; hidden while a Dawn overlay
  locks `body.overflow-hidden`; uses `window.lenis` if present, else native smooth scroll)

When building a premium section, prefer these existing hooks over writing new JS. The `data-pp-*`
namespace is reserved for premium **product-page** hooks. Premium sections use **no inline event
handlers** — wire behavior via these data-attributes or a section `<script>`.

### Premium sections are self-contained

Each `sections/premium-*.liquid` bundles its **section-specific** `<style>`, optional `<script>`
(IIFE scoped via `document.currentScript.closest('section')`), and `{% schema %}` inside the one
file. Only genuinely shared styles/behaviors belong in the global `premium-custom.*` assets.

### Where pages are composed

- `templates/index.json` (homepage) is built **entirely** from premium sections in this order:
  `premium-hero → premium-problems → premium-products → premium-science → premium-reviews →
  premium-demo-form → premium-footer`.
- `templates/product.premium.json` is an alternate product template using `premium-product`
  (with FAQ blocks) instead of Dawn's `main-product`. Assign it to a product in the admin to use it.
- `templates/collection.premium.json` → `premium-collection.liquid` (assignable per-collection):
  premium grid + server-side `collection.filters` sidebar + sort + pagination.
- `templates/404.json` → `premium-404.liquid` (branded glassmorphism 404).
- `templates/cart.json` → `premium-cart.liquid`. **Note:** Shopify has no per-cart alternate
  template (the cart page always uses `cart.json`), so the premium cart is wired directly into
  `cart.json` rather than a `cart.premium.json`. It's a standard cart form (`updates[]` + `name="checkout"`
  + `item.url_to_remove`) with auto-submit on quantity change.
- `premium-demo-form.liquid` is a lead-capture form using Shopify's `{% form 'contact' %}` — it
  sends to the store's contact/notification email (no app or external endpoint) and shows a thank-you
  via `form.posted_successfully?`.
- `snippets/seo-meta.liquid` (rendered in `<head>` after `meta-tags`) adds **JSON-LD**: Organization +
  WebSite on every page, and Product on product pages (brand "D'Cal", INR price, availability, static
  4.8/12840 aggregateRating fallback). OG/Twitter tags still come from Dawn's `meta-tags.liquid`.

### Globally-rendered elements (in `layout/theme.liquid`)

These render on **every** page, not just the homepage:
- `{% section 'premium-promo-bar' %}` — `sections/premium-promo-bar.liquid`, a dismissible top
  announcement bar (message + optional link + dismissible toggle). Dismissal is stored in
  `localStorage` under `dcal_promo_dismissed_v1_<handleized-message>`, so **editing the message
  re-shows it** to everyone. Dawn's stock `announcement-bar` was removed from `header-group.json` to
  avoid two stacked bars — so the body grid is back to `auto auto 1fr auto` (promo, header, main, footer).
- `<button id="pp-back-to-top">` — the floating scroll-to-top button (styled `.pp-back-to-top`).

> The `*.json` template files carry an auto-generated header: the **theme editor can overwrite
> them**. Prefer changing section *defaults/logic* in the `.liquid` files; treat JSON as content/config.

## Design conventions

- **No emojis anywhere in the codebase, ever** (not in markup, JS strings, or comments). The one
  known exception is Dawn's stock `assets/component-rating.css` (`content:"★★★★★"`), which is dormant
  — premium pages render their own star icons.
- **All icons are inline Lucide-style SVGs** rendered via `{% render 'icon', name: '...', size: N %}`
  (`snippets/icon.liquid`). No icon font, no CDN. Pattern: `viewBox="0 0 24 24"`, `stroke="currentColor"`,
  `stroke-width:2`, `fill:none` (except `star`, which is filled), `aria-hidden="true"`. Icons **inherit
  colour via `currentColor`** — set the parent's `color`. To add an icon, add a `when` case to the snippet.
- For markup built in **JS** (cart toast, reactive variant button labels), use the `PPICON` SVG-string
  constants at the top of `premium-custom.js` (keep them in sync with the snippet).
- **Header** uses Dawn's stock `header` section (`header-group.json`), configured `sticky_header_type:
  on-scroll-up` (hides on scroll-down, reveals on scroll-up). Dawn's header already uses inline SVG
  icons, so it stays emoji-free. (A custom `premium-header` was built and then reverted per request.)
- **Promo bar** uses solid `#0B1220`, a `sparkles` icon, white 13px text, and a highlighted code chip
  (`.promo-code`) driven by the `highlight_code` setting.

## Conventions & gotchas

- **Product picker settings return a product object, not a handle.** For a `{"type":"product"}`
  setting, use `block.settings.product` (or `section.settings.x`) directly — e.g.
  `{% assign p = block.settings.product %}`. Do **not** do `all_products[block.settings.product]`;
  feeding a product object into `all_products[...]` (which expects a handle string) yields a blank/empty
  drop, which then surfaces downstream as `Liquid error: invalid url input` on `image_url`.
- Guard image rendering: only call `... | image_url: width: N` when the image exists
  (`{% if p.featured_image %}`), and provide an emoji/placeholder fallback otherwise — the premium
  cards already follow this image / no-image / no-product three-state pattern.
- Content is brand- and region-specific (D'Cal, India). Match the existing tone and locale when
  editing copy.
- **Footer link columns** parse a `Label | URL` syntax per line (e.g. `Water Softener | /products/x`);
  a line with no `|` renders as plain text. Footer **social** icons only render when their URL is set.
- **Sold-out / variant safety (product page):** `premium-product.liquid` renders the correct
  disabled/sold-out button state server-side from `product.available`; the `data-pp-*` JS then keeps
  it in sync per-variant. Keep both in agreement when editing — don't rely on JS alone.
- **`premium-product-reviews.liquid` is a marquee seed section** for product pages (wired into
  `templates/product.premium.json` between `main` and `problems`). It reuses the shared
  `.marquee` / `.marquee__track` classes from `premium-custom.css`, but runs slower (40s vs the
  homepage's 35s) and pauses on hover; cards are 380px (300px under 768px). It honours
  `prefers-reduced-motion` by zeroing the animation. Stars are inline SVG (not the icon snippet)
  because the design uses a colored fill that differs from the snippet's `currentColor` rule.
  This is intentionally a seed dataset of 10 hand-written reviews — replace with Judge.me real
  reviews when the app is wired in (delete the section or hide all blocks).
- **Premium Problems cards support an optional per-block image** (`block.settings.image`). When set,
  the card switches to image-top layout (16:10 image, 4:3 on mobile) with the Lucide icon shrunk to
  a 40px gradient badge in the image's top-left corner — the icon is preserved as a visual accent.
  When no image is uploaded, the card falls back to the original icon-only design (larger Lucide
  icon in a colored tile, no padding change). Both layouts share `.pp-problem-card`; the
  `.pp-problem-card-img` modifier flips padding/flex/overflow for the image variant.
- **Shared CSS in `premium-custom.css`:** a unified chevron applies to every `select` / `input[list]`;
  `section[id]` gets `scroll-margin-top:96px` (so `#science`/`#demo-form` anchors clear the sticky
  header + promo bar — don't re-add inline `scroll-margin-top`); `:focus-visible` outlines are defined
  for premium buttons/thumbs/inputs; `prefers-reduced-motion` disables the heavy animations.

## User-managed setup (Shopify admin — not code)

These are content/config the merchant must complete; the code is ready for them:
- **Main menu**: Online Store → Navigation → *Main menu*. The header (`section.settings.menu`)
  renders whatever links exist here — add links to the product, collections, etc.
- **Footer links**: edit each *Premium Footer* "Link column" block using the `Label | URL` syntax;
  add real URLs to the *Social* blocks (empty = hidden).
- **Premium Products section**: pick a real product in each *Product card* block (empty blocks show
  the placeholder card by design).
- **Before/after slider** (`premium-science`): upload *Before image* and *After image* in the
  section settings — the slider only activates when both exist.
- **Hero image**: upload via the *Premium Hero* section settings.

## TODO / not-yet-done (future sessions)

- **Lenis smooth scroll was intentionally NOT added.** It was requested via the `unpkg` CDN, which
  conflicts with the "no new dependencies / vanilla only" rule and adds a third-party runtime script
  to a live storefront. Native smooth scroll (`scroll-behavior:smooth` + `window.scrollTo`) is used
  instead, and `#pp-back-to-top` already calls `window.lenis` *if present*. To enable Lenis, vendor it
  locally into `assets/` (don't hot-link a CDN) and set `window.lenis`.
- 🟢 Move the Google Fonts `@import` in `premium-custom.css` to a `<link rel="preconnect">` +
  stylesheet in `theme.liquid` `<head>` for faster first paint.
- 🟢 Optional polish: pause marquee on hover; replace repeated inline hex with the CSS `--brand*`
  tokens; hide the demo-form steps after a successful submit.
- 🟢 `premium-collection` filters are server-side (full page reload per filter) — could be AJAX-ified.
- ⚠️ Deleting unused Dawn sections was **deliberately skipped** (risk of breaking the editor / other
  templates outweighs the benefit). Revisit only with a careful reference check.
- See `TODO.md` in the repo root for everything that must be done in the Shopify admin (not code).

_Done in the catch-up pass:_ branded 404, premium cart, premium collection template, promo bar
(message/link/dismissible), back-to-top, gallery swap + fade, reactive variants + unavailable-pill
strike-through, quantity stepper (no inline `onclick`), honeypot, JSON-LD, focus states, chevron,
anchor offsets, reduced-motion.
