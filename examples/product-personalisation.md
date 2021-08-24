---
layout: default
title: Product Personalisation
parent: Examples & Concepts
nav_order: 12
---

# Product Personalisation

Some products on our platform can be personalised with custom information before being dispatched. This could be anything from custom engraving/packaging to just a note in the package that is bespoke to that order.
We currently support 2 different types of personalisation field types, they are FREE_TEXT and SINGLE_SELECTION.

If a product is personalisable, this can be checked on the `ProductVariant` object like so:

```graphql
query GetProduct{
  product(sku:12852950 strict: false) {
    variants {
      sku
      title
      personalisationFields {
        ... on FreeTextProductPersonalisationField {
          name
          title
          type
          maxLength
          required
        }
        ... on SingleSelectionProductPersonalisationField {
          name
          title
          type
          required
          options {
            name
            value
          }
        }
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
      "sku": 13165630,
      "title": "Water Bottle - 250ml",
      "personalisationFields": [
        {
          "name": "name",
          "title": "Name",
          "type": "FREE_TEXT",
          "maxLength": 10,
          "required": true
        },
        {
          "name": "message",
          "title": "Message",
          "type": "FREE_TEXT",
          "maxLength": 30,
          "required": true
        },
        {
          "name": "template",
          "title": "Template",
          "type": "SINGLE_SELECTION",
          "required": true,
          "options": [
            {
              "name": "Design 5",
              "value": "unicorns"
            },
            {
              "name": "Design 1",
              "value": "mountains"
            },
            {
              "name": "Design 2",
              "value": "balloons"
            },
            {
              "name": "Design 3",
              "value": "trees"
            },
            {
              "name": "Design 4",
              "value": "hearts"
            },
            {
              "name": "Design 6",
              "value": "trumpets"
            }
          ]
        }
      ]
    }
  }
```

As can be seen above, this product has 3 fields that can be personalised named `Name`, `Message` and `Template`. These objects come with a title that can be displayed to the end user and have a type and whether they are required.
Depending on the field type, they will have different options. For example the `FREE_TEXT` field type has a max length as this is provided by the end user. The `SINGLE_SELECTION` type has a list of options with names and values. `name` is intended for display purposes and `value` is what is submitted to the API.

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
Before submitting an add to basket mutation for all the fields, it is highly recommended that you validate the submission as a whole. This can be done through the `personalisationSubmissionValid` query. This query will individually validate all of the fields but will also validate the submission to make sure no required fields are missing.
This validation will provide the most granular breakdown in case of any issues with the submission and should be done before the add to basket action. An empty list response means there are no errors.

```graphql
query ValidateSubmission {
  personalisationSubmissionValid(sku: 12852950 value: {
    fields: [
      {name: "name", value: "Hello"}
      {name: "message", value: "Congratulations"}    
      {name: "template", value: "unicorns"}    
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

NOTE: When we return the `personalisationValues` of a basket item where one of the fields is of type `SINGLE_SELECTION` we will return the `name` of the option as opposed to the `value` so that this can be displayed to the customer. 

If an invalid submission is provided, a validation error will be thrown and null will be returned for the basket. This should never happen if the submission is validated before being added to basket.
