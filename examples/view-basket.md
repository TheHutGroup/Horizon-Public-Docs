---
layout: default
title: View Basket
parent: Examples & Concepts
nav_order: 10
---

# View Basket

The basket object is returned whenever any mutations are made to the basket but can also be queried directly for things like a basket page where we just want to display the contents of the basket.

The basket comprises:
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
    - These are the messages to display on the basket. All will have a type, some will also have a server-defined message (e.g. a custom consolation message for an expired discount code) while others have only a type. For those types the client should show a generic message of its own. All types, and recommendations for handling them are listed in the [BasketMessageType docs](https://api.thehut.net/lfint/en/docs#BasketMessageType).
    - Note: Some of these messages will appear once and disappear. For example, the `PRODUCT_OUT_OF_STOCK` message will show and detail the item that is out of stock, then the product and message will automatically be removed next time the basket is requested.
- merged - We persist basket information for logged in customers so that when they return, their basket is still there, regardless of device. This does mean that someone can come to the site and add items to their basket, then when attempting to checkout and they login, their saved basket is also available. In this case and a few other edge cases, this causes a basket merge. 


```graphql
query ViewBasket {
  basket(
    id: "2298e807-50ee-4120-abfa-3c0f0ef78fca:1611587680727",
    settings: {
      currency:GBP
      shippingDestination: GB
  	}
    acknowledgeMerge: true
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
    merged
  }
}
```

## Basket Merge

As stated above, we persist basket information for logged in customers so that when they return, their basket is still there, regardless of device. 
This does mean that someone can come to the site, add items to their basket, then when attempting to checkout and they login, their saved basket is also available. In this case and a few other edge cases, this causes a basket merge. 

When baskets merge, we prevent checkout from being available so that users do not go from the Basket Page to Checkout and suddenly extra items appear in their basket. To resolve this, on the basket query, an `acknowledgeMerge` flag can be passed that will clear this and allow the customer to continue. 
It is recommended that if the `CheckoutStartError` is `BASKETS_MERGED`, the customer is returned to the basket page with a call to the API acknowledging the merge and a message shown to the customer describing why they have been returned. 
