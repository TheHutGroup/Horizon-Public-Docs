---
layout: default
title: Landing Pages
parent: GraphQL Examples
grand_parent: Schema Information
nav_order: 5
---

# Landing Page
Dynamic pages on site fall into 3 categories. Product pages, Search pages and Listing Pages. Listing pages come in 2 varieties, Landing pages and Product List pages. Both are just pages that display a list of widgets.

Listing pages are built using widgets which take data and display them based on how the Frontend widget is defined. This could be a simple image where the widget is just the image URL and a link or could be more complex like a carousel where there is a carosel widget which has an array of widgets as one of its values, each with images and links.

For carosels where widgets may have child widgets, so that the query doesnt need to become recursive, we flatten the child widgets which can then be matched based on the IDs.

```graphql
widgets {
  ... on MyParentWidgetType {
    banners: {
      id
    }
  }
}
flattenedChildWidgets {
  id
  ... on MyChildWidgetType {
    greeting
  }
}
```

New widgets are named and defined in the THG Tooling then become available to use in both the API and for assigning to pages.

Landing pages are usually a list of different widgets like the homepage of a website, Product list pages are usually 1 ProductListWidget on the page.

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
