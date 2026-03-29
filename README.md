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
