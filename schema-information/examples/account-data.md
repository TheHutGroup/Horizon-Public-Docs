---
layout: default
title: Account Section
parent: GraphQL Examples
grand_parent: Schema Information
nav_order: 14
---

# Account Section

To power the account section of a site, we have included some example queries below on how do get certain data for these areas as we as the mutations for common actions.

## Viewing Order History

It is possible to either get all orders, or to filter the orders query to be a specific order number, or just orders of a specific status.

An order can be in 1 of 5 states:
- ORDER_PLACED
- PROCESSING
- DISPATCHED
- PAYMENT_PROBLEM
- CANCELLED

The `OUTSTANDING` filter will return the following as the order is not in a terminal state:
- ORDER_PLACED
- PROCESSING
- PAYMENT_PROBLEM

The `DISPATCHED` filter will return the following. This is a terminal state where the order was successful:
- DISPATCHED

The `COMPLETED` filter will return the following. This is shows all orders in a terminal state:
- DISPATCHED
- CANCELLED

The above filters could be used to show "Active", "Successful" and "All" orders.

```graphql
query OrderHistory{
  customer {
    active: orders(limit: 6, filter: {
      status: OUTSTANDING
    }) {
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
    completed: orders(limit: 6, filter: {
      status: COMPLETED
    }) {
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
query OrderDetails{
  customer {
    orders(limit: 1, filter: {
      orderNumber: "152050629"
    }) {
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
      groups: [
        { group: 0, reason: NO_LONGER_REQUIRED }
      ]
    }
  )
}
```

## Returning Orders
Todo when we work out where this logic should live

## Marketing Preferences
On our platform, you can opt in for marketing through SMS or Email channels. The current opt in status can be checked using the following query.

```graphql
query MarketingPreferences {
  customer {
    email:marketingPreferences(type: EMAIL)
    sms: marketingPreferences(type: SMS)
  }
}
```

To sign up to marketing, the following query can be used. As with all mutations related to opt in, audit data is needed for GDPR tracking purposes.

```graphql
mutation ChangeMarketingPreferences {
  updateMarketingPreferences(input: {
    type: EMAIL
    newValue: true
    auditData: {
      messageShown: "Do you want to sign up for deals?"
      formIdentifier: "Account section change details"
      formLocation: "/accountSettings.account"
    }
  }) {
    error
  }
}
```

## Account Details
To view the account details of a customer, this can simply be done by querying the fields on a customer.

```graphql
query AccountDetails {
  customer {
    fullName
    email
  }
}
```

To edit information about a customer, this works similarly to the account creation form where this is dynamic. To get the fields to show, you can run the following query.

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

The result of this can then be used to construct the `updateAccountSettings` mutation.
```graphql
mutation UpdateAccountSettings {
  updateAccountSettings(input: [{
    fieldName: "fullName"
    value: "New Name"
  }]) {
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

To update the email address of a customers account, this uses a separate mutation as the customer will be required to provide their password for this. This can be seen below.
```graphql
mutation UpdateEmail {
  updateEmailAddress(changes: {
    currentPassword: "pa55word"
    newEmailAddress: "newemail@thehutgroup.com"
  }) {
    error
    fieldErrors {
      fieldName
      validators
    }
  }
}
```

## Addresses
It is possible to query all of a customer's saved addresses. Note: This will not return addresses that were used for a previous order but have seen been deleted. Only addresses currently in their address book.

When adding an address, the following fields are mandatory:
- AddresseeName
- AddressLine1
- PostalCode
- Country

```graphql
query Addresses{
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

```graphql
mutation AddAddress {
  addAddress(input: {
    addresseeName: "Name"
    addressLine1: "House Number"
    addressLine2: "Road"
    addressLine3: "Village"
    addressLine4: "Town"
    addressLine5: "County"
    postalCode: "Postcode"
    country: GB
  })
}
```

```graphql
mutation DeleteAddress {
  deleteAddress(id: "11205BD1-0977-4EBA-B6C8-A296FF3DF35E") 
}
```

## Payment Cards
It is possible to view some data of your saved cards ti display in the account section. Note: This is a subset of the data stored for a customer's card and is a copy that is stored separately to the more sensitive data.

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

```graphql
mutation DeletedSavedCard {
  deletePaymentCard(cardId: "5B0D5AD9-0B7B-4E4B-8EF2-7DD1C17BCA6D") 
}
```

## Social Links
This shows all the links to social media accounts for social login. For here, a customer can also unlink their social account if needed.

Note: A link can be in a pending state when a social login is attempted for an email address that already exists, and the social provider doesn't perform email verification (like WeChat). In this case we put the social link in a pending state and include this as an error in the response (`SOCIAL_LINK_PENDING`). At this point, one of 2 mutations can be used, a normal login, then the `approveSocialLink` mutation. Or a `requestSocialLinkVerificationEmail` and the token in that email submitted to `loginAndApproveSocialLink`.

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

```graphql
mutation DeletedSocialLink {
  removeSocialLink(input: {
    socialLinkId: "30C62623-2569-4DCE-B156-277EFDDD1DE8"
  })
}
```

```graphql
# If logged in using your username and password, you can use the following
mutation ApproveSocialLink {
  approveSocialLink(input: {
    socialLinkId: "30C62623-2569-4DCE-B156-277EFDDD1DE8"
  })
}

# When attempting the link, the email that goes to the account owner will contain this token
mutation LoginAndApproveSocialLink {
  loginAndApproveSocialLink(input: {
    verificationToken: "30C62623-2569-4DCE-B156-277EFDDD1DE8"
  }) {
    error
  }
}
```

## Account Credit
Account credit can be earn and spent on site and is redeemed within checkout. Because our sites support multiple currencies, and also multiple locales where a customer is shared across those locales, credit can be earnt in multiple currencies. Note: It can only be spent in a single currency at one time.

```graphql
query CreditAccounts {
  customer {
    creditAccounts(filter: {
      currency: GBP # A customer can earn credit in multiple currencies
    }) {
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