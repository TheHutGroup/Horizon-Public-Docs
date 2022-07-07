---
layout: default
title: Add To Basket
parent: Basket
grand_parent: Examples & Concepts
nav_order: 1
---

# Add to Basket

When adding to basket for the first time, the basket ID should be set as null which will generate a basket for you. This should then be provided back each time.

Note: The basket ID will update with each manipulation to the basket as it contains a last-updated timestamp. The basket ID may also change on the first request after a login, as a basket merge can occur.

Altering the quantity of the basket is a separate mutation that allows you to set the quantity of an item if you know its already there. However, calling the addProductToBasket mutation using a basketId 
containing a pre-existing product will result in the quantity of the pre-existing product to also be updated, where the new quantity of the pre-existing product will be the sum of the 
old quantity of the product, plus the quantity of the product you wish to add.

```graphql
mutation AddToBasket {
  addProductsToBasket(
    basketId: null
    items: [
          { sku: 12815375, quantity: 2 }
          { sku: 12823433, quantity: 3 }
    ]
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


Note: Adding products to basket was previously done with the below mutation but has now been deprecated. The deprecated mutation will be removed from the schema 6 months after the release of the addProductsToBasket mutation. (Release date: 13/05/2022)

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
    messages {
      type
      message
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