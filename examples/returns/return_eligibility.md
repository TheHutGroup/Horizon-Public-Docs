---
layout: default
title: Return Eligibility
parent: Returns
grand_parent: Examples & Concepts
nav_order: 1
---
# Return Eligibility
When the customer access the return's page, we check if the order is eligible for a return.

We query this via a `isReturnable` field, which is a part of the `Order` object:

```graphql
extend type Order {
    """If no input provided, all skus in the product will be used"""
    isReturnable(input: ReturnsEligibilityInput): ReturnsEligibilityResult @authenticated @if(feature: ORDER_RETURNS)
}
```

For the input, we pass in a list of product SKUs which we would like to return against this order:

```graphql
input ReturnsEligibilityInput @if(feature: ORDER_RETURNS) {
    skus: [SKU!]!
}
```

Horizon will then send a request off to Returns Manager, which will do several checks against the order and the provided product SKUs to determine if the order is eligible for a return. 
If the order is not eligible for a return, the reason for this will be included in the results object:

```graphql
type ReturnsEligibilityResult @if(feature: ORDER_RETURNS) {
    success: Boolean!
    orderError: ReturnsEligibilityOrderError
    productErrors: [ReturnsEligibilitySkuAndProductError!]
    returnOrder: ReturnOrder
    returnReasons: [ReturnReason!]
}
```

- `success`: has the order passed the eligibility check given the provided SKUs.
- `orderError`: if the order has failed, this object will contain the reason why.
- `productErrors`: if a product(s) did not meet eligibility, it will be detailed here.
- `returnOrder`: to simplify the self-serve flow, after a successful eligibility check an empty 
ZigZag return is created. This is then returned back to Horizon to continue the self-serve flow.
- `returnReasons`: a list of return reasons specific to the current site. Selected by the user when
choosing the reason for their return.

