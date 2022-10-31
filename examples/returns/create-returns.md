---
layout: default
title: Create Returns
parent: Returns
grand_parent: Examples & Concepts
nav_order: 2
---
# Create Returns
Once the customer chooses the products and their corresponding return reasons, we can create returns 
for these products.

## Returns and Return Orders
Firstly, let's make clear what we mean by a return order and returns. To create a return for an 
order in ZigZag we first create what's known as a return order. This return order basically tells
ZigZag that we could be expecting a return for this order.

When the user wants to go through with a return, that's when we create a return against a return
order. Usually, we would only create one return against a return order as the customer would send
one shipment back with the products they wish to return. However, if the products returned need to 
go to separate warehouses (dropship products, for example), then we create multiple returns against 
the return order. 

The customer can then choose a different carrier service for the products that they need to send
to specific warehouses.

## The Endpoint
We create returns with a singular mutation:

```graphql
extend type Mutation {
    "Creates returns for the current return order"
    # Will return multiple Returns depending on if some items need to go to different warehouses
    createReturn(input: CreateReturnInput!): [Return!]! @authenticated @if(feature: ORDER_RETURNS)
}
```

### Input
The mutation takes in the following input:

```graphql
input CreateReturnInput @if(feature: ORDER_RETURNS) {
    products: [ReturnProductInput!]!
    notes: String
    giftCardSelected: Boolean!
    orderNumber: OrderNumber!
    retailerCode: String!
    returnOrderId: Int!
}

input ReturnProductInput @if(feature: ORDER_RETURNS) {
    productSku: SKU!
    quantity: Int!
    returnReasonId: Int!
    returnOption: ReturnOption!
    returnReasonDetails: String
    returnReasonAcknowledged: String
    exchangeNotes: String
}
```

- `products`: contains details about each product that is to be returned.
- `notes`: any misc notes to include about this product.
- `giftCardSelected`: is this a gift card.
- `orderNumber`: the order number of the current return.
- `retailerCode`: the retailer code of the current client.
- `returnOrderId`: the return order Id. This is fetched from the return eligibility call as 
described in the prevuious section.

### Result
The endpoint will return a list of returns, of which there could be multiple (as explained above).

```graphql
type Return @if(feature: ORDER_RETURNS) {
    returnId: String!
    dcWarehouse: String!
    products: [Product!]!
    carrierServices: [CarrierService!]!
}
```

- `returnId`: the ZigZag return Id for the return against the `products` in this object. This should
be used in the inputs 
- `dcWarehouse`: this is the Id of the warehouse that the products of this return will be addressed
to. In cases of multiple returns, this will be different for every return in the list.
- `products`: the products that are a part of this return.
- `carrierServices`: the list of carrier services available to the customer for this return - 
addressed to the provided `dcWarehouse`.

## Next Step
After a return(s) is created, we allow the customer to choose a [carrier service](../returns/complete_return.html).
