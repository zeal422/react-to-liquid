---
name: react-to-liquid
description: >
  Converts full React applications and pages into production-ready Shopify Online Store 2.0 theme code using Liquid templating.
  Use this skill whenever a user wants to: migrate a React site or component to Shopify, convert JSX/TSX files to Liquid sections or snippets,
  turn a React UI into a Shopify theme, make React code work inside Shopify, or produce merchant-editable Shopify theme sections with schema settings.
  Trigger even when the user says things like "port this to Shopify", "make this a Shopify theme", "convert my React store to Shopify", 
  or shares React/JSX code and mentions Shopify in any context. Always use this skill for React-to-Shopify conversions — do NOT attempt
  these conversions without it, as the mapping rules and schema conventions are detailed and easy to get wrong.
---

# React → Shopify Liquid Skill

Converts full React apps/pages into production-ready **Shopify Online Store 2.0** theme files, with merchant-editable `{% schema %}` settings.

---

## Workflow Overview

Follow these phases in order:

1. **Audit** — Understand the React codebase
2. **Map** — Decide what becomes a section, snippet, template, or asset
3. **Convert** — Translate React constructs to Liquid equivalents
4. **Schema** — Generate merchant-editable `{% schema %}` blocks
5. **Output** — Produce correctly structured theme files
6. **SVGs** — Convert icon SVGs to snippets for dynamic color control
7. **JavaScript** — Convert React state/events to vanilla JS + Ajax API
8. **Review** — Flag anything requiring manual intervention

---

## Phase 1: Audit the React Code

Before writing any Liquid, analyze the React source:

- Identify all **pages** (React Router routes, Next.js pages, etc.) → each becomes a `templates/*.json` + `sections/*.liquid`
- Identify all **reusable components** → each becomes a `snippets/*.liquid`
- Identify all **static assets** (images, fonts, SVGs) → go to `assets/`
- Identify all **data sources**:
  - Hardcoded data → convert to schema `default` values
  - API calls to Shopify (products, collections, cart) → replace with Liquid objects
  - External API calls → flag for manual handling (Shopify Apps or Metafields)
- Identify all **state** (useState, useReducer, Redux, Zustand):
  - UI state (open/close, tabs, accordions) → JavaScript in `assets/*.js`
  - Data state (cart, wishlist) → Shopify Ajax API / Section Rendering API
  - User-set preferences → Liquid + Cookies or Metafields
- Identify all **props** passed between components → become snippet parameters via `{% render %}` with variables

---

## Phase 2: Map React Structure → Shopify Theme Structure

Use this mapping table:

| React Concept | Shopify Equivalent |
|---|---|
| Page component (`/pages/index.jsx`) | `templates/index.json` + `sections/index-content.liquid` |
| Reusable component (`<ProductCard />`) | `snippets/product-card.liquid` |
| Layout component (`<Layout />`) | `layout/theme.liquid` |
| Global CSS / Tailwind | `assets/base.css` |
| Component CSS Module | `assets/component-name.css` — **must** be loaded at the top of the section/snippet with `{{ 'component-name.css' | asset_url | stylesheet_tag }}` |
| `useEffect` fetching Shopify data | Liquid objects (`product`, `collection`, `cart`) |
| React Context (theme, cart) | Liquid globals + `window.__st` / Ajax API |
| `.env` variables / config | Shopify theme settings (`settings_schema.json`) |
| SVG icon (small, reusable) | `snippets/icon-[name].liquid` — inline SVG so fill/stroke can be controlled via Liquid variables |
| SVG illustration (large, static) | `assets/[name].svg` — reference with `{{ '[name].svg' | asset_url }}` |
| Dynamic routing (`/products/[slug]`) | `templates/product.json` (Shopify handles routing) |
| `<Link to="...">` | `<a href="{{ routes.all_products_collection_url }}">` |

---

## Phase 3: Convert React → Liquid Syntax

