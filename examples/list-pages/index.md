---
layout: default
title: Landing Pages
parent: Examples & Concepts
has_children: true
nav_order: 4
---

# Landing Page
Dynamic pages on site fall into 3 categories: Product pages, Search pages and Listing Pages. Listing pages come in 2 varieties: Landing pages and Product List pages. Both are just pages that display a list of widgets.

Listing pages are built using widgets which take data and display them based on how the Frontend widget is defined. This could be a simple image, where the widget is just the image URL and a link, or a more complex widget like a carousel.

A carousel widget has an array of widgets as one of its values, each with images and links. We flatten these child widgets so the queries don't become recursive. They can then be matched based on the widget IDs.

```graphql
query PageWidgets {
  page(path: "/") {
    widgets {
      ... on MyParentWidgetType {
        banners {
          id
        }
      }
    }
    flattenedChildWidgets {
      id
      ... on MyChildWidgetType {
        title
      }
    }
  }
}
```

Here `MyParentWidgetType` and `MyChildWidgetType` would be replaced with specific widget types for your site. For example `GlobalTabbedWidgetSet` and `GlobalWaitListSignUpWidget`.

New widgets are named and defined in the THG Tooling then become available to use in both the API and for assigning to pages.

Landing pages are usually a list of different widgets like the homepage of a website. Product list pages are usually a page with a single ProductListWidget.

Widgets can be defined on a per site basis depending on what the client supports. For THG built websites, we will support our global list of widgets on all of them.

```graphql
query LandingPage {
  page(path: "/") {
    title
    metaDescription
    metaSearchKeywords
    widgets {
      ... on GlobalPrimaryBanner {
        id
        imageSmall
        imageMedium
        imageLarge
        bannerURL
      }
      ... on GlobalBrandLogos {
        id
        itemOneURL
        itemOneImage
        itemTwoURL
        itemTwoImage
        itemThreeURL
        itemThreeImage
        itemFourURL
        itemFourImage
        itemFiveURL
        itemFiveImage
        itemSixURL
        itemSixImage
      }
      ... on GlobalSectionPeek {
        id
        title
        numberOfProducts
        url
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
      ... on GlobalThreeItemEditorial {
        id
        widgetTitle
        itemOneUrl
        itemOneImage
        itemOneTitle
        itemTwoUrl
        itemTwoImage
        itemTwoTitle
        itemThreeUrl
        itemThreeImage
        itemThreeTitle
      }
      ... on GlobalGeneralImageBanner {
        id
        smallImage
        mediumImage
        largeImage
        linkUrl
      }
      ... on GlobalTwoItemEditorial {
        itemOneURL
        itemOneTitle
        itemOneDescription
        itemOneImage
        itemOneCTAText
        itemTwoURL
        itemTwoTitle
        itemTwoDescription
        itemTwoImage
        itemTwoCTAText
      }
    }
  }
}
```
