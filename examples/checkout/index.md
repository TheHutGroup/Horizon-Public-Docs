---
layout: default
title: Checkout
parent: Examples & Concepts
nav_order: 2
---

# Checkout

Once the user is finished on the basket, the basket can be handed over the checkout to complete. To do this, the customer must be logged in otherwise you will be presented with an error.

The response of the checkout handover is the checkout token and the URL. To get to checkout, redirect to the checkout URL, which will have a copy of the token in it as the `ct` query parameter. 

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

## Guest Checkout

Some sites on our platform also have the option of Guest Checkout. This is where customers can checkout without being logged in/creating an account. 
An email address is required to be able to place an order on our platform and can either be done upfront, before going to checkout, or from within checkout.

```graphql
mutation GuestCheckout {
  guestCheckout(input: {
    checkoutStartInput: {
      basketId: "c391611d-177f-4ab6-b604-3d3c415a3086:1610646726489"
      shippingDestination: GB
      currency: GBP
    }
    guestCheckoutEmailInput: {
      email: ""
      marketingConsent: I_DO_NOT_CONSENT_TO_RECEIVING_MARKETING_MATERIAL
      marketingConsentAuditData: {
        messageShown: "Sign up for emails to receive marketing about offers and promotions"
        formIdentifier: "GuestCheckout"
        formLocation: "GuestCheckout From Login Page"
      }
    }
  }) {
    error
    token
    checkoutUrl
  }
}
```

```graphql
mutation GuestCheckoutWithoutEmail {
  guestCheckoutWithoutEmail(input: {
    checkoutStartInput: {
      basketId: "c391611d-177f-4ab6-b604-3d3c415a3086:1610646726489"
      shippingDestination: GB
      currency: GBP
    }
  }) {
    error
    token
    checkoutUrl
  }
}
```

## Handover errors

The [CheckoutStartError](https://api.thehut.net/lfint/en/docs#CheckoutStartError) enum details situations that will prevent the checkout handover succeeding. Most of these (those starting with "invalid" or "no such") are a response to incorrect user input, or edge cases with the basket contents. The one most commonly seen is `BASKETS_MERGED` which can occur if checking-out immedidately after a login. See [here](../basket/index.md#basket-merge) for instructions for clearing that flag.