### JSX → Liquid HTML

```jsx
// React
function Hero({ title, subtitle, ctaText, ctaUrl, backgroundImage }) {
  return (
    <section style={{ backgroundImage: `url(${backgroundImage})` }}>
      <h1>{title}</h1>
      <p>{subtitle}</p>
      <a href={ctaUrl}>{ctaText}</a>
    </section>
  );
}
```

```liquid
{% comment %} Liquid - sections/hero.liquid {% endcomment %}
<section style="background-image: url('{{ section.settings.background_image | image_url: width: 1920 }}')">
  <h1>{{ section.settings.title }}</h1>
  <p>{{ section.settings.subtitle }}</p>
  <a href="{{ section.settings.cta_url }}">{{ section.settings.cta_text }}</a>
</section>
```

### Conditional Rendering

```jsx
// React
{isLoggedIn && <AccountMenu />}
{items.length > 0 ? <CartItems /> : <EmptyCart />}
```

```liquid
{% if customer %}
  {% render 'account-menu' %}
{% endif %}

{% if cart.item_count > 0 %}
  {% render 'cart-items' %}
{% else %}
  {% render 'cart-empty' %}
{% endif %}
```

### Lists / `.map()`

```jsx
// React
{products.map(product => <ProductCard key={product.id} product={product} />)}
```

```liquid
{% for product in collection.products %}
  {% render 'product-card', product: product %}
{% endfor %}
```

### Props → Snippet Variables

```jsx
// React - passing props
<ProductCard product={product} showQuickBuy={true} imageRatio="square" />
```

```liquid
{% comment %} Liquid - caller passes variables to snippet {% endcomment %}
{% render 'product-card',
  product: product,
  show_quick_buy: true,
  image_ratio: 'square'
%}
```

```liquid
{% comment %} snippets/product-card.liquid - receives variables {% endcomment %}
<div class="product-card">
  {{ product.featured_image | image_url: width: 600 | image_tag }}
  <h3>{{ product.title }}</h3>
  <p>{{ product.price | money }}</p>
  {% if show_quick_buy %}
    <button data-product-id="{{ product.id }}">Quick Buy</button>
  {% endif %}
</div>
```

### Shopify Liquid Objects Reference

| React Data Source | Liquid Equivalent |
|---|---|
| Product from API | `product.title`, `product.price`, `product.variants`, `product.images` |
| Collection | `collection.products`, `collection.title`, `collection.description` |
| Cart | `cart.items`, `cart.total_price`, `cart.item_count` |
| Customer | `customer.first_name`, `customer.email`, `customer.orders` |
| Shop info | `shop.name`, `shop.currency`, `shop.email` |
| All products | `collections['all'].products` |
| Blog posts | `blogs[blog_handle].articles` |
| Navigation menus | `linklists['main-menu'].links` |
| Current page URL | `canonical_url` |
| Routes | `routes.cart_url`, `routes.account_url`, `routes.all_products_collection_url` |

---

## Phase 4: Generate {% schema %} Blocks

This is the most important phase for merchant-editability. Every `sections/*.liquid` file MUST end with a `{% schema %}` block.

### Schema Anatomy

```liquid
{% schema %}
{
  "name": "Section Display Name",
  "tag": "section",
  "class": "section-css-class",
  "settings": [
    {% comment %} Settings array - see types below {% endcomment %}
  ],
  "blocks": [
    {% comment %} Repeatable child blocks (optional) {% endcomment %}
  ],
  "max_blocks": 12,
  "presets": [
    {
      "name": "Default preset name"
    }
  ]
}
{% endschema %}
```

### Schema Setting Types

Map React props to schema settings using this guide:

