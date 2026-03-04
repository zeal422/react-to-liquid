# Shopify Schema Settings — Complete Guide

Full reference for `{% schema %}` blocks in OS 2.0 sections.

---

## All Setting Types

### Text & Content

```json
{ "type": "text", "id": "heading", "label": "Heading", "default": "Hello", "placeholder": "Enter heading" }
{ "type": "textarea", "id": "body", "label": "Body Text", "default": "Description here" }
{ "type": "richtext", "id": "content", "label": "Content", "default": "<p>Hello world</p>" }
{ "type": "html", "id": "custom_html", "label": "Custom HTML" }
{ "type": "liquid", "id": "custom_liquid", "label": "Liquid Code" }
```

### Numbers & Ranges

```json
{ "type": "number", "id": "count", "label": "Item Count", "default": 4 }
{
  "type": "range",
  "id": "opacity",
  "label": "Opacity",
  "min": 0,
  "max": 100,
  "step": 5,
  "unit": "%",
  "default": 80
}
```

### Toggles & Selects

```json
{ "type": "checkbox", "id": "show_button", "label": "Show Button", "default": true }
{
  "type": "select",
  "id": "layout",
  "label": "Layout",
  "options": [
    { "value": "grid", "label": "Grid" },
    { "value": "list", "label": "List" },
    { "value": "carousel", "label": "Carousel" }
  ],
  "default": "grid"
}
{
  "type": "radio",
  "id": "alignment",
  "label": "Alignment",
  "options": [
    { "value": "left", "label": "Left" },
    { "value": "center", "label": "Center" }
  ],
  "default": "center"
}
```

### Media

```json
{ "type": "image_picker", "id": "image", "label": "Image" }
{ "type": "video", "id": "video", "label": "Video" }
{ "type": "video_url", "id": "video_url", "label": "Video URL", "accept": ["youtube", "vimeo"] }
```

### Colors & Fonts

```json
{ "type": "color", "id": "bg_color", "label": "Background Color", "default": "#ffffff" }
{ "type": "color_background", "id": "gradient", "label": "Background Gradient" }
{ "type": "color_scheme", "id": "color_scheme", "label": "Color Scheme", "default": "scheme-1" }
{ "type": "color_scheme_group", "id": "color_scheme_group", "label": "Color Scheme Group" }
{ "type": "font_picker", "id": "font", "label": "Font", "default": "helvetica_n4" }
```

### Links & Shopify Resources

```json
{ "type": "url", "id": "link", "label": "Link URL" }
{ "type": "product", "id": "featured_product", "label": "Featured Product" }
{ "type": "product_list", "id": "products", "label": "Products", "limit": 12 }
{ "type": "collection", "id": "collection", "label": "Collection" }
{ "type": "collection_list", "id": "collections", "label": "Collections", "limit": 6 }
{ "type": "blog", "id": "blog", "label": "Blog" }
{ "type": "page", "id": "page", "label": "Page" }
{ "type": "link_list", "id": "menu", "label": "Menu" }
{ "type": "metaobject", "id": "metaobject", "label": "Metaobject", "metaobject_type": "custom_type" }
{ "type": "metaobject_list", "id": "metaobjects", "label": "Metaobjects", "metaobject_type": "custom_type" }
```

### Layout / Visual Separators

```json
{ "type": "header", "content": "Section Title — Groups settings visually" }
{ "type": "paragraph", "content": "Helpful note for merchants shown in the editor." }
```

---

## Accessing Settings in Liquid

```liquid
{# Section settings #}
{{ section.settings.heading }}
{{ section.settings.show_button }}
{{ section.settings.bg_color }}
{{ section.settings.featured_product.title }}
{{ section.settings.image | image_url: width: 800 | image_tag }}

{# Block settings (inside {% for block in section.blocks %}) #}
{{ block.settings.title }}
{{ block.settings.link }}
{{ block.id }}
{{ block.type }}
{{ block.shopify_attributes }}   {# REQUIRED for theme editor drag-to-reorder #}
```

---

## Blocks — Full Example

```json
{
  "name": "Feature Cards",
  "blocks": [
    {
      "type": "card",
      "name": "Card",
      "limit": 1,
      "settings": [
        { "type": "image_picker", "id": "icon", "label": "Icon" },
        { "type": "text", "id": "title", "label": "Title", "default": "Feature" },
        { "type": "textarea", "id": "description", "label": "Description" },
        { "type": "url", "id": "link", "label": "Link" },
        {
          "type": "select",
          "id": "style",
          "label": "Card Style",
          "options": [
            { "value": "default", "label": "Default" },
            { "value": "outlined", "label": "Outlined" },
            { "value": "filled", "label": "Filled" }
          ],
          "default": "default"
        }
      ]
    }
  ],
  "max_blocks": 6,
  "presets": [
    {
      "name": "Feature Cards",
      "blocks": [
        { "type": "card" },
        { "type": "card" },
        { "type": "card" }
      ]
    }
  ]
}
```

---

## Presets

Presets define how a section appears when added from the "Add section" menu. Always include at least one:

```json
"presets": [
  {
    "name": "Hero Banner",
    "settings": {
      "title": "Welcome to our store",
      "cta_text": "Shop now"
    },
    "blocks": [
      { "type": "slide", "settings": { "heading": "First Slide" } }
    ]
  }
]
```

---

## Disabled On

Restrict a section to specific templates:

```json
{
  "name": "Product Reviews",
  "enabled_on": {
    "templates": ["product"],
    "groups": ["header", "footer", "custom.product"]
  }
}
```

---

## Color Schemes (Dawn-style)

For Dawn-convention themes, use color_scheme settings to support light/dark variants:

```json
{
  "type": "color_scheme",
  "id": "color_scheme",
  "label": "Color scheme",
  "default": "scheme-1"
}
```

```liquid
<div class="{{ section.settings.color_scheme }}">
  ...
</div>
```

In `config/settings_schema.json`, define the schemes:

```json
{
  "name": "Color schemes",
  "settings": [
    {
      "type": "color_scheme_group",
      "id": "color_schemes",
      "definition": [
        { "type": "color", "id": "background", "label": "Background", "default": "#FFFFFF" },
        { "type": "color", "id": "text", "label": "Text", "default": "#121212" },
        { "type": "color", "id": "button", "label": "Button", "default": "#121212" },
        { "type": "color", "id": "button_label", "label": "Button label", "default": "#FFFFFF" }
      ],
      "role": {
        "background": "background",
        "text": "text",
        "primary_button": "button",
        "on_primary_button": "button_label"
      }
    }
  ]
}
```
