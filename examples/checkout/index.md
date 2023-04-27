---
layout: default
title: Checkout
parent: Examples & Concepts
nav_order: 2
---

# Checkout

When a customer is finished with their basket, they can proceed to checkout to complete their purchase. To do this, the customer must be logged in, otherwise they will see an error message.

Upon checkout, the response includes a checkout token and URL. To redirect to the checkout page, go to the checkout URL, which will contain a copy of the token as the ct query parameter.

```graphql
mutation Checkout {
  checkout(
    input: {
      basketId: "c391611d-177f-4ab6-b604-3d3c415a3086:1610646726489"
      shippingDestination: GB
      currency: GBP
    }
  ) {
    error
    token
    checkoutUrl
  }
}
```

## Guest Checkout

Some sites on our platform allow for guest checkout, which enables customers to purchase items without creating an account or logging in. However, an email address is required to place an order and can either be provided before checkout or during the checkout process.

```graphql
mutation GuestCheckout {
  guestCheckout(
    input: {
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
    }
  ) {
    error
    token
    checkoutUrl
  }
}
```

```graphql
mutation GuestCheckoutWithoutEmail {
  guestCheckoutWithoutEmail(
    input: {
      checkoutStartInput: {
        basketId: "c391611d-177f-4ab6-b604-3d3c415a3086:1610646726489"
        shippingDestination: GB
        currency: GBP
      }
    }
  ) {
    error
    token
    checkoutUrl
  }
}
```

## Handover Checkout Errors

If a customer encounters an error during checkout, the [CheckoutStartError](https://api.thehut.net/lfint/en/docs#CheckoutStartError) enum lists possible situations that could prevent the checkout handover from succeeding. Most of these errors (those starting with "invalid" or "no such") are a result of incorrect user input or edge cases with the basket contents.
For example, the `BASKETS_MERGED` error may occur if a customer attempts to check out immediately after logging in. To clear this flag, follow the instructions [here](../basket/index.md#basket-merge).
