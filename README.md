# PM Digital — Dev Test: Custom PDP Bundle Banner Section

Shopify theme customization built from a Figma design spec. Implements a custom Bundle Banner hero section, floating header card, and announcement bar styling — all integrated into a Shopify Online Store 2.0 theme.

## Preview

- **Shopify store:** *(add your dev store preview link here)*
- **Loom walkthrough:** *(add your Loom video link here)*
- **Figma reference:** [Design Spec](https://www.figma.com/design/qu4qtym3QCMaLQKauRciHO/Platter-Technical-Challenge--25--Theme-Developers?node-id=0-1&p=f&t=wdL37JgaBslTE0ON-0)

## Approach

### Bundle Banner Section (`sections/bundle-banner.liquid`)

The main deliverable — a standalone Shopify section built mobile-first with vanilla CSS.

**Key decisions:**

- **Real `<img>` tags instead of CSS `background-image`** — Early iterations used CSS custom properties (`var()`) with `background-image` layers, but this approach had rendering issues in Shopify's `{% stylesheet %}` context. Switching to semantic `<img>` elements solved the gradient overlay problem and improved accessibility.
- **`::before` pseudo-element for gradient overlays** — A pseudo-element on the product image wrapper creates the dark-to-transparent fade that blends products into the background. Direction changes per breakpoint: `to right` on desktop, `to top` on mobile.
- **Responsive image loading** — Both images use `srcset` + `sizes` for optimal file delivery, with `loading="eager"` and `fetchpriority="high"` since the banner is above the fold.
- **Scoped dynamic styles** — The highlight color is injected via a `<style>` tag scoped to the section ID, while static styles use `{% stylesheet %}` for theme-level caching.

**Schema fields:** heading (inline richtext with `<em>` highlight support), subheading, CTA label, CTA link, background image, product image, highlight color.

### Floating Header (`assets/custom-header.css`)

Custom CSS overrides that transform the theme's default header into a floating rounded card.

**Key decisions:**

- **ID-based specificity strategy** — Uses `#header-component` selectors (specificity 1,x,0) to override theme class-based rules (0,x,0) without `!important`.
- **Responsive border-radius** — `clamp(24px, 3.125vw, 45px)` scales from mobile to desktop.
- **Viewport margin** — `max-width: calc(100% - var(--page-margin) * 2)` keeps the header inset from edges at all screen sizes.
- **Mobile adjustments** — Logo centered, account icon hidden, search icon repositioned to right side via CSS grid-area.

### Announcement Bar

Styled with underlined text and WCAG-compliant minimum font size (12px on mobile).

## Folder Structure

```
pm-digital-dev-test/
├── sections/
│   └── bundle-banner.liquid      # Custom bundle banner section (main deliverable)
├── assets/
│   ├── custom-header.css         # Floating header + announcement bar overrides
│   └── icon-menu.svg             # Custom menu icon
├── snippets/
│   └── stylesheets.liquid        # CSS asset loading (loads custom-header.css)
├── config/
│   └── settings_data.json        # Theme settings (color schemes, typography)
├── templates/
│   └── index.json                # Homepage template wiring
└── sections/
    └── header-group.json         # Header configuration (announcement bar + header)
```

## Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/JulianOspinaLondono/pm-digital-dev-test.git
   cd pm-digital-dev-test
   ```

2. **Connect to your Shopify dev store**
   ```bash
   shopify theme dev --store your-store.myshopify.com
   ```

3. **Configure the Bundle Banner in the theme customizer:**
   - Add the "Bundle Banner" section to a page
   - Upload a background image (dark teal with glow effect)
   - Upload a product image (PNG with transparent background recommended)
   - Set heading, subheading, CTA label, and CTA link

No build step required — all CSS is vanilla, no frameworks or preprocessors.

## Technical Details

| Area | Implementation |
|------|---------------|
| CSS approach | Mobile-first, vanilla CSS, no frameworks |
| JavaScript | None required — pure CSS layout |
| Images | `<img>` with `srcset`/`sizes` for responsive loading |
| Accessibility | `aria-label` on section, `role="presentation"` on decorative images, semantic HTML5 |
| Breakpoint | 750px (matches theme's native breakpoint) |
| Performance | `loading="eager"` + `fetchpriority="high"` for above-the-fold assets |

## AI Tools Usage

Claude (via Claude Code CLI) was used as a development assistant throughout this project:

- **Architecture decisions** — Helped evaluate approaches (CSS background-image vs real `<img>` tags) and pivot when the first approach had rendering issues
- **CSS debugging** — Identified specificity conflicts with the theme's header styles and suggested ID-based selector strategy
- **Accessibility review** — Flagged missing ARIA attributes, decorative image roles, and WCAG font-size compliance (10px was below minimum)
- **Code cleanup** — Removed dead CSS from earlier iterations, fixed typos in comments
- **Responsive audit** — Identified fixed values that should use `clamp()` for fluid scaling

All code was reviewed, tested, and adjusted manually. AI was used to move faster and catch issues — not to generate the entire solution blindly.



## Task 2 — A/B Test Spec: Bundle Banner vs. Hero (Written Spec)

### Tool Choice & Reasoning

For this test I would use **Convert.com** (or GrowthBook as an open-source alternative).

I have hands-on experience with both Convert.com and GrowthBook across multiple client projects, and they are the platforms I am most comfortable configuring end-to-end — from experiment setup to metric instrumentation and result analysis. I have also worked with Intelligems and VWO, but Convert.com stands out for three reasons: its visual editor and URL targeting interface are intuitive enough to move fast without sacrificing precision; its support team is responsive and technically strong; and the experiment engine has been consistently reliable in my experience, with clean data and minimal flicker issues when the snippet is implemented correctly.

Intelligems is purpose-built for Shopify price testing, which makes it excellent for revenue-focused experiments, but it is more opinionated in its setup and less flexible for purely content-level tests like this one. Convert.com gives us more control over targeting logic, event definitions, and segmentation, which is important for a test where clean data integrity is the priority.

### URL Targeting Rules

The test must run exclusively on Product Detail Pages (PDPs) and must not affect the homepage, collection pages, or any utility/modal views.

**Include rules** — ALL must match:

| Rule | Condition | Value |
|------|-----------|-------|
| URL path | Contains | `/products/` |
| URL path | Does NOT start with | `/collections/` |
| URL path | Does NOT equal | `/` (homepage) |

**Exclude rules:**

| Rule | Condition | Value |
|------|-----------|-------|
| URL | Contains | `?view=` |
| URL | Contains | `/products` (exact match, no trailing slash — catches the `/products` listing page if one exists) |

This combination ensures the experiment fires only on individual product URLs (e.g. `/products/my-product-handle`) and excludes the root `/products` page, any alternate template views via `?view=`, and all collection or homepage traffic.

### Traffic Split & Minimum Sample Size

The test will run on a **50/50 traffic split** between Control (existing hero section) and Variation (bundle banner).

For sample size estimation, assuming a baseline add-to-cart rate of approximately 5–8% on PDPs (a reasonable Shopify e-commerce benchmark), and targeting a minimum detectable effect of 15% relative lift with 80% statistical power at a 95% confidence level, the estimated minimum sample size is roughly **2,500–4,000 unique visitors per variant** before results can be considered statistically reliable. Using a tool like Evan Miller's Sample Size Calculator or Convert.com's built-in estimator is recommended to plug in actual baseline data once the test is live.

A hard end date of **30 days** should be set regardless of significance. If significance is not reached by then, results should be treated as directional rather than conclusive.

### Success Metrics

**Primary metric — Add-to-Cart Rate**

Defined as the percentage of PDP sessions in which a user triggers an `add_to_cart` event (Shopify's native Analytics event or a custom dataLayer push, depending on the tracking setup). This metric directly measures whether the bundle banner drives more purchase intent at the product level.

**Secondary metric — Revenue Per Session**

Defined as total revenue attributed to sessions that included a PDP visit, divided by the total number of qualifying sessions in each variant. This captures whether the variation not only increases cart additions but also improves overall order value — which is the core business goal behind the bundle offer.

Both metrics should be scoped to the same session in which the variant was served, and tracked via Convert.com's goal configuration connected to the Shopify Analytics dataLayer or a custom event listener.

### QA Process Before Launch

1. **Preview link verification** — Use Convert.com's Force Variation feature (or a preview URL parameter) to load the variation directly in an incognito window. Confirm the bundle banner renders correctly on desktop and mobile, with no layout breaks, missing images, or CTA issues.
2. **Control page verification** — In a separate incognito session, load the control variant and confirm the original hero section is unaffected. Navigate to the homepage, a collection page, and a non-product URL to confirm the experiment does not fire on excluded pages.
3. **Event tracking validation** — Add a product to cart while in each variant and confirm that the `add_to_cart` goal fires in Convert.com's Live Log (or equivalent real-time debug view). Verify that no duplicate events are being sent.
4. **Cross-page targeting check** — Browse to `/collections/`, the homepage, and a `/products` listing page (if it exists in the store) while in the variation and confirm the bundle banner does not appear on any of these — validating that the exclusion rules are working correctly.
5. **Mobile QA** — Repeat steps 1–3 on a real mobile device or via Chrome DevTools device emulation to confirm the banner is responsive and the CTA is tappable above the fold or within one scroll.

### Risks & Edge Cases

| Risk | Description | Mitigation |
|------|-------------|------------|
| Returning visitors | A visitor who saw the control on visit 1 might see the variation on visit 2, contaminating the data | Ensure the platform assigns variants via a persistent cookie with a TTL of ≥30 days to maintain consistent assignment across sessions |
| Mobile vs. desktop split | The bundle banner may perform differently on mobile due to smaller viewports and different scroll behavior | Segment results by device type post-test. If a significant interaction effect is found, consider a follow-up device-specific test |
| Low traffic / underpowered test | If PDP traffic is lower than estimated, the test may run for weeks without reaching significance | Set a hard 30-day end date. Treat results as directional if significance is not reached. Do not extend indefinitely |
| Novelty effect | Early lift in the variation may be driven by curiosity about the new design rather than genuine preference | Do not call the test within the first 3–4 days. Monitor weekly cohorts to check whether lift is stable over time |
| Bundle out of stock | If the bundle collection is empty or the offer changes mid-test, the variation becomes misleading | Add a monitoring alert for bundle product availability. Pause the test if the bundle offer changes significantly |
| Shopify theme caching | Shopify's CDN may cache the control variant and serve it incorrectly to variation users | Ensure the platform injects the variant at the JavaScript/DOM level after page load, or use server-side rendering hooks if available in the theme setup |
| Cart attribution across sessions | A user who adds to cart in session 1 and completes purchase in session 2 may be misattributed | Verify that the platform handles this via persistent session cookies and order-level attribution during QA Step 3 |

**I used AI jus to rewrite and rephrase here**