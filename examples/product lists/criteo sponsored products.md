---
layout: default
title: Criteo Sponsored Products
parent: Product Lists & Search
grand_parent: Examples & Concepts
nav_order: 6
---

# Criteo Sponsored Products
Horizon exposes sponsored products (also known as sponsored ads) within objects of type `HmacAppliedPlacement`, 
which contains the sponsored product placements. 
The Sponsored Products themselves are exposed in objects of type `WidgetProductItem` in the following way:

```
HmacAppliedPlacement > PlacementFormatElement > HmacAppliedProduct > WidgetProductItem
```

`WidgetProductItem` encapsulate two objects - `Product` and `ProductVariant`, one of which may be null at any given time, 
depending on whether sponsored products returned from Criteo are of type `Product` or `ProductVariant`.

Sponsored Products are generally queried as part of:
- Product List Page (PLP) - Product list - In Grid
- Product List Page (PLP) - other widgets - TOP
- Homepage - Bottom or PromoBanner
- Product Pages (PDP) 
- Search - In Grid

Depending on where you query for sponsored ads, different sponsored ads placements will be returned.

## Sponsored ads on PLP - In Grid (product list)
When Sponsored ads are queried for via `productList` on a PLP, the following will apply: 
- the placement here will be `viewCategoryApiDesktop-InGrid`
- the `key` of the `PlacementFormatElement` object will be `sponsored_products`


example query
```graphql
query ProductList {
  page(path: "/health-beauty/face/acne-blemishes") {
    widgets {
    ... on ProductListWidget {
        productList(input: {
          currency: GBP
          shippingDestination: GB
          limit: 30
          offset: 0
          sort: RELEVANCE
          facets: []          
        }) {
          sponsoredAds {
            placementFormatToProducts {
              key
              value {
                widgetProductItem {
                  product {
                    title
                  }
                  productVariant {
                    title
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## Sponsored ads PLP - TOP (widgets)
When Sponsored ads are queried for via widgets on a PLP, the following will apply:

- the placement here will be `viewCategoryApiDesktop-top`
- the `key` of the `PlacementFormatElement` object will be `FS`

```graphql
query Widget {
  page(path: "/health-beauty/make-up/complexion/foundation-makeup") {
    widgets {
      ... on CriteoSponsoredBannerAds {
        SponsoredAdsPlacement {
          placementFormatToProducts {
            key
            value {
              widgetProductItem {
                product {
                  title
                }
                productVariant {
                  title
                }
              }
            }
          }
        }
      }
    }
  }
}
```
---
**a note on widgets**: _To add `sponsoredAds` field to a widget, a field of the type `SponsoredAdsPlacement` must 
be added to the widget template._

---

## Homepage
When sponsored ads are queried for as part of Homepage, the sponsored ads placements will be either or both of the following:
- placement - `viewHomeApiDesktop-bottom`
  - `key` - `sponsored_products`


- placement - `viewHomeApiDesktop-promoBanner`
  - `key` - `FS`

```graphql
query Homepage {
  page(path: "/") {
    widgets {
      ... on CriteoSponsoredBannerAds {
        SponsoredAdsPlacement {
          placementFormatToProducts {
            key
            value {
              widgetProductItem {
                product {
                  title
                }
                productVariant {
                  title
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## Sponsored ads on PDP
- the placement here will be `viewItemApiDesktop-rightSide`
- the `key` of the `PlacementFormatElement` object will be `sponsored_products`

```graphql
query product {
  product (sku: 11352857, strict: false, skipRedirects: true){
    sku
    sponsoredAds {
      placementFormatToProducts {
        key
        value {
          widgetProductItem {
            product {
              title
            }
            productVariant {
              title
            }
          }
        }
      }
    }
  }
}
```

## Sponsored ads on Search 
- the placement here will be `viewSearchResultApiDesktop-InGrid`
- the `key` of the `PlacementFormatElement` object will be `sponsored_products`

example query for search 
```graphql
query Search {
  search(
    options: {
      currency: GBP
      shippingDestination: GB
      limit: 30
      offset: 0
      sort: RELEVANCE
      facets: []
    }
    query: "elixir"
    skipRedirects: true
  ) {
    productList {
      products {
        title
      }
      sponsoredAds {
        placementFormatToProducts {
          key
          value {
            widgetProductItem {
              product {
                title
              }
              productVariant {
                title
              }
            }
          }
        }
      }
    }
  }
}
```