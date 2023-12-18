---
layout: default
title: Product Personalisation
parent: Features
grand_parent: Examples & Concepts
nav_order: 2
---

# Product Personalisation

Some products on our platform can be personalised with custom information before being dispatched. This could be anything from custom engraving to adding personalised messages on packaging and labels.

We currently support 3 different types of personalisation fields, these are FREE_TEXT, SINGLE_SELECTION and MULTI_SELECTION. 

A `FREE_TEXT` field is used for the input of text, such as a name or a message. A `SINGLE_SELECTION` field is used for the input of a template design between the available selection. A `MULTI_SELECTION` field is used for the input of multiple products (each with a quantity) 
within the available selection to complete a pick 'n mix box.

If a product is personalisable, all the personalisation data can be retrieved on the `ProductVariant` object like so:

```graphql
query GetProductVariant{
    productVariant(sku:12852950) {
        sku
        title
        personalisationData {
            personalisationFields {
                ... on FreeTextProductPersonalisationField {
                    name
                    title
                    type
                    maxLength
                    required
                    rotation
                    incompatibleWith
                    numberOfLines
                }
                ... on SingleSelectionProductPersonalisationField {
                    name
                    title
                    type
                    required
                    rotation
                    incompatibleWith
                    options {
                        name
                        value
                        displayAsset
                        previewAssetSetIdentifier
                        order
                    }
                }
                ... on MultiSelectionProductPersonalisationField {
                    name
                    title
                    type
                    required
                    rotation
                    incompatibleWith
                    options {
                        name
                        value
                        displayAsset
                        previewAssetSetIdentifier
                        order
                    }
                    fixedQuantity
                }
            }
            personalisationFonts {
                fontId
                name
                family
                weight
                lineHeight
                letterSpacing
                maxPreviewFontSize
            }
            personalisationPreviews {
                previewImages {
                    images {
                        size
                        url
                    }
                    imagesWithAssetSets {
                        assetSet
                        images {
                            size
                            url
                        }
                    }
                }
                locations {
                    x
                    y
                    width
                    height
                    defaultFontColour
                    fieldName
                }
                face
            }
            personalisationSupportImages {
                face
                supportImages {
                    images {
                        size
                        url
                    }
                    imagesWithAssetSets {
                        assetSet
                        images {
                            size
                            url
                        }
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
    "productVariant": {
      "sku": 12852950,
      "title": "Personalised Original 360g Bar - White",
      "personalisationData": {
        "personalisationFields": [
          {
            "name": "name",
            "title": "Name",
            "type": "FREE_TEXT",
            "maxLength": 10,
            "required": true,
            "rotation": null,
            "incompatibleWith": [],
            "numberOfLines": 1
          },
          {
            "name": "message",
            "title": "Message",
            "type": "FREE_TEXT",
            "maxLength": 30,
            "required": true,
            "rotation": null,
            "incompatibleWith": [],
            "numberOfLines": 1
          },
          {
            "name": "template",
            "title": "Template",
            "type": "SINGLE_SELECTION",
            "required": true,
            "rotation": null,
            "incompatibleWith": [],
            "options": [
              {
                "name": "mountains",
                "value": "Design 1",
                "displayAsset": "/productimg/70/70/13165630-1345095262265915.png",
                "previewAssetSetIdentifier": "mountains",
                "order": 0
              },
              {
                "name": "balloons",
                "value": "Design 2",
                "displayAsset": "/productimg/70/70/13165630-1665047029317682.jpg",
                "previewAssetSetIdentifier": "balloons",
                "order": 1
              },
              {
                "name": "hearts",
                "value": "Design 4",
                "displayAsset": "/productimg/70/70/13165630-1665047029317682.jpg",
                "previewAssetSetIdentifier": "hearts",
                "order": 2
              },
              {
                "name": "trees",
                "value": "Design 3",
                "displayAsset": "/productimg/70/70/13165630-1665047029317682.jpg",
                "previewAssetSetIdentifier": "trees",
                "order": 3
              },
              {
                "name": "trumpets",
                "value": "Design 6",
                "displayAsset": "/productimg/70/70/13165630-1665047029317682.jpg",
                "previewAssetSetIdentifier": "trumpets",
                "order": 4
              },
              {
                "name": "unicorns",
                "value": "Design 5",
                "displayAsset": "/productimg/70/70/13165630-1665047029317682.jpg",
                "previewAssetSetIdentifier": "unicorns",
                "order": 5
              }
            ]
          }
        ],
        "personalisationFonts": [
          {
            "fontId": "914936535851663364",
            "name": "Block",
            "family": "Cormorant Garamond",
            "weight": 700,
            "lineHeight": 14,
            "letterSpacing": 0,
            "maxPreviewFontSize": 12
          }
        ],
        "personalisationPreviews": [
          {
            "previewImages": {
              "images": [],
              "imagesWithAssetSets": [
                {
                  "assetSet": "trumpets",
                  "images": [
                    {
                      "size": "THUMBNAIL",
                      "url": "/productimg/70/70/13165630-1765049582767535.jpg"
                    },
                    {
                      "size": "SMALLPROD",
                      "url": "/productimg/100/100/13165630-1765049582767535.jpg"
                    },
                    {
                      "size": "LARGEPRODUCT",
                      "url": "/productimg/300/300/13165630-1765049582767535.jpg"
                    },
                    {
                      "size": "CAROUSEL",
                      "url": "/productimg/480/480/13165630-1765049582767535.jpg"
                    },
                    {
                      "size": "MAGNIFY",
                      "url": "/productimg/1600/1600/13165630-1765049582767535.jpg"
                    },
                    {
                      "size": "PRODUCT",
                      "url": "/productimg/130/130/13165630-1765049582767535.jpg"
                    },
                    {
                      "size": "ORIGINAL",
                      "url": "/productimg/original/13165630-1765049582767535.jpg"
                    }
                  ]
                },
                {
                  "assetSet": "unicorns",
                  "images": []
                },
                {
                  "assetSet": "halloween",
                  "images": []
                },
                {
                  "assetSet": "mountains",
                  "images": []
                },
                {
                  "assetSet": "balloons",
                  "images": []
                },
                {
                  "assetSet": "hearts",
                  "images": []
                },
                {
                  "assetSet": "trees",
                  "images": []
                }
              ]
            },
            "locations": [
              {
                "x": 1,
                "y": 1,
                "width": 1,
                "height": 1,
                "defaultFontColour": "#ff0000",
                "fieldName": "name"
              },
              {
                "x": 2,
                "y": 2,
                "width": 2,
                "height": 2,
                "defaultFontColour": "#ff0000",
                "fieldName": "message"
              }
            ],
            "face": "FRONT"
          }
        ],
        "personalisationSupportImages": [
          {
            "face": "SUPPORT1",
            "supportImages": {
              "images": [],
              "imagesWithAssetSets": [
                {
                  "assetSet": "trees",
                  "images": [
                    {
                      "size": "THUMBNAIL",
                      "url": "/productimg/70/70/13165630-1375095242860850.png"
                    },
                    {
                      "size": "SMALLPROD",
                      "url": "/productimg/100/100/13165630-1375095242860850.png"
                    },
                    {
                      "size": "LARGEPRODUCT",
                      "url": "/productimg/300/300/13165630-1375095242860850.png"
                    },
                    {
                      "size": "CAROUSEL",
                      "url": "/productimg/480/480/13165630-1375095242860850.png"
                    },
                    {
                      "size": "MAGNIFY",
                      "url": "/productimg/1600/1600/13165630-1375095242860850.png"
                    },
                    {
                      "size": "PRODUCT",
                      "url": "/productimg/130/130/13165630-1375095242860850.png"
                    },
                    {
                      "size": "ORIGINAL",
                      "url": "/productimg/original/13165630-1375095242860850.png"
                    }
                  ]
                }
              ]
            }
          },
          {
            "face": "SUPPORT2",
            "supportImages": {
              "images": [],
              "imagesWithAssetSets": []
            }
          },
          {
            "face": "SUPPORT3",
            "supportImages": {
              "images": [],
              "imagesWithAssetSets": []
            }
          },
          {
            "face": "SUPPORT4",
            "supportImages": {
              "images": [],
              "imagesWithAssetSets": []
            }
          }
        ]
      }
    }
  }
}
```

