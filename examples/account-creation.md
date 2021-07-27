---
layout: default
title: Account Creation
parent: Examples & Concepts
nav_order: 1
---

# Account Creation
On THGs E-commerce platform, the account creation form is completely dynamic. This means the form fields can be added and configured in our tooling and they automatically propagate through to the website.

Because of this, we have a mechanism in the schema to query the current account creation form and its field types. This form query is also used in other areas where dynamic forms are used like the Profiles in the account section.

```graphql
query {
  form(input: {
    identifier:"ACCOUNT_CREATION"
  }){
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
At a minimum, an email address and password are required. Each form field can have `answerOptions` for drop down fields e.g. Title and `validators` to that we use to validate the input server side.

Supported validators can be found [here](https://api.thehut.net/lfint/en/docs#ValidatorName).

```graphql
mutation Registration {
  register(input: {
    email:"HorizonTestUser1@thehutgroup.com"
    password: "HorizonTestUser1@thehutgroup.com"
    fullName: "Horizon Test"
    marketingConsent: "I_DO_NOT_CONSENT_TO_RECEIVING_MARKETING_MATERIAL"
    marketingConsentAuditData: {
      messageShown: "Sign up for emails to receive marketing about offers and promotions"
      formIdentifier: "Registration"
      formLocation: "Registration Form"
    }
  }) {
    fieldErrors {
      fieldName
      validators 
      requiredButNotProvided
      invalidOption
    }
  }
}

```

A customer object is accessible during registration and login in case you want to immediately access any customer data on these mutations.

The authentication token used to access sensitive data after a login or register is returned as a cookie so does not need to be requested in the query. 
