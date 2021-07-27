---
layout: default
title: Add To Basket
parent: GraphQL Examples
grand_parent: Schema Information
nav_order: 9
---

# Add to Basket

When adding to basket for the first time, the basket ID should be set as null which will generate a basket for you. This should then be provided back each time. Note: The basket id will update with each manipulation to the basket as it contains a last updated timestamp. The basket ID can also change during a login where a basket merge can occur.

Altering the quantity of the basket is a separate mutation that allows you to set the quantity of an item if you know its already there.

```graphql
mutation AddToBasket {
  addProductToBasket(
    basketId: null
    sku: 10797927
    quantity: 8
    settings: { currency: GBP, shippingDestination: GB }
  ) {
    id
    items {
      product {
        title
      }
      chargePricePerUnit {
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
        removeable
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
    selectYourSample {
      id
      title
      message
      currentAmountSpent {
        currency
        amount
        displayValue
      }
      tiers {
        id
        thresholdAmountSpent {
          currency
          amount
          displayValue
        }
        products {
          sku
          title
          images {
            thumbnail
          }
        }
        selectedProducts {
          sku
        }
        maxSelectedProducts
      }
    }
  }
}
```

## Select Your Sample
Select your sample is a feature available on our promotions system. This allows a site to have a promotion that once the criteria is met, the customer can choose a free gift from a range of options. It is also possible to have multiple tiers with different spend thresholds in to incentivise customers to increase the average order values.



```graphql
selectYourSample {
      id
      title
      message
      currentAmountSpent {
        currency
        amount
        displayValue
      }
      tiers {
        id
        thresholdAmountSpent {
          currency
          amount
          displayValue
        }
        products {
          sku
          title
          images {
            thumbnail
          }
        }
        selectedProducts {
          sku
        }
        maxSelectedProducts
      }
    }
```