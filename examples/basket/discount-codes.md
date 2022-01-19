---
layout: default
title: Discount Codes
parent: Basket
grand_parent: Examples & Concepts
nav_order: 2
---

# Discount Codes

Our platform supports discount codes that can be added and removed from the basket. Some of these are single use and some are multi-use. To add a discount code to the basket, the following mutation can be used.

```graphql
mutation AddCodeToBasket {
  applyCodeToBasket(basketId:"63bf415e-f992-41bc-915d-2cd7badde013:1630492816368", code: "BEST", settings:{
    currency: GBP
    shippingDestination: GB
  }) {
    id
    items {
      product {
        title
      }
      standardPricePerUnit {
        currency
        amount
        displayValue
      }
      chargePricePerUnit {
        currency
        amount
        displayValue
      }
      discountPerUnit {
        currency
        amount
        displayValue
      }
    }
    appliedOffers {
      totalBasketDiscount {
        currency
        amount
        displayValue
      }
      removeable
      message
    }
    messages {
      type
      message
    }
  }
}
```

In the response, you should check the `messages` object and check the `type` and `message` fields. 

If the type is `CODE_APPLIED` this means the code was valid, and the offer applied successfully. Other statuses can be checked for different types of errors.

When a code is added to the basket, the associated offer in the `appliedOffers` object will be marked as `removeable` and can be removed using the `removeCodeFromBasket` mutation.