| React Prop Type | Schema Setting Type | Example |
|---|---|---|
| `string` (short text) | `"type": "text"` | Button label, heading |
| `string` (long text) | `"type": "textarea"` | Description, bio |
| `string` (rich text) | `"type": "richtext"` | Body copy with formatting |
| `boolean` | `"type": "checkbox"` | Show/hide toggle |
| `number` | `"type": "number"` | Item count, padding |
| `string` (from enum) | `"type": "select"` | Layout style, alignment |
| `string` (color) | `"type": "color"` | Background, text color |
| image/url | `"type": "image_picker"` | Hero image, logo |
| video | `"type": "video"` | Background video |
| product | `"type": "product"` | Featured product |
| collection | `"type": "collection"` | Featured collection |
| URL/link | `"type": "url"` | CTA button link |
| font | `"type": "font_picker"` | Custom font |
| range slider | `"type": "range"` | Opacity, columns count |

### Full Schema Example — Hero Section

```liquid
{% schema %}
{
  "name": "Hero Banner",
  "tag": "section",
  "class": "hero-banner",
  "settings": [
    {
      "type": "image_picker",
      "id": "background_image",
      "label": "Background Image"
    },
    {
      "type": "text",
      "id": "title",
      "label": "Heading",
      "default": "Welcome to our store"
    },
    {
      "type": "textarea",
      "id": "subtitle",
      "label": "Subheading",
      "default": "Shop the latest collection"
    },
    {
      "type": "text",
      "id": "cta_text",
      "label": "Button Label",
      "default": "Shop Now"
    },
    {
      "type": "url",
      "id": "cta_url",
      "label": "Button Link"
    },
    {
      "type": "select",
      "id": "text_alignment",
      "label": "Text Alignment",
      "options": [
        { "value": "left", "label": "Left" },
        { "value": "center", "label": "Center" },
        { "value": "right", "label": "Right" }
      ],
      "default": "center"
    },
    {
      "type": "color",
      "id": "overlay_color",
      "label": "Overlay Color",
      "default": "#000000"
    },
    {
      "type": "range",
      "id": "overlay_opacity",
      "label": "Overlay Opacity",
      "min": 0,
      "max": 100,
      "step": 5,
      "unit": "%",
      "default": 40
    }
  ],
  "presets": [
    {
      "name": "Hero Banner"
    }
  ]
}
{% endschema %}
```

### Blocks — For Repeatable Content

Use blocks when React maps over a dynamic array that merchants should control (e.g., testimonials, feature cards, slides):

```liquid
{% comment %} In the section Liquid template {% endcomment %}
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'testimonial' %}
      <blockquote {{ block.shopify_attributes }}>
        <p>{{ block.settings.quote }}</p>
        <cite>{{ block.settings.author }}</cite>
      </blockquote>
  {% endcase %}
{% endfor %}

{% schema %}
{
  "name": "Testimonials",
  "blocks": [
    {
      "type": "testimonial",
      "name": "Testimonial",
      "settings": [
        {
          "type": "textarea",
          "id": "quote",
          "label": "Quote"
        },
        {
          "type": "text",
          "id": "author",
          "label": "Author Name"
        },
        {
          "type": "image_picker",
          "id": "avatar",
          "label": "Author Photo"
        }
      ]
    }
  ],
  "presets": [{ "name": "Testimonials" }]
}
{% endschema %}
```

---

## Phase 5: Output File Structure

Generate files in this exact OS 2.0 structure:

```
theme/
├── layout/
│   └── theme.liquid          ← <html>, <head>, global nav/footer
├── templates/
│   ├── index.json            ← Homepage — lists sections to render
│   ├── product.json          ← Product page
│   ├── collection.json       ← Collection/category page
│   ├── cart.json             ← Cart page
│   ├── page.json             ← Generic CMS page
│   └── customers/
│       ├── login.json
│       └── account.json
├── sections/
│   ├── header.liquid         ← Nav (with schema for logo, menu)
│   ├── footer.liquid         ← Footer links/copyright
│   ├── hero-banner.liquid    ← One file per converted page section
│   ├── featured-products.liquid
│   └── [etc].liquid
├── snippets/
│   ├── product-card.liquid   ← One file per reusable component
│   ├── cart-item.liquid
│   └── [etc].liquid
├── assets/
│   ├── base.css              ← Global styles (loaded in layout/theme.liquid)
│   ├── component-[name].css  ← Per-component styles — MUST be linked at the top of the section/snippet that uses it
│   └── [name].js             ← Interactive JS (no React/JSX)
└── config/
    └── settings_schema.json  ← Global theme settings
```