As can be seen above, this product has a configuration assigned. The configuration has 3 fields that can be personalised, named `Name`, `Message` and `Template`. These objects come with a `title` that can be displayed to the end user, a `type`, whether they are `required` 
and other common properties.

Depending on the field type, they will have additional options. For example the `FREE_TEXT` field type has a `max length`, which is the max amount of characters the user can input. The `SINGLE_SELECTION` type has a list of `options` with names and values. `name` can be 
displayed to the end user, `value` is submitted to the API, `dislayAsset` has a small carousel image for the specific template and `order` is the template position in the carousel.

The configuration also has fonts, previews and supportImages. Since the returned configuration is very long not all supportImages and previewImages sizes have been included here, but images are available in all sizes and template designs (`assetSets`).

There are always support images and preview images for all the existing `previewAssetSetIdentifier` so that when the customer clicks on one template design (= one `option`) only the images relevant to that template need to appear on the PDP.

To know specifics and meaning of all these properties see Personalisation docs on Confluence. 


An example response of a config with `MULTI_SELECTION` fields is:
```json
{
  "data": {
    "productVariant": {
      "sku": 14845090,
      "title": "6-BAR GIFT PACK",
      "personalisationData": {
        "personalisationFields": [
          {
            "name": "toblerone_mix_tastes",
            "title": "Toblerone_mix_tastes",
            "type": "MULTI_SELECTION",
            "required": true,
            "rotation": null,
            "incompatibleWith": [],
            "options": [
              {
                "name": "dark chocolate",
                "value": "13165630",
                "displayAsset": null,
                "previewAssetSetIdentifier": null,
                "order": 0
              },
              {
                "name": "fruit chocolate",
                "value": "13165635",
                "displayAsset": null,
                "previewAssetSetIdentifier": null,
                "order": 0
              },
              {
                "name": "milk chocolate",
                "value": "13165640",
                "displayAsset": null,
                "previewAssetSetIdentifier": null,
                "order": 0
              }
            ],
            "fixedQuantity": 3
          },
          {
            "name": "toblerone_mix_tastes2",
            "title": "Toblerone_mix_tastes2",
            "type": "MULTI_SELECTION",
            "required": true,
            "rotation": null,
            "incompatibleWith": [],
            "options": [
              {
                "name": "orange chocolate",
                "value": "13165655",
                "displayAsset": null,
                "previewAssetSetIdentifier": null,
                "order": 0
              },
              {
                "name": "almond chocolate",
                "value": "13165650",
                "displayAsset": null,
                "previewAssetSetIdentifier": null,
                "order": 0
              }
            ],
            "fixedQuantity": 1
          }
        ],
        "personalisationFonts": [],
        "personalisationPreviews": [
          {
            "previewImages": {
              "images": [],
              "imagesWithAssetSets": []
            },
            "locations": [
              {
                "x": 1,
                "y": 1,
                "width": 1,
                "height": 1,
                "defaultFontColour": null,
                "fieldName": "toblerone_mix_tastes"
              }
            ],
            "face": "FRONT"
          }
        ],
        "personalisationSupportImages": []
      }
    }
  }
}
```

