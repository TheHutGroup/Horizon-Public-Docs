---
layout: default
title: Account Section
parent: Customer & Authentication
grand_parent: Examples & Concepts
nav_order: 2
---

# Account Section

This section provides example queries and mutations to support the account section of a site. We have included some example queries below on how to get certain data for these areas as well as the mutations for common actions.

## Viewing Order History

It is possible to either get all orders or filter the orders query to a specific order number or just orders of a specific status.

An order can be in one of 7 states:

- ORDER_PLACED
- PROCESSING
- DISPATCHED
- PAYMENT_PROBLEM
- CANCELLED
- READY_TO_COLLECT
- COLLECTED

The `OUTSTANDING` filter returns orders that are not in a terminal state, including:

- ORDER_PLACED
- PROCESSING
- PAYMENT_PROBLEM
- READY_TO_COLLECT

The `DISPATCHED` filter returns orders that are in a terminal state, specifically:

- DISPATCHED

The `COMPLETED` filter shows all orders in a terminal state, including:

- DISPATCHED
- CANCELLED
- COLLECTED

The above filters could be used to show "Active", "Successful" and "All" orders.

```graphql
query OrderHistory {
  customer {
    active: orders(limit: 6, filter: { status: OUTSTANDING }) {
      orders {
        orderNumber
        createdAt
        status
        products {
          productVariant {
            images {
              thumbnail
            }
          }
        }
        totalCost {
          currency
          amount
        }
      }
      hasMore
      total
    }
    completed: orders(limit: 6, filter: { status: COMPLETED }) {
      orders {
        orderNumber
        createdAt
        status
        products {
          productVariant {
            images {
              thumbnail
            }
          }
        }
        totalCost {
          currency
          amount
        }
      }
      hasMore
      total
    }
  }
}
```

```graphql
query OrderDetails {
  customer {
    orders(limit: 1, filter: { orderNumber: "152050629" }) {
      orders {
        orderNumber
        createdAt
        status
        totalQuantity
        products {
          sku
          productVariant {
            images {
              original
            }
            title
          }
          quantity
          costPerUnit {
            currency
            amount
          }
          status
          dispatchDate
          deliveryDateRange {
            from
            to
          }
          deliveryMethod
          trackingUrls
          specialOfferGroup
        }
        eligibleForSelfServiceDOR
        deliveryCost {
          currency
          amount
        }
        totalCost {
          currency
          amount
        }
        discounts {
          amount {
            currency
            amount
          }
          message
        }
        deliveryAddress {
          addresseeName
          addressLine1
          addressLine2
          addressLine3
          addressLine4
          addressLine5
          postalCode
          country
        }
        discussions {
          discussions {
            messages {
              messages {
                message
              }
            }
          }
        }
      }
    }
  }
}
```

## Adding new Order Statuses

Horizon serves as the entry point for external clients and aggregates data from a wide range of underlying systems within THG.

In order to aggregate this data and serve it in a logical way to external clients, Horizon maps these statuses into Horizon Order Statuses.

The statuses supported by Horizon are declared in the GraphQL schema, within an enum:

```
  enum OrderStatus {
    ORDER_PLACED
    PROCESSING
    DISPATCHED
    PAYMENT_PROBLEM
    CANCELLED
    READY_TO_COLLECT @if(feature: CLICK_AND_COLLECT)
    COLLECTED @if(feature: CLICK_AND_COLLECT)
  }
```

In order to add a new status in Horizon, it needs to be added within the ENUM declared within the schema and needs to be confirmed with the teams managing the underlying systems that maintain order statuses internally.

## Cancelling Orders

After an order is placed, there is a window of around 30 minutes where it can be cancelled before it is dispatched. When cancelling an order, it is possible to either cancel the whole thing, or just some specific products.

When attempting a partial cancellation, you need to first check the order and information about the `OrderProduct.specialOfferGroup`. This is because we restrict the ability to cancel part of an offer group. We recommend showing products in the order cancellation page, grouped by special offer group.

What this means is that for an order where we have Buy Product X, Get Y free, you cannot cancel Product X without cancelling Product Y as they have had a single offer apply to both of them so have to be cancelled together.

If no offers are on any of the products, you are able to cancel them individually without any restriction.

Note: Before cancelling an order, you should check if the order is cancellable.

```graphql
mutation CancelOrder {
  cancelOrder(input: { orderNumber: "95726695", reason: NO_LONGER_REQUIRED })
}

mutation CancelOrderProducts {
  cancelOrderProducts(
    input: {
      orderNumber: "95726695"
      products: [
        { sku: 123, quantity: 1, reason: NO_LONGER_REQUIRED }
        { sku: 456, quantity: 1, reason: NO_LONGER_REQUIRED }
      ]
    }
  )
}

mutation CancelOfferGroups {
  cancelOrderSpecialOfferGroups(
    input: {
      orderNumber: "95726695"
      groups: [{ group: 0, reason: NO_LONGER_REQUIRED }]
    }
  )
}
```

## Returning Orders

TODO: Documentation coming soon

## Marketing Preferences

On our platform, customers can opt-in for marketing through SMS or Email channels. The current opt-in status can be checked using the following query:

```graphql
query MarketingPreferences {
  customer {
    email: marketingPreferences(type: EMAIL)
    sms: marketingPreferences(type: SMS)
  }
}
```

To sign up for marketing, you can use the following mutation. As with all mutations related to opt-in, audit data is needed for GDPR tracking purposes.

```graphql
mutation ChangeMarketingPreferences {
  updateMarketingPreferences(
    input: {
      type: EMAIL
      newValue: true
      auditData: {
        messageShown: "Do you want to sign up for deals?"
        formIdentifier: "Account section change details"
        formLocation: "/accountSettings.account"
      }
    }
  ) {
    error
  }
}
```

