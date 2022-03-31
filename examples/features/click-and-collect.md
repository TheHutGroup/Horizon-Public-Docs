---
layout: default
title: Click & Collect / Omnichannel
parent: Features
grand_parent: Examples & Concepts
nav_order: 1
---

# Click & Collect / Omnichannel
Click and collect is a feature that can be enabled for Omnichannel based sites that have physical stores. This adds the ability to choose a fulfilment type, select a store as well as get stock information on a per store basis.

## Stores

Stores can be queried in 1 of 3 ways. All Stores, By ID, or Nearest Stores. For the nearest stores query, first we need to get the location the customer cares about, then this can be used to query for the nearest stores. Stores are also aware of other nearby stores.

```graphql
query AllStores {
  stores {
    id
    displayName
    urlTag
    phoneNumber
    openingTimes {
      day
      openingTime
      closingTime
    }
    address {
      addresseeName
      companyName
      addressLine2
      addressLine3
      addressLine4
      addressLine5
      state
      postalCode
      country
      phoneNumber
      clickAndCollect
    }
  }
}

query StoreById {
  store(id: 88) {
    id
    displayName
    urlTag
    phoneNumber
    openingTimes {
      day
      openingTime
      closingTime
    }
    address {
      addresseeName
      companyName
      addressLine2
      addressLine3
      addressLine4
      addressLine5
      state
      postalCode
      country
      phoneNumber
      clickAndCollect
    }
    nearbyStores {
      id
      displayName
      urlTag
      phoneNumber
      openingTimes {
        day
        openingTime
        closingTime
      }
      address {
        addresseeName
        companyName
        addressLine2
        addressLine3
        addressLine4
        addressLine5
        state
        postalCode
        country
        phoneNumber
        clickAndCollect
      }
    }
  }
}

query NearestStores {
  searchLocations(query: "Salford Quays") {
    displayName
    longitude
    latitude
    postcode
  }
  nearbyStores(longitude: -2.28995, latitude: 53.4767) {
    id
    displayName
    urlTag
    phoneNumber
    openingTimes {
      day
      openingTime
      closingTime
    }
    address {
      addresseeName
      companyName
      addressLine2
      addressLine3
      addressLine4
      addressLine5
      state
      postalCode
      country
      phoneNumber
      clickAndCollect
    }
  }
}
```

### Querying a store for a product
When you want to find out if a product is available in your local store, this is very similar to the nearest stores query. The response is slightly different as you now also get the distance and stock quantity of that store. Against the product variant, we also expose the fulfilment methods that are supported for a given product. More information about these can be found in the schema.

```graphql
query ClickAndCollect {
  product(sku: 12826915, strict:false) {
    variants {
      title
      eligibleForFulfilmentMethods
      clickAndCollectStores(longitude: -2.28995, latitude: 53.4767) {
        store {
          displayName
        }
        distanceMiles
        stock
      }
    }
  }
}
```

## Adding a product to basket
For sites that support omnichannel, so have more than 1 type of fulfilment method (other than the usual home delivery of purely online sites), when adding a product to basket, more information is now required.

A fulfilment method, and optional store id are now required (depending on if the fulfilment method utilises a store).

```graphql
mutation AddToBasket {
  addProductToBasket(
    basketId: null
    sku: 10797927
    quantity: 8
    fulfilmentInput: { # new field in the input
      method: COLLECT_IN_STORE
      storeId: 88
    }
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

## Basket
As we now have more information about the product when adding it to basket, this data is also exposed against the basket item as shown below. Store and store stock will only be exposed if the fulfilment type involves a store.

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
      fulfilmentMethod # new field
      store { # new field
        id
        displayName
      }
      storeStock # new field
    }
    messages {
      type
      message
    }
    eligibleForCheckout # new field
  }
}
```

Another change to the basket to support the mixed fulfilment methods is around errors and messaging. We now expose new error types as well as show item level messaging for line level issues. These errors can also block the ability for a customer to checkout and require the customer to fix these issues if they exist.

The basket level messages now support 2 new types: 
- MIXED_FULFILMENT_METHODS - this signifies the site does not support warehouse and store based fulfilment in the same order so must be corrected
- ITEMS_HAVE_ERRORS - this signifies that that one of the individual items has an error that must be fixed

For the new basket item messages under basket.items.messages, these have a message and a type. The 3 supported types and descriptions can be found below and in the schema.

### NOT_ENOUGH_STOCK 
This flag will appear if there is not enough stock for the product and fulfilment option the
customer has chosen. These issues can be fixed by changing the requested stock.

If there is absolutely no stock, FULFILMENT_ISSUE is returned instead.

### PRODUCT_ISSUE
This flag will appear if the product is no longer eligible for the selected fulfilment option,
e.g. this product may no longer be available for home delivery, or click and collect or the
product is marked as not purchasable online anymore. These problems can be fixed by changing
the fulfilment type - note the product may no longer be available for any fulfilment type.

### FULFILMENT_ISSUE
This flag will appear if the selected fulfilment type is no longer available. This can be where
the store is closed or does not have click and collect available and is related the the
fulfilment type itself rather than the product. These problems can be fixed by changing
something about the fulfilment type e.g. changing store.