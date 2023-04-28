---
layout: default
title: Search
parent: Product Lists & Search
grand_parent: Examples & Concepts
nav_order: 2
---

# Search
Our search system has 2 variants: Normal and Instant.
Instant should be used when the user types into the search bar at the top of the page before they press enter. This search is slightly different to normal as the search performed is more fuzzy in terms of the data returned and also provides spelling corrections and suggested full search queries.

## Facets
Facets are used to filter down lists of products. The possible values are returned as part of a search/product list query and are then supplied back to filter the list down.

## Normal

```graphql
query Search {
  search(options: {
    currency: GBP
    shippingDestination: GB
    limit: 30
    offset: 0
    sort: RELEVANCE
    facets: []
  }, query: "Protain" # purposefully spelt wrong to highlight corrections
  ) {
    total
    hasMore
    correctedQuery
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
```

## Normal Search Using Barcode
You are also able to search for listed products by their barcode, simply state `barcode` as an option and pass your required barcode value as a String to the option field.

```graphql
query Search {
    search(options: {
        currency: GBP
        shippingDestination: GB
        facets: []
        barcode: "<your required barcode value>"
    }, query: ""
    ) {
        products {
            title
        }
    }
}
```

## Instant

```graphql
query InstantSearch {
  instantSearch(
    currency: GBP
    shippingDestination: GB
    limit: 5
    query: "protei" # purposefully spelt wrong to highlight corrections
  ) {
    corrections {
      correction
      highlightedSearchCorrection
    }
    suggestedSearchQueries
    products {
      url
      title
      images {
        thumbnail
        largeProduct
        zoom
      }
      reviews {
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
```