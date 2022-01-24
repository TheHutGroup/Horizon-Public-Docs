---
layout: default
title: Component Widgets
parent: Landing Pages
grand_parent: Examples & Concepts
nav_order: 1
---

# Component Widgets

Widgets are usually associated with pages on site, but there is sometimes the need to display widgets across other components such as a popup, or alongside core e-commerce objects like the basket.

These "special" sets of widgets can be queried through the componentWidgets query and the supported components can be found using the supportedComponentWidgetNames query.

```graphql
query SupportedComponentWidgetNames {
  supportedComponentWidgetNames
}
```

```json
{
  "data": {
      "supportedComponentWidgetNames": [
        "no-results",
        "basket",
        "search",
        "page-unavailable",
        "wishlist",
        "global",
        "product-page",
        "account"
      ]
  }
}
```

As you can see from the above, these may be widgets that are 'global' and could be used to power a pop-up across many pages of the site, used for when the basket page is empty, as a replacement to standard 404 pages, or to enhance the search results page.

As with list pages and their widgets, these are set up, configured and assigned through THGs tooling.