### CSS Loading Rule

Every section or snippet that has a dedicated CSS file **must** link it at the very top of the Liquid file:

```liquid
{% comment %} sections/hero-banner.liquid {% endcomment %}
{{ 'component-hero-banner.css' | asset_url | stylesheet_tag }}

<section class="hero-banner">
  ...
</section>
```

Never generate a `component-*.css` file without also adding the corresponding `stylesheet_tag` line in the Liquid file that uses it. Global styles go in `layout/theme.liquid` only.

---

### Template JSON Format (OS 2.0)

Each template JSON file declares which sections appear on that page:

```json
{
  "sections": {
    "hero": {
      "type": "hero-banner",
      "settings": {
        "title": "Welcome",
        "cta_text": "Shop Now"
      }
    },
    "featured-products": {
      "type": "featured-products",
      "settings": {}
    }
  },
  "order": ["hero", "featured-products"]
}
```

### layout/theme.liquid Skeleton

```liquid
<!doctype html>
<html lang="{{ request.locale.iso_code }}">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{{ page_title }} | {{ shop.name }}</title>
  {{ content_for_header }}
  {{ 'base.css' | asset_url | stylesheet_tag }}
</head>
<body>
  {% section 'header' %}
  <main id="MainContent">
    {{ content_for_layout }}
  </main>
  {% section 'footer' %}
  {{ 'global.js' | asset_url | script_tag }}
</body>
</html>
```

---

## Phase 6: SVG Handling

Small, reusable SVGs (icons, logos, UI elements) should **always** become Liquid snippets, not asset files. This allows color to be controlled dynamically via Liquid variables or schema color settings.

### SVG → Snippet Conversion

```jsx
// React
import CartIcon from './icons/cart.svg';
<CartIcon color={iconColor} width={24} />
```

```liquid
{% comment %} snippets/icon-cart.liquid {% endcomment %}
{% comment %} Accepts: color, size {% endcomment %}
<svg
  xmlns="http://www.w3.org/2000/svg"
  width="{{ size | default: 24 }}"
  height="{{ size | default: 24 }}"
  viewBox="0 0 24 24"
  fill="none"
  stroke="{{ color | default: 'currentColor' }}"
  stroke-width="2"
>
  <!-- paste SVG path data here -->
  <path d="M6 2L3 6v14a2 2 0 002 2h14a2 2 0 002-2V6l-3-4z"/>
</svg>
```

```liquid
{% comment %} Usage in a section {% endcomment %}
{% render 'icon-cart', color: section.settings.icon_color, size: 24 %}
```

**Rules for SVG conversion:**
- All icons → `snippets/icon-[name].liquid` with `color` and `size` parameters
- All brand logos → `snippets/icon-logo.liquid` (controllable from theme editor)
- Large decorative illustrations with no color needs → `assets/[name].svg`
- Never use `<img src="...svg">` for icons — it blocks CSS color control

---

## Phase 7: JavaScript Conversion

React event handlers and UI state become vanilla JS in `assets/*.js`:

```jsx
// React
const [isOpen, setIsOpen] = useState(false);
<button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
<nav className={isOpen ? 'nav--open' : ''}>...</nav>
```

