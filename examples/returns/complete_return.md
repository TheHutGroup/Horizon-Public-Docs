---
layout: default
title: Complete Return
parent: Returns
grand_parent: Examples & Concepts
nav_order: 3
---
# Complete Return
Once the user chooses a carrier service for each return, we can submit them and thus complete
all returns.

## The Endpoint
The mutation looks as follows:

```graphql
extend type Mutation {
    "Completes returns for the current return Order"
    completeReturn(input: [CompleteReturnInput!]!): [ShipmentWithLabel!] @authenticated @if(feature: ORDER_RETURNS)
}
```

### Input
The mutation takes in a list of all the returns to complete:

```graphql
input CompleteReturnInput @if(feature: ORDER_RETURNS)  {
    orderNumber: String!
    retailerCode: String!
    returnId: String!
    carrierServiceId: Int
    numberOfProducts: Int!
    dropOffPointId: String
    inStore: Boolean!
}
```

- `orderNumber`: the order number.
- `retailerCode`: the retailer code.
- `returnId`: the ZigZag return Id for the return.
- `carrierServiceId`: the Id of the carrier service chosen by the customer for this return.
- `numberOfProducts`: the cumulative quantity of all items to be sent via the chosen carrier service.
- `dropOffPointId`: if the chosen carrier service uses dropoff points, then we include the Id of 
the chosen dropoff point.
- `inStore`: if the chosen carrier service was an in-store return (not through ZigZag), then we set
this to `true`. The values of both the `carrierServiceId` and `dropOffPointId` can be `null`.

### Result
As a result of this mutation you will receive a list of labels for each return that has been submitted.

```graphql
type ShipmentWithLabel @if(feature: ORDER_RETURNS) {
  shipmentNumber: Int!
  returnId: String!
  labelWithDetails: LabelWithDetails!
}

type LabelWithDetails @if(feature: ORDER_RETURNS) {
  label: String!
  deliveryInstructions: DeliveryInstructions!
  warehouseDC: String!
}

type DeliveryInstructions {
  title: String!
  generalInstructions: [String!]!
  customsDeclarationInstructions: String!
  refundTimeInstructions: String!
  collectionReferenceText: String!
  returnCode: String!
  instructionsFooter: String!
  footer: String!
}
```
