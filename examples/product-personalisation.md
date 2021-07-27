---
layout: default
title: Product Personalisation
parent: Examples & Concepts
nav_order: 12
---

# Product Personalisation

Some products on our platform can be personalised with custom information before being dispatched. This could be anything from custom engraving/packaging to just a note in the package that is bespoke to that order.
If a product is personalisable, this can be checked on the `ProductVariant` object like so:

```graphql
query GetProduct{
  product(sku:12852950 strict: false) {
    variants {
      sku
      title
      personalisationFields {
        name
        title
        type
        maxLength
        required
      }
    }
  }
}
```

An example response is below:
```json
{
  "data": {
    "product": {
      "variants": [
        {
          "sku": 12852950,
          "title": "Silver Mountain Water - 100ml Spray",
          "personalisationFields": [
            {
              "name": "Title",
              "title": "Title",
              "type": "FREE_TEXT",
              "maxLength": 30,
              "required": true
            },
            {
              "name": "MainMessage",
              "title": "Message in Package",
              "type": "FREE_TEXT",
              "maxLength": 100,
              "required": false
            }
          ]
        },
        {
          "sku": 12852951,
          "title": "Silver Mountain Water - 250ml Splash",
          "personalisationFields": null
        },
        {
          "sku": 12852952,
          "title": "Silver Mountain Water - 500ml Splash",
          "personalisationFields": null
        },
        {
          "sku": 12852953,
          "title": "Silver Mountain Water - 50ml Spray",
          "personalisationFields": null
        }
      ]
    }
  }
}
```

As can be seen above, this product has 2 fields that can be personalised named `Title` and `MainMessage`. These objects come with a title that can be displayed to the end user and have a type, max length and whether they are required.
For now, the only type is `FREE_TEXT` but in the future, we plan to support options like `SELECTION` where we provide a list of valid options.

## Validating Fields
Fields can be validated before the final submissions by using the validation query. This will check that the field name is valid for the SKU provided, if the value is within the correct length and makes sure the value isn't in a preconfigured disallow list.

Example query below:
```graphql
query ValidateField {
  personalisationValueValid(sku: 12852950 value: {
    name: "Title"
    value: "Hello"
  })
}
```

This will return one of the possible `ProductPersonalisationFieldValidationErrorType` options or null in the event of no issues.

## Validating Submissions
Before submitting an add to basket mutation for all the fields, it is highly recommended that you validate the submission as a whole. This can be done through the `personalisationSubmissionValid` query. This query will indivudally validate all of the fields but will also validate the submission to make sure no required fields are missing.
This validation will provide the most granular breakdown in case of any issues with the submission and should be done before the add to basket action. An empty list response means there are no errors.

```graphql
query ValidateSubmission {
  personalisationSubmissionValid(sku: 12852950 value: {
    fields: [
      {name: "Title", value: "Hello"}
      {name: "MainMessage", value: "Congratulations"}    
    ]
  }) {
    fieldName
    error
    requiredButNotProvided
  }
}
```

## Adding to Basket
Once the field and submission have been submitted, the personalised product can then be added to basket as following
```graphql
mutation AddPersonalisedProductToBasket {
  addPersonalisedProductToBasket(
    basketId: null
    sku: 12852950
    quantity: 1
    settings: { 
      currency: GBP, shippingDestination: GB 
    }
    personalisationValues: { 
      fields: [
        { name: "Title", value: "Hello" }
      ] 
    }
  ) {
    id
    items {
      product {
        title
      }
      personalisationValues {
        name
        value
      }
    }
  }
}
```

If an invalid submission is provided, a validation error will be thrown and null will be returned for the basket. This should never happen if the submission is validated before being added to basket.