```javascript
// assets/header.js
const toggleButton = document.querySelector('[data-nav-toggle]');
const nav = document.querySelector('[data-nav]');

toggleButton?.addEventListener('click', () => {
  const isOpen = nav.classList.toggle('nav--open');
  toggleButton.setAttribute('aria-expanded', isOpen);
});
```

```liquid
{% comment %} In sections/header.liquid {% endcomment %}
<button data-nav-toggle aria-expanded="false" aria-controls="MainNav">Menu</button>
<nav id="MainNav" data-nav>...</nav>
{{ 'header.js' | asset_url | script_tag }}
```

### Cart Interactions (Ajax API)

Replace React cart state with Shopify's Ajax API:

```javascript
// assets/cart.js — Add to cart
async function addToCart(variantId, quantity = 1) {
  const response = await fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ id: variantId, quantity })
  });
  const item = await response.json();
  // Update cart count in header
  document.querySelector('[data-cart-count]').textContent = 
    (await (await fetch('/cart.js')).json()).item_count;
  return item;
}
```

---

## Phase 8: settings_schema.json

Global theme settings (colors, typography, social links) go here:

```json
[
  {
    "name": "theme_info",
    "theme_name": "My Converted Theme",
    "theme_version": "1.0.0",
    "theme_author": "Your Name",
    "theme_documentation_url": "",
    "theme_support_url": ""
  },
  {
    "name": "Colors",
    "settings": [
      {
        "type": "color",
        "id": "color_primary",
        "label": "Primary Color",
        "default": "#000000"
      },
      {
        "type": "color",
        "id": "color_secondary",
        "label": "Secondary Color",
        "default": "#ffffff"
      }
    ]
  },
  {
    "name": "Typography",
    "settings": [
      {
        "type": "font_picker",
        "id": "font_heading",
        "label": "Heading Font",
        "default": "helvetica_n4"
      },
      {
        "type": "font_picker",
        "id": "font_body",
        "label": "Body Font",
        "default": "helvetica_n4"
      }
    ]
  }
]
```

Access in Liquid: `{{ settings.color_primary }}`, `{{ settings.font_heading | font_face }}`

---

## Flags — What Requires Manual Intervention

After conversion, always produce a `CONVERSION_NOTES.md` listing:

| Issue Type | Examples | Recommendation |
|---|---|---|
| **External API calls** | Weather, CMS, payment providers | Wrap in Shopify App proxy or use Metafields |
| **React-only libraries** | Framer Motion, React Query, SWR | Replace with CSS transitions + vanilla fetch |
| **Authentication flows** | JWT, OAuth, custom auth | Use Shopify Customer Accounts |
| **Server-side logic** | Express routes, API handlers | Shopify Functions or App extensions |
| **Real-time features** | Websockets, live updates | Shopify Channels / App |
| **Complex state** | Redux store, Zustand | Evaluate per-case; often simplifies to Liquid |
| **Dynamic routing** | Catch-all routes, middleware | Shopify handles product/collection routing natively |

---

## Output Checklist

Before delivering converted files, verify:

- [ ] Every `sections/*.liquid` file has a `{% schema %}` block
- [ ] Every `{% schema %}` has at least one preset (strictly required only for dynamically-added sections, but always include one as a safe default — it costs nothing and prevents editor issues)
- [ ] All hardcoded strings have been moved to schema `default` values
- [ ] `{{ block.shopify_attributes }}` added to all block elements (enables editor dragging)
- [ ] `{{ content_for_header }}` present in `layout/theme.liquid` `<head>`
- [ ] `{{ content_for_layout }}` present in `<main>`
- [ ] Images use `| image_url:` + `| image_tag` (not raw `<img src>`)
- [ ] All prices use `| money` filter
- [ ] No hardcoded URLs — use `routes.*` Liquid objects
- [ ] `CONVERSION_NOTES.md` produced listing manual items

---

## Read More

For Shopify Liquid reference details, read `references/liquid-objects.md`.
For schema setting types and advanced blocks, read `references/schema-guide.md`.
