---
layout: default
title: Loyalty
parent: Features
grand_parent: Examples & Concepts
nav_order: 2
---

# Loyalty
THG supports a variety of different loyalty schemes, from earn based schemes, to burn based schemes, or both combined. What this means is that when a customer makes purchases on site, or interacts with a few key areas of the site, they can earn points. Depending on the scheme, these points can move a user between tiers, which unlock rewards, or these points can be redeemed to get money off their order e.g. 500 points = £5.

## Opt In
Loyalty schemes have the option of asking the user if they want to be opted in or out. This is not a strict requirement of a loyalty scheme at THG but differs per site. 

If a scheme is enabled with loyalty opt in support, this information is gather during a customers initial registration, and is then something that can be changed in the customers account section on the site.

On the registration form, this is through a field that is called `loyaltyOptIn`. The below mutation can then be used to update these settings later and can be retried from the `Customer` object.

```graphql
mutation UpdateLoyaltySettings {
  updateLoyaltyOptIn(newValue: true) {
    error
    fieldErrors {
      validators
      requiredButNotProvided
    }
    customer {
      loyaltyOptIn
    }
  }
}
```

## Schemes 
A THG loyalty scheme comes in 2 main forms, a multi-tiers scheme or a tierless scheme. A tierless scheme may be a scheme where customers earns points and then can redeem those points against their purchases later. If a multi-tiered scheme is in place, each tier will have a threshold set against it which is the number of points required to reach that tier. Each tier also contains a list of rewards that are associated with it and this is used to help power a frontend in terms of which rewards to display. It is common for this to be fully powered by widgets instead of this functionality, depending on the flexibility required and the rewards contained within the scheme in use.

A tierless scheme may have Interaction awards (see below) but will return an empty list of tiers. 

## Points

Points are mostly earnt through the purchases of products. This information is exposed on the Price object and can differ per product and by logged in state due to product and customer specific rate overrides.

```graphql
query Product {
  product(sku:11370303, strict: false) {
    variants {
      price(currency: USD, shippingDestination: US) {
        price {
          scalarValue
        }
        earnableLoyaltyPoints(sku: 11370303)
      }
    }
  }
}
```

It is also possible to see the number of points that will be earnt for a given basket taking into account the products and quantities using the below query.

```graphql
query Basket {
  basket(id:"2298e807-50ee-4120-abfa-3c0f0ef78fca:1611587680727", settings:{
    currency: GBP
    shippingDestination: GB
  }) {
    earnableLoyaltyPoints
  }
}
```

Once a customer has earnt points, this information, alongside tiering information can be seeing against the customer.

[//]: # (TODO: Insert customer query here)

## Interaction Awards

Interaction awards are where customers are awarded points for actions on site that are not related to purchases. Some examples of this are leaving product reviews, creating an account or completing your customer profile.

For these interaction awards, we expose the awards that are enabled for the current scheme on site as well as the number of points the customer will receive.

This information can be queried in one of 2 ways, as a list, or for a specific award type.

```graphql
query LoyaltyScheme {
  loyaltyScheme {
    interactionAwards {
      type
      earnablePoints
    }
  }
  interactionAward(type: REVIEW) {
    type
    earnablePoints
  }
}
```

## Redeemable Points