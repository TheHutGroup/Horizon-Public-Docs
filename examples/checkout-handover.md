---
layout: default
title: Checkout Handover
parent: Examples & Concepts
nav_order: 11
---

# Checkout Handover

Once the user is finished on the basket, the basket can be handed over the checkout to complete. To do this, the customer must be logged in otherwise you will be presented with an error.

The response of the checkout handover is the checkout token and the URL. To get to check, the url format is ${checkoutUrl}?ct=${token}.

```graphql
mutation Checkout {
  checkout(input: {
    basketId: "c391611d-177f-4ab6-b604-3d3c415a3086:1610646726489"
    shippingDestination: GB
    currency: GBP
  }) {
    error
    token
    checkoutUrl
  }
}
```