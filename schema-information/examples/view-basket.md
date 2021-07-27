---
layout: default
title: View Basket
parent: GraphQL Examples
grand_parent: Schema Information
nav_order: 9
---

# View Basket
```graphql
query ViewBasket {
  basket(
    id: "2298e807-50ee-4120-abfa-3c0f0ef78fca:1611587680727",
    settings: {
      currency:GBP
      shippingDestination: GB
  	}
  ) {
    id
    items {
      product {
        title
      }
      chargePricePerUnit { # item price after offers have applied
        currency
        amount
        displayValue
      }
      totalChargePrice { # line price after offers have applied
        currency
        amount
        displayValue
      }
      quantity
      appliedOffers {
        totalBasketDiscount {
          currency
          amount
          displayValue
        }
        removeable # if applied through discount code, it can be removed
        message
        info
      }
      freeGift
    }
    appliedOffers {
      totalBasketDiscount {
        currency
        amount
        displayValue
      }
      removeable
      message
      info
    }
    messages {
      type
      message
    }
  }
}
```