This is the kind of config most likely used to set up a pick 'n mix product without personalisation additions. 

Each `MULTI_SELECTION` field contains a group of products. The customer will be able to choose one or a combination of them. Each `option` is an individual product, and the `value` is containing its SKU. Each MULTI_SELECTION field also has a `fixedQuantity`. 
The customer will have to select a combination of one or more products, within the group, to reach the total quantity specified here.

For validation purposes all configs have at least one `location`. The location indicates were to put the text on the preview image. In case there is no preview, the location is useless and should not be displayed in the frontend. As a rule of thumb, if the location `fieldName`,
is not a `FREE_TEXT` field then the location should be ignored.

## Validating Fields
Fields can be validated before the final submissions by using the validation query. This will check that the field `name` is valid for the SKU provided, if the `value` is within the correct length and makes sure the value isn't in a preconfigured disallow list.

In the singleSelection and the multiSelection case it will also check that the `value` (not the template name) exists in the config. 

In the multiSelection case it will additionally check that the sum of the quantities is equal to the `fixedQuantity`.

Example query for `FREE_TEXT` field:
```graphql
query ValidateFreeTextField {
  personalisationValueValid(sku: 12852950 value: {
    name: "message"
    value: "Happy Birthday"
  })
}
```
Example query for `SINGLE_SELECTION` field:
```graphql
query ValidateSingleSelectionField {
  personalisationValueValid(sku: 12852950 value: {
    name: "template"
    value: "Design 4"
  })
}
```
Example query for `MULTI_SELECTION` field:
```graphql
query ValidateMultiSelectionField {
  personalisationValueValid(sku: 14845090 value: {
      name: "toblerone_mix_tastes"
      multiSelectionSubmissions: [
          {quantity: 1, value: "13165635"}
          {quantity: 2, value: "13165640"}
      ]
  })
}
```

This will return one of the possible `ProductPersonalisationFieldValidationErrorType` options or null in the event of no issues.

## Validating Submissions
Before submitting an add to basket mutation for all the fields, its highly recommended that you validate the submission as a whole. This can be done through the `personalisationSubmissionValid` query. This query will individually validate all of the fields but will also 
validate the submission to make sure no required fields are missing, and the correct input is submitted according to the field type. 

It will also make sure that the fontId is there if it needs to be, and it's not if it's going to cause an exception.


This validation will provide the most granular breakdown in case of any issues with the submission and should be done before the add to basket action. 

