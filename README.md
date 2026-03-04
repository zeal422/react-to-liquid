# React to Liquid — LLM Skill

Converts React applications into **Shopify Online Store 2.0** theme code.

## What It Does

This skill helps you migrate React sites and components to Shopify themes. It handles:

- **Full page conversions** — React pages become Shopify sections with templates
- **Component conversions** — Reusable React components become Liquid snippets
- **Schema generation** — Creates merchant-editable settings (text, images, colors, products, collections, etc.)
- **State management** — Converts React state to vanilla JavaScript + Shopify Ajax API
- **SVG handling** — Icon SVGs become dynamic snippets with color control
- **Data mapping** — Replaces API calls with Liquid objects

## Usage

Trigger this skill when working with:
- "Convert this to Shopify"
- "Port this React code to Liquid"
- "Make this a Shopify theme"
- Any React → Shopify conversion task

## Key Concepts

| React | Shopify |
|-------|---------|
| Page component | `templates/*.json` + `sections/*.liquid` |
| Reusable component | `snippets/*.liquid` |
| `useState` / `useEffect` | JavaScript in `assets/*.js` + Ajax API |
| Props | `{% render 'snippet', var: value %} |
| Hardcoded data | Schema `default` values |

## Output

Produces production-ready theme files:
- `sections/` — Theme sections with schema
- `snippets/` — Reusable Liquid partials
- `templates/` — Page templates
- `assets/` — CSS, JS, images

Each section includes a `{% schema %}` block, enabling merchants to edit content via the Shopify Theme Editor without touching code.

## References

- `references/liquid-objects.md` — Quick reference for Liquid objects
- `references/schema-guide.md` — Complete schema settings guide
