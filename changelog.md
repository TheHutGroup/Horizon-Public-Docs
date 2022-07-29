---
layout: default
title: Breaking Changes
nav_order: 5
---

# Breaking Changes

This page lists any backwards-incompatible changes made to the Horizon schema. 

## Change in registration input fields (Released on 27/10/2021)

The field "lastName" has been renamed as "surname" to make it compatible with the "surname" field returned from the "accountCreationForm" query.

## Change in supersize variant (Released on 23/03/2022)

The supersize variants query has changed. The previous query has been deprecated and will be removed from the schema in 6 months.
. [See here](examples/features/supersize.md)

## Change in add to basket mutation (Released on 13/05/2022)

There is a new mutation to add products to basket which allows adding multiple products to basket within one operation. The previous mutation has been deprecated and will be removed from the schema in 6 months.
. [See here](examples/basket/add-to-basket.md)

## Change in addProductToBasket Basket mutation (Released on 25/05/2022)

The way Horizon calculates a product quantity inside a Basket has been changed. This means that Horizon now takes into 
account the quantity of a pre-existing product inside a Basket when clients attempt to add additional products using the 
addProductToBasket mutation. Now, given a basket containing a pre-existing product of quantity 1, calling the following 
mutation:

```graphql
mutation AddToBasket {
    addProductToBasket(
        basketId: <BASKET ID>
        sku: <SKU OF PRE-EXISTING PRODUCT>
        quantity: 1
        settings: { currency: GBP, shippingDestination: GB }
    ) {...}
}
```

Will result in the basket now containing 1 product of quantity 2.

## Change in product to cheapestVariantPrice field (Released on 19/07/2022)

The field cheapestVariantPrice is now deprecated and will be removed from the schema in 6 months. Please use the cheapestVariant field to access the cheapest variant price. 