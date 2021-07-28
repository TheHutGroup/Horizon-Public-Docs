---
layout: default
title: View Basket
parent: Examples & Concepts
nav_order: 10
---

# View Basket

The basket object is returned whenever any mutations are made to the basket but can also be queried directly for things like a basket page where we just want to display the contents of the basket.

The basket is comprised of:
- id - This is the id of the basket and changes with each change to the basket
- items 
    - The product information
    - The charge price per unit - after offers have applied
    - The total charge price - charge price per unit multiplied by quantity - after offers have applied
    - Quantity
    - The offers applied to that product
        - This includes the total saving to the entire basket by this offer
        - Whether the offer is removable e.g. a discount code offer
        - The offer message to display to the user for the product
    - Whether the item was added automatically as a free gift
- appliedOffers
    - Same as the per product information but lists all offers affecting the basket. On a product level it only shows the offers that affect that product.
- messages
    - These are the messages to display on the basket. These have a type and a message to display. The types and recommendations can be found [here](https://api.thehut.net/lfint/en/docs#BasketMessageType)
    - Note: Some of these messages will appear once and disappear. For example, the `PRODUCT_OUT_OF_STOCK` message will show and detail the item that is out of stock, then the product and message will automatically be removed next time the basket is requested.

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