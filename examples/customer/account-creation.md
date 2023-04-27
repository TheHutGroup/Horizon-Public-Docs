---
layout: default
title: Account Creation
parent: Customer & Authentication
grand_parent: Examples & Concepts
nav_order: 1
---

# Account Creation

THG's e-commerce platform allows for dynamic account creation forms, with the ability to add and configure form fields using our tooling, which will automatically propagate through to the website. This makes the account creation process completely dynamic.

To query the current account creation form and its field types, use the following GraphQL query:

```graphql
query {
  form(input: { identifier: "ACCOUNT_CREATION" }) {
    fields {
      name
      type
      required
      confirmable
      disabled
      answerOptions {
        optionKey
        translation
      }
      validators {
        name
        argument
      }
    }
  }
}
```

Each form field can have `answerOptions` for drop-down fields (e.g. title) and `validators` to validate the input server-side. Supported validators can be found [here](https://api.thehut.net/lfint/en/docs#ValidatorName).

An email address and password are required at a minimum for account creation. You can also include additional fields as needed, such as `fullName`, and specify `marketingConsent` with `marketingConsentAuditData` if applicable.
To create an account, use the following GraphQL mutation:

```graphql
mutation Registration {
  register(
    input: {
      email: "HorizonTestUser1@thehutgroup.com"
      password: "HorizonTestUser1@thehutgroup.com"
      fullName: "Horizon Test"
      marketingConsent: "I_DO_NOT_CONSENT_TO_RECEIVING_MARKETING_MATERIAL"
      marketingConsentAuditData: {
        messageShown: "Sign up for emails to receive marketing about offers and promotions"
        formIdentifier: "Registration"
        formLocation: "Registration Form"
      }
    }
  ) {
    fieldErrors {
      fieldName
      validators
      requiredButNotProvided
      invalidOption
    }
  }
}
```

Note that a customer object is accessible during registration and login in case you want to immediately access any customer data on these mutations.

The authentication token used to access sensitive data after a login or registration is returned as a cookie, so it does not need to be requested in the query.