```graphql
query ValidateSubmission {
    personalisationSubmissionValid(sku: 13165645 value: {
        fieldSubmissionList: [
            {name: "name", value: "Lizzo"}
            {name: "message", value: "Its about time"}
            {name: "template", value: "Design 4"}
        ]
        fontId: "914936535851663364"
    }) {
        fieldName
        error
        requiredButNotProvided
    }
}
```

For the `MULTI_SELECTION` (the pick 'n mix) case:
```graphql
query ValidateSubmissionMultiSelection {
    personalisationSubmissionValid(sku: 14845090 value: {
        fieldSubmissionList: [
            {name: "toblerone_mix_tastes", multiSelectionSubmissions: [
                {value: "13165635", quantity:2},
                {value: "13165640", quantity:1}
            ]}
            {name: "toblerone_mix_tastes2", multiSelectionSubmissions: [
                {value: "13165655", quantity:1}
            ]}
        ]
    }) {
        fieldName
        error
        requiredButNotProvided
    }
}
```

An empty list response means there are no errors.

## Adding to Basket
Once the field and submission have been submitted, the personalised product can then be added to basket as following:
```graphql
mutation AddPersonalisedProductToBasket {
    addPersonalisedProductToBasket(
        basketId: null
        sku: 13165645
        quantity: 1
        settings: {
            currency: GBP, shippingDestination: GB
        }
        personalisationValues: {
            fieldSubmissionList: [
                { name: "name", value: "Lizzo" }
                { name: "message", value: "its aboout time" }
                { name: "template", value: "Design 4" }
            ]
        }
    ) {
        id
        totalQuantity
        items {
            quantity
            product {
                title
                sku
            }
            personalisationValues {
                name
                value
            }
        }
    }
}
```


Example for a pick 'n mix or `MULTI_SELECTION` configuration:
```graphql
mutation AddPersonalisedProductToBasketMultiSelection {
    addPersonalisedProductToBasket(
        basketId: null
        sku: 14845090
        quantity: 2
        settings: {
            currency: GBP, shippingDestination: GB
        }
        personalisationValues: {
            fieldSubmissionList: [
                {name: "toblerone_mix_tastes", multiSelectionSubmissions: [
                    {value: "13165635", quantity:1},
                    {value: "13165640", quantity:2}
                ]}
                {name: "toblerone_mix_tastes2", multiSelectionSubmissions: [
                    {value: "13165655", quantity:1}
                ]}
            ]
        }
    ) {
        id
        totalQuantity
        items {
            quantity
            product {
                title
                sku
            }
            personalisationValues {
                name
                value
                quantity
            }
        }
    }
}
```

If an invalid submission is provided, a validation error will be thrown and null will be returned for the basket. This should never happen if the submission is validated before being added to basket.

Return example for `SINGLE_SELECTION`:
```json
{
  "data": {
    "addPersonalisedProductToBasket": {
      "id": "2c8f10d0-3146-4210-abf8-b386f723ad58:1702568738380",
      "totalQuantity": 1,
      "items": [
        {
          "quantity": 1,
          "product": {
            "title": "Personalised Original 360g Bar - White",
            "sku": 13165645
          },
          "personalisationValues": [
            {
              "name": "template",
              "value": "hearts"
            },
            {
              "name": "name",
              "value": "Lizzo"
            },
            {
              "name": "message",
              "value": "its aboout time"
            }
          ]
        }
      ]
    }
  }
}
```

NOTE: When we return the `personalisationValues` of a basket item where the field is of type `SINGLE_SELECTION` we will return the `name` of the option as opposed to the `value` so that this can be displayed to the customer.

Return example for `MULTI_SELECTION`: 
```json
{
  "data": {
    "addPersonalisedProductToBasket": {
      "id": "072e8309-c63a-448d-9796-4f321bd472fa:1702564462626",
      "totalQuantity": 2,
      "items": [
        {
          "quantity": 2,
          "product": {
            "title": "6-BAR GIFT PACK",
            "sku": 14845090
          },
          "personalisationValues": [
            {
              "name": "Personalised Original 360g Bar - Fruit & Nut",
              "value": null,
              "quantity": 1
            },
            {
              "name": "Personalised Original 360g Bar - Milk",
              "value": null,
              "quantity": 2
            },
            {
              "name": "Personalised Original 360g Bar - Orange",
              "value": null,
              "quantity": 1
            }
          ]
        }
      ]
    }
  }
}
```
Note: The `value` will be always null in the return `MULTI_SELECTION` response because we don't want to display on basket the sku of the chosen product. The `name` instead is populated from Mars using the title of the selected sku.

