---
layout: default
title: Cancel Return
parent: Returns
grand_parent: Examples & Concepts
nav_order: 4
---
# Cancel Return
In the event that we wish to cancel a return, we do so through a mutation:

```graphql
extend type Mutation {
    cancelReturn(shipmentNumber: Int!): Void @authenticated @if(feature: ORDER_RETURNS)
}
```
 By providing the shipment number we cancel the return for that shipment number.
