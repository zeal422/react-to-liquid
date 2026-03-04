# Shopify Liquid Objects Reference

Quick reference for the most commonly needed Liquid objects when converting React data fetching.

---

## Product Object

```liquid
{{ product.id }}
{{ product.title }}
{{ product.handle }}               {# URL slug #}
{{ product.description }}
{{ product.price }}                {# In cents — always pipe through | money #}
{{ product.price | money }}        {# "$29.99" #}
{{ product.compare_at_price }}
{{ product.available }}            {# Boolean #}
{{ product.vendor }}
{{ product.type }}
{{ product.tags }}                 {# Array #}
{{ product.url }}

{# Images #}
{{ product.featured_image | image_url: width: 800 | image_tag }}
{% for image in product.images %}
  {{ image | image_url: width: 600 | image_tag: alt: image.alt }}
{% endfor %}

{# Variants #}
{% for variant in product.variants %}
  {{ variant.id }}
  {{ variant.title }}
  {{ variant.price | money }}
  {{ variant.available }}
  {{ variant.sku }}
  {{ variant.inventory_quantity }}
  {% for option in variant.options %}{{ option }}{% endfor %}
{% endfor %}

{# Options (Size, Color etc.) #}
{% for option in product.options_with_values %}
  {{ option.name }}    {# "Size" #}
  {% for value in option.values %}{{ value }}{% endfor %}
{% endfor %}
```

---

## Collection Object

```liquid
{{ collection.id }}
{{ collection.title }}
{{ collection.handle }}
{{ collection.description }}
{{ collection.url }}
{{ collection.image | image_url: width: 800 | image_tag }}
{{ collection.products_count }}

{% for product in collection.products %}
  {% render 'product-card', product: product %}
{% endfor %}

{# Pagination #}
{% paginate collection.products by 24 %}
  {% for product in collection.products %}...{% endfor %}
  {{ paginate | default_pagination }}
{% endpaginate %}
```

---

## Cart Object

```liquid
{{ cart.item_count }}
{{ cart.total_price | money }}
{{ cart.total_weight }}

{% for item in cart.items %}
  {{ item.product.title }}
  {{ item.variant.title }}
  {{ item.quantity }}
  {{ item.price | money }}
  {{ item.line_price | money }}
  {{ item.image | image_url: width: 200 | image_tag }}
  {{ item.url }}
  {{ item.key }}              {# Used for Ajax API remove/update #}
{% endfor %}
```

---

## Customer Object

```liquid
{% if customer %}
  {{ customer.id }}
  {{ customer.first_name }}
  {{ customer.last_name }}
  {{ customer.email }}
  {{ customer.phone }}
  {{ customer.orders_count }}
  {{ customer.total_spent | money }}
  {{ customer.tags }}

  {% for order in customer.orders %}
    {{ order.name }}           {# Order #1001 #}
    {{ order.created_at | date: "%B %d, %Y" }}
    {{ order.financial_status }}
    {{ order.fulfillment_status }}
    {{ order.total_price | money }}
  {% endfor %}

  {% for address in customer.addresses %}
    {{ address.first_name }} {{ address.last_name }}
    {{ address.address1 }}
    {{ address.city }}, {{ address.province }} {{ address.zip }}
    {{ address.country }}
  {% endfor %}
{% endif %}
```

---

## Shop Object

```liquid
{{ shop.name }}
{{ shop.description }}
{{ shop.email }}
{{ shop.phone }}
{{ shop.url }}
{{ shop.currency }}
{{ shop.money_format }}         {# "${{amount}}" #}
{{ shop.locale }}
{{ shop.metafields.namespace.key }}
```

---

## Routes Object (use instead of hardcoded URLs)

```liquid
{{ routes.root_url }}                        {# / #}
{{ routes.account_url }}                     {# /account #}
{{ routes.account_login_url }}               {# /account/login #}
{{ routes.account_logout_url }}              {# /account/logout #}
{{ routes.account_register_url }}            {# /account/register #}
{{ routes.cart_url }}                        {# /cart #}
{{ routes.cart_add_url }}                    {# /cart/add #}
{{ routes.cart_change_url }}                 {# /cart/change #}
{{ routes.search_url }}                      {# /search #}
{{ routes.collections_url }}                 {# /collections #}
{{ routes.all_products_collection_url }}     {# /collections/all #}
```

---

## Linklists (Navigation Menus)

```liquid
{% for link in linklists['main-menu'].links %}
  <a href="{{ link.url }}" {% if link.active %}aria-current="page"{% endif %}>
    {{ link.title }}
  </a>
  {% if link.links.size > 0 %}
    {% for child in link.links %}
      <a href="{{ child.url }}">{{ child.title }}</a>
    {% endfor %}
  {% endif %}
{% endfor %}
```

---

## Filters Reference

```liquid
{# Prices #}
{{ product.price | money }}                    {# $29.99 #}
{{ product.price | money_without_currency }}   {# 29.99 #}
{{ product.price | money_with_currency }}      {# $29.99 USD #}

{# Images #}
{{ image | image_url: width: 800 }}
{{ image | image_url: width: 800, height: 600, crop: 'center' }}
{{ image | image_url: width: 800 | image_tag: loading: 'lazy', alt: 'Alt text' }}

{# Strings #}
{{ string | upcase }}
{{ string | downcase }}
{{ string | capitalize }}
{{ string | strip_html }}
{{ string | truncate: 100 }}
{{ string | replace: 'old', 'new' }}
{{ string | split: ',' }}

{# Dates #}
{{ article.created_at | date: "%B %d, %Y" }}
{{ 'now' | date: "%Y" }}

{# URLs #}
{{ 'base.css' | asset_url }}
{{ 'global.js' | asset_url | script_tag }}
{{ 'base.css' | asset_url | stylesheet_tag }}
{{ product | link_to }}
{{ product.url | within: collection }}

{# Arrays #}
{{ array | join: ', ' }}
{{ array | first }}
{{ array | last }}
{{ array | size }}
{{ array | sort: 'price' }}
{{ array | where: 'available', true }}
{{ array | map: 'title' }}
{{ array | uniq }}
```

---

## Metafields

```liquid
{# Access product metafields #}
{{ product.metafields.custom.my_field }}
{{ product.metafields.reviews.rating }}

{# Render metafield types #}
{{ product.metafields.custom.description | metafield_tag }}

{# In schema — link a setting to metafields #}
{
  "type": "text",
  "id": "custom_label",
  "label": "Custom Label"
}
```

---

## Pagination

```liquid
{% paginate collection.products by 24 %}
  <div class="product-grid">
    {% for product in collection.products %}
      {% render 'product-card', product: product %}
    {% endfor %}
  </div>

  {# Built-in pagination links #}
  {{ paginate | default_pagination }}

  {# Or custom: #}
  {% if paginate.previous %}
    <a href="{{ paginate.previous.url }}">← Previous</a>
  {% endif %}
  <span>Page {{ paginate.current_page }} of {{ paginate.pages }}</span>
  {% if paginate.next %}
    <a href="{{ paginate.next.url }}">Next →</a>
  {% endif %}
{% endpaginate %}
```

---

## Section Rendering API (Dynamic Section Updates)

For React-like partial page updates without full reload:

```javascript
// assets/cart-drawer.js
async function refreshCartSection() {
  const response = await fetch(`/?sections=cart-drawer`);
  const data = await response.json();
  document.querySelector('#CartDrawer').innerHTML = data['cart-drawer'];
}
```

The section file (`sections/cart-drawer.liquid`) renders fresh Liquid on each request.
