---
layout: default
title: Product List
parent: Examples & Concepts
nav_order: 6
---

# Product List

Product lists are very similar to search in the sense they display lists of products and can be filtered with facets but are normally pre-built pages. The only difference to a landing page is that they have the ProductListWidget which takes similar parameters to the search query.

```graphql
query ProductList {
  page(path: "/clothing/sale/mens-clothing-sale") {
    title
    metaDescription
    metaSearchKeywords
    widgets {
      ... on ProductListWidget {
        id
        title
        descriptionHtml {
          content {
            type
            content
          }
        }
        productList(input: {
          currency: GBP
          shippingDestination: GB
          limit: 30
          offset: 0
          sort: RELEVANCE
          facets: []          
        }) {
          total
          hasMore
          facets {
            ... on RangedFacet {
              facetName
              facetHeader
              options {
                displayName
                from
                to
                matchedProductCount
              }
            }
            ... on SimpleFacet {
              facetName
              facetHeader
              options {
                optionName
                displayName
                matchedProductCount
              }
            }
            ... on SliderFacet {
              facetName
              facetHeader
              minValue
              maxValue
            }
          }
          products {
            url
            title
            images {
              thumbnail
              largeProduct
              zoom
            }
            reviews {
              total
              averageScore
            }
            defaultVariant(options: {
              currency:GBP,
              shippingDestination: GB,
            }) {
              title
              price(currency: GBP, 
                shippingDestination: GB) {
                price {
                  currency
                  amount
                  displayValue
                }
                rrp {
                  currency
                  amount
                  displayValue
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