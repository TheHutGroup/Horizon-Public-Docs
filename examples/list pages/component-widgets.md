---
layout: default
title: Component Widgets
parent: Landing Pages
grand_parent: Examples & Concepts
nav_order: 1
---

# Component Widgets

Widgets are usually associated with pages on site, but there is sometimes the need to display widgets across other components such as popups and core e-commerce objects like the basket.

These special sets of widgets can be retrieved through the `componentWidgets` query and the supported components can be found using the `supportedComponentWidgetNames` query.

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

As you can see from the above, these may be widgets that are 'global' and can be used to power pop-ups across many pages of the site, when the basket page is empty, as a replacement to standard 404 pages or to enhance the search results page.

As with list pages and their widgets, these are set up, configured and assigned through THG's tooling.