---
layout: default
title: Supersize
parent: Features
grand_parent: Examples & Concepts
nav_order: 3
---

# Super Size
It is recommended that you read how product relationships work on THG's platform before reading this document. [See here](../product/index.md#relationships)

Each product variant that is a variant in the "Amount" dimension is aware of the related product that shares all the same `VariationOption` values but is the next size up for the Amount VariationOption.
The variation type configured in THGs systems. This type also powers how variation options are sorted.

An example of this is Impact Whey Protein on Myprotein where if we have 2 `VariationOption`s, Amount and Flavour, where the different amounts we have are 500g, 1kg and 2.5kg and the flavours are, Chocolate and Vanilla. 
In this example, the Chocolate 500g variant is aware than the next size up from this is the Chocolate 1kg variant. The logic also takes into account stock levels and will recommend the 2.5kg variant if the 1kg variant is out of stock.

This data can be accessed by querying the following, where the SKU returned is the next size up from 13442764 in this example:

```graphql
query SuperSizeVariantAndSaving {
    productVariant(sku:13442764) {
        supersize (settings: { currency: GBP, shippingDestination: GB }) {
              variant {
                sku
              }
              saving {
                savingUnit {
                  value
                  unit
                }
                nextSupersizeOption {
                  value
                  unit
                }
                savingPerUnit {
                  currency
                  amount
                  displayValue
                  scalarValue
                }
              }
        }
    }
}
```

Note: supersize variant was used to be queried as below but it's been deprecated now. Below way of querying will be removed from the schema in 6 months.

```graphql
query SuperSizeVariant {
    productVariant(sku:13442764) {
        supersizeVariant {
            sku
        }
    }
}
```

The above query can be used to provide more data on the basket page and upsell to customers to try and get them to upgrade.

If they choose to update, this can be simplified by using the following mutation. This mutation will handle the removal of the current variant, and replace it with the supersized variant and keep the quantity the same.

```graphql
mutation SuperSizeProduct {
    supersizeProductInBasket(
        basketId: "c81844c4-d433-4b1d-8804-15a9c45485fb:1642774945809"
        itemId: "10530744"
        settings: { currency: GBP, shippingDestination: GB }
    ) {
        items {
            id
            product {
                sku
            }
            quantity
        }
    }
}
```