## Account Details

To view a customer's account details, you can simply query the fields on a customer.

```graphql
query AccountDetails {
  customer {
    fullName
    email
  }
}
```

To edit a customer's account information, the `accountSettingsForm` query can be used to retrieve the form fields for updating the account settings. The result can then be used to construct the `updateAccountSettings` mutation.

```graphql
query AccountSettingForm {
  accountSettingsForm {
    identifier
    fields {
      name
      type
      validators {
        name
        argument
      }
      required
      confirmable
      answerOptions {
        optionKey
        translation
      }
      defaultValue
    }
  }
}
```

```graphql
mutation UpdateAccountSettings {
  updateAccountSettings(input: [{ fieldName: "fullName", value: "New Name" }]) {
    error
    fieldErrors {
      fieldName
      validators
      requiredButNotProvided
      invalidOption
    }
  }
}
```

To update a customer's email address, the updateEmailAddress mutation can be used. The customer must provide their current password and the new email address.

```graphql
mutation UpdateEmail {
  updateEmailAddress(
    changes: {
      currentPassword: "pa55word"
      newEmailAddress: "newemail@thehutgroup.com"
    }
  ) {
    error
    fieldErrors {
      fieldName
      validators
    }
  }
}
```

## Addresses

To query all of a customer's saved addresses, the addresses field can be queried on a customer object.
Note: This will not return addresses that were used for a previous order but have since been deleted. Only addresses currently in their address book will be returned.

```graphql
query Addresses {
  customer {
    addresses {
      addresses {
        id
        addressUuid
        address {
          addresseeName
          addressLine1
          addressLine2
          addressLine3
          addressLine4
          addressLine5
          postalCode
          country
        }
      }
      total
      hasMore
    }
  }
}
```

When adding an address, the following fields are mandatory:

- AddresseeName
- AddressLine1
- PostalCode
- Country

```graphql
mutation AddAddress {
  addAddress(
    input: {
      addresseeName: "Name"
      addressLine1: "House Number"
      addressLine2: "Road"
      addressLine3: "Village"
      addressLine4: "Town"
      addressLine5: "County"
      postalCode: "Postcode"
      country: GB
    }
  )
}
```

To delete an address, the deleteAddress mutation can be used, specifying the address ID.

```graphql
mutation DeleteAddress {
  deleteAddress(id: "11205BD1-0977-4EBA-B6C8-A296FF3DF35E")
}
```

## Payment Cards

To view some data of a customer's saved payment cards, the paymentCards field can be queried on a customer object.
Note: This is a subset of the data stored for a customer's card and is a copy that is stored separately to the more sensitive data.

```graphql
query SavedCards {
  customer {
    paymentCards {
      cards {
        id
        card {
          nameOnCard
          obfuscatedCardNumber
          validFromMonth
          validFromYear
          validToMonth
          validToYear
          issueNumber
          type
        }
      }
      total
      hasMore
    }
  }
}
```

To delete a saved payment card, the `deletePaymentCard` mutation can be used, specifying the card ID.

```graphql
mutation DeletedSavedCard {
  deletePaymentCard(cardId: "5B0D5AD9-0B7B-4E4B-8EF2-7DD1C17BCA6D")
}
```

## Social Links

This query returns a list of social media accounts linked to the customer's account.

Note: If a social login is attempted for an email address that already exists and the social provider does not perform email verification (e.g. WeChat), the social link may be in a pending state. In this case, the query will return SOCIAL_LINK_PENDING as an error in the response. To resolve this issue, customers can either use a normal login and then the approveSocialLink mutation, or use the requestSocialLinkVerificationEmail mutation and submit the token in the email to loginAndApproveSocialLink.

```graphql
query SocialLinks {
  customer {
    socialLinks {
      socialLinkId
      socialLoginProvider {
        code
      }
      username
      status
    }
  }
}
```

Customers can also unlink their social account using this mutation:

```graphql
mutation DeletedSocialLink {
  removeSocialLink(
    input: { socialLinkId: "30C62623-2569-4DCE-B156-277EFDDD1DE8" }
  )
}
```

This mutation approves a pending social link. To use this mutation, customers must log in using their username and password.

```graphql
mutation ApproveSocialLink {
  approveSocialLink(
    input: { socialLinkId: "30C62623-2569-4DCE-B156-277EFDDD1DE8" }
  )
}
```

When attempting to link a social media account, the email that goes to the account owner will contain this token.

```graphql
mutation LoginAndApproveSocialLink {
  loginAndApproveSocialLink(
    input: { verificationToken: "30C62623-2569-4DCE-B156-277EFDDD1DE8" }
  ) {
    error
  }
}
```

## Account Credit

Customers can earn and spend account credit on the site, and it is redeemed during checkout. Because our sites support multiple currencies/locales for the same customer, customers can earn credit in multiple currencies, but it can only be spent in a single currency at a time.

```graphql
query CreditAccounts {
  customer {
    creditAccounts(
      filter: {
        currency: GBP # A customer can earn credit in multiple currencies
      }
    ) {
      currency
      balance {
        currency
        amount
      }
      expiringIn(days: 7) {
        currency
        amount
      }
      actions {
        actions {
          id
          type
          status
          amount {
            currency
            amount
          }
          amountUsed {
            currency
            amount
          }
          amountAvailable {
            currency
            amount
          }
          message
          order {
            orderNumber
          }
          addedAt
          expiresAt
        }
        total
        hasMore
      }
    }
  }
}
```
