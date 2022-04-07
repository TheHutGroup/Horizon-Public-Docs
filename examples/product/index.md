---
layout: default
title: Product
parent: Examples & Concepts
nav_order: 6
---

# Product

On our platform, product pages are made up of product content, reviews and recommendations. These can all be accessed easily through the product query.

## Relationships

When it comes to products, we have the concept of a Product, and a Product Variant. On other platforms, these are sometimes referred to as Master/Child or Parent/Child.

A Product Variant is something that can actually be purchased and many products only have a single variant. For products that have multiple variants, there are variant options that are grouped by the variant key e.g. flavour or size.

What is shown on a product page (and therefore has a distinct URL) and in search/list results is a Product, not a variant. In our tooling, we do have the concept of Linked SKUs where 2 Products can be linked together so they appear together as variants but also have distinct URLs and will appear distinctly in searches and lists.

Content can be set on either the product or product variant, depending on whether this content should be shared across all variants or update when changing variants on site.

## Content

Product content is any content that can be added to a product that isn't mandatory data like a price, a title, a url and a name. Product content can be in one of these predefined types found [here](https://api.thehut.net/lfint/en/docs#ProductContentValue).

As previously stated, this can be set on the Product or the Product Variant so is best to query this for both objects to choose which to show.

## URLs and Strict flag

When using the product query, a strict flag can be supplied. This flag determines whether the server should handle any redirects based on the product. These could be because you have entered a variant SKU, so we will actually return the master for you. Or if there is a redirect in place for discontinued products for example.

If this is set to true, we will return null if a variant SKU is supplied.

URLs are currently defined in our product stack and are used to service the website as well as performance marketing integrations like Google Shopping, hence why they are provided in the data.

## Images

Currently, images are returned in 3 possible sizes. In future, this will be changed to return a single image URL where the exact size is customisable based on URL parameters.

## PAPs - Promotion Aware Products

As well as a promotion being aware of the products it applies to, on our platform, the inverse is also true. This means that for a product, if a promotion (or many) apply to it, the product is aware of this and promotion related information can be surfaced alongside the product.

This includes information like:

- Offer Title
- Offer Message
- Offer Link - A link to other products in this promotion
- Link Text

These are all customised and set up through our Content Management System that allow this information to be displayed beside a product.

When multiple promotions apply to a single product, the offer that provides the best value to the customer is the one that has the information displayed.

This is seen in the `marketedSpecialOffer` object below.

## Buy Now Pay Later Providers

Some payment providers offer the customer the option to purchase products in installments. We call these providers buy now pay later providers.

This option depends on various factors, such as country, currency and the total price of the product variant and all providers have different qualifying criteria.

To see what buy now pay later providers are avaiable for a product variant, and what installment breakdown they offer, you can use 'buyNowPayLaterProviders' field on the product variant.

## Example

```graphql
query ProductPage {
  product(sku: 10530421, strict: false) {
    title
    marketedSpecialOffer {
      title {
        content {
          type
          content
        }
      }
      description {
        content {
          type
          content
        }
      }
      landingPageLink {
        text
        url
      }
    }
    images {
      thumbnail
      largeProduct
      zoom
    }
    url
    options {
      key
      choices {
        optionKey
        key
        colour
        title
      }
    }
    content {
      key
      value {
        ... on ProductContentStringValue {
          stringValue: value
        }
        ... on ProductContentStringListValue {
          stringListValue: value
        }
        ... on ProductContentIntValue {
          intValue: value
        }
        ... on ProductContentIntListValue {
          intListValue: value
        }
        ... on ProductContentRichContentValue {
          richContentValue: value {
            content {
              type
              content
            }
          }
        }
        ... on ProductContentRichContentListValue {
          richContentListValue: value {
            content {
              type
              content
            }
          }
        }
      }
    }
    reviews {
      total
      averageScore
      maxScore
      count1Score
      count2Score
      count3Score
      count4Score
      count5Score
      reviews {
        reviews {
          id
          title
          authorName
          verifiedPurchase
          posted
          elements {
            ... on RatingReviewElement {
              key
              maxScore
              score
            }
            ... on TextReviewElement {
              key
              value
            }
          }
        }
        total
        hasMore
      }
    }
    fbt: recommendations(type: FREQUENTLY_BOUGHT_TOGETHER) {
      sku
      url
      title
      images {
        thumbnail
      }
      defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
        price(currency: GBP, shippingDestination: GB) {
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
    personalised: recommendations(type: PERSONALISED) {
      sku
      url
      title
      images {
        thumbnail
      }
      defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
        price(currency: GBP, shippingDestination: GB) {
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
    variants {
      sku
      title
      images {
        thumbnail
        largeProduct
        zoom
      }
      choices {
        optionKey
        key
        colour
        title
      }
      buyNowPayLaterProviders(
        settings: { currency: GBP, shippingDestination: GB }
      ) {
        providerName
        displayName
        numberOfInstalments
        instalmentAmount {
          currency
          amount
          displayValue
          scalarValue
        }
        landingPageLink
      }
    }
    platform
    sizeGuide {
      content {
        type
        content
      }
    }
    defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
      sku

      choices {
        optionKey
        key
        colour
        title
      }
      title
      inStock
      availabilityMessage
      price(currency: GBP, shippingDestination: GB) {
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
      content {
        key
        value {
          ... on ProductContentStringValue {
            stringValue: value
          }
          ... on ProductContentStringListValue {
            stringListValue: value
          }
          ... on ProductContentIntValue {
            intValue: value
          }
          ... on ProductContentIntListValue {
            intListValue: value
          }
          ... on ProductContentRichContentValue {
            richContentValue: value {
              content {
                type
                content
              }
            }
          }
          ... on ProductContentRichContentListValue {
            richContentListValue: value {
              content {
                type
                content
              }
            }
          }
        }
      }
    }
    brand {
      name
      page {
        path
      }
      imageUrl
    }
  }
}
```
