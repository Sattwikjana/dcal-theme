# D'Cal — User TODO

Everything below must be done by **you** in the Shopify admin or externally — Claude cannot do these
from code. The theme code itself is ready for all of it.

> **Current state:** the theme **"Dawn" (#140491882599) is the live/published theme.** Changes in
> this repo are pushed to it with `shopify theme push --theme 140491882599`. There is no separate
> draft — edits go live, so preview on an unpublished copy first if you want a safety net.

---

## 🔧 SHOPIFY ADMIN (admin.shopify.com)

- [ ] **Navigation:** Online Store → Navigation → *Main menu* → add items (Shop, Science, Reviews, Demo Kit, Contact). The header already renders whatever is in Main menu.
- [ ] **Header menu link:** Customize → Header → confirm *Menu* = Main menu (default is already `main-menu`).
- [ ] **Form email:** Settings → Notifications → confirm the address that receives **contact-form** submissions (the demo form posts there).
- [ ] **Domain:** Settings → Domains → connect your custom domain (e.g. globalshopper.in).
- [ ] **Payments:** Settings → Payments → set up Razorpay / Stripe for INR.
- [ ] **Shipping:** Settings → Shipping → add zones for India + enable COD.
- [ ] **Taxes:** Settings → Taxes & duties → enable GST.
- [ ] **Products:** upload real product images (≥1200×1200, square, white/transparent bg).
- [ ] **Product template:** for each product → Online Store template → choose **`product.premium`**.
- [ ] **Collections:** (optional) assign the **`collection.premium`** template to a collection to use the new premium grid + filters.
- [ ] **Collection filters:** Search & Discovery app (free, by Shopify) → add filters so the premium-collection sidebar has options.
- [ ] **Hero image:** Customize → *Premium Hero* → upload hero image.
- [ ] **Before/after slider:** Customize → *Premium Science* → upload Before + After images (slider stays inert until both are set).
- [ ] **Premium Products:** Customize → *Premium Products* → pick a real product in each card (empty cards show a placeholder by design).
- [ ] **Footer links:** Customize → *Premium Footer* → each "Link column" uses `Label | URL` (e.g. `Water Softener | /products/dcal-overhead-tank-water-softener-cartridge`); add URLs to *Social* blocks (empty = hidden).
- [ ] **Promo bar:** Customize → *Premium Promo Bar* → edit message / link / dismissible toggle (changing the message re-shows it to everyone).
- [ ] **Favicon:** Customize → Theme settings → Favicon.
- [ ] **Logo:** Customize → Theme settings → Logo (currently text-only "D'Cal"). Adding a logo also populates the Organization JSON-LD logo.
- [ ] **Review all copy** in every section.

## 📱 FREE APPS TO INSTALL
- [ ] Shopify Inbox — live chat
- [ ] Judge.me — product reviews (free plan; the product page already has Judge.me widget placeholders)
- [ ] Shopify Search & Discovery — collection filters + better search (free, official)
- [ ] Plug In SEO — SEO audit (free plan)
- [ ] Pandectes / GDPR Cookie Banner — consent (free)

## 🖼️ CONTENT TO CREATE
- [ ] **Hero image** — e.g. "A single crystal-clear water droplet about to hit a still mirror-like water surface, ultra-macro, hyper-realistic, soft cinematic lighting upper-left, teal-cyan gradient background, 8K, --ar 4:5"
- [ ] **Product image — Cartridge** — "sleek cylindrical white-and-teal water softener cartridge on soft pastel cyan background, premium product photography, --ar 1:1"
- [ ] **Product image — Washing Ball**
- [ ] **Product image — Tile Cleaner**
- [ ] **Before / After** science images for the Premium Science slider
- [ ] Real customer reviews (or keep placeholders until Judge.me collects real ones)

## 🔌 INTEGRATIONS (lower priority)
- [ ] Meta Pixel + Conversions API
- [ ] Google Analytics 4
- [ ] Google Tag Manager (if needed)
- [ ] Klaviyo / Mailchimp for email
- [ ] WhatsApp Business for order updates
- [ ] Shiprocket / Delhivery for shipping

## 🚀 LAUNCH CHECKLIST
- [ ] Test product page(s) on mobile (variant pills, add-to-cart toast, gallery swap)
- [ ] Submit the demo form end-to-end and confirm it lands in Settings → Notifications inbox
- [ ] Add to cart → confirm cart bubble updates + cart page (premium) shows the item
- [ ] Complete a test checkout with a test payment
- [ ] Verify mobile menu + all anchor links (#science, #demo-form) scroll cleanly under the header
- [ ] Run Lighthouse — aim for 90+ on Performance/SEO/Accessibility
- [ ] Test on slow 3G
- [ ] Cross-browser: Safari, Chrome, Firefox
- [ ] Submit sitemap in Google Search Console

---

### Code decision to be aware of
- **Lenis smooth scroll was not added** (it required hot-linking a CDN, against the no-dependencies
  rule). Native smooth scroll is in place. If you want Lenis, vendor it into `assets/` and set
  `window.lenis` — the back-to-top button already uses it when present.
