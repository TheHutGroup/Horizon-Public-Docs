---
title: GraphQL Examples
---

Here are some sample queries detailing how to perform key e-commerce actions. Extra information around key GraphQL objects will be added in relevant areas. These queries have been aimed at the Myprotein.com website to return similar data to what is required to display that page.

## Account Creation
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

Supported validators can be found [here](/docs/schema/validatorname.doc.html).

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

## Login
Login is much simpler than registration in the fact that only the username and password is required. Currently our platform only supports email addresses as usernames but their are plans to remove this restriction long term.

```graphql
mutation Login {
  login(input: {
    username: "HorizonTestUser1@thehutgroup.com"
    password: "HorizonTestUser1@thehutgroup.com"
  }) {
    newCustomer
    error
    fieldErrors {
      fieldName
      validators
      requiredButNotProvided
      invalidOption
    }
    customer {
      fullName
    }
  }
}
```

## Social Login
On our platform, we also support logging in users using supported SSO providers like Facebook, Google and a few other popular social media companies.

### Viewing Available Providers and Logging In
Providers need to be setup and configured on the THG side. This is because after a successful login/registration, we need to be able to pull data from the provider like the customer's email address.

Once the provider is set up by THG, the following query can be used to view the providers that are currently configured.

```graphql
query SocialProviders {
  socialLoginProviders {
    name
    code
    loginUrl
    iconUrl
    colour
  }
}
```

This response can be used to power the buttons on the login page as it will return the text, the button colouring, the ordering, and the link to be used when the button is pressed.

To login, the `loginUrl` can be used but should have a `returnTo` parameter appended.

Example: https://loginservice.thehut.net/google?site=83&subsite=en&shield=true&returnUrl=https://www.myprotein.com/socialAuthentication.account

This return to URL provided will need to support HTTP POST requests, more on this in the next section.

### Handling Responses
Once the customer has logged in to the social provider, the social provider will actually return to our system first to validate the response. Once validated, we then pass a token to the URL provided in the `returnTo` as a HTTP POST request.

This will be submitted as a form body with a single parameter called `token`.

Once you have received the POST request with the token, this should be submitted to us using the following GraphQL request. This will return the Opaque token as a cookie as with the login and register mutations.

Note: We also support social login being performed by a query instead of a mutation. This is in case you also want to request other data in the same request which is possible in queries but not mutations. 

```graphql
mutation SocialLogin {
  socialLogin(input: {
    socialAuthenticationToken:"oqehb6cryq44j1-64cscpxvwye00jwfipde"    
  }) {
    authenticationResponse {
      error
    }
  }
}

query SocialLogin {
  exchangeSocialLoginToken(input: {
    socialAuthenticationToken:"oqehb6cryq44j1-64cscpxvwye00jwfipde"
  }) {
    authenticationResponse {
        error
    }
  }
}    
```

### Edge Cases
Unfortunately, with social login there are a few edge cases that need to be handled. This includes the following things:

- Mandatory information needed to create an account that is not provided by social providers
- Attempting to perform a social login for an account that already exists

If information is missing, the `AuthenticationResponse` will be `INVALID_DATA` as not all the data required is provided or valid. In this case, the form field should be checked and a form presented for them to provide it.
In this case, the same mutation can be made with the `missingInformation` fields populated in the input and the `socialLoginToken` returned used instead.

If attempting to social login to an account that already exists, and the provider does not perform email verification before allowing social login, the error will be `SOCIAL_LINK_PENDING`. If this is the case, we reveal the name and email of the customer you are attempting to link to as `socialIdentity`. 

If this is the case, a `requestSocialLinkVerificationEmail` mutation should be made with the `socialAuthenticationToken` which will trigger an email verification email to the owner of the account. This can then be redeemed following steps found in the [Social Link section](#social-links).

```graphql
mutation SocialLogin {
  socialLogin(input: {
    socialAuthenticationToken:"oqer41ofdp8q7dpu9he22k-g2n-1pb_yqud"
    missingInformation: {
      phoneNumber: "07989658965"
    }
  }) {
    authenticationResponse {
      error
    }
    form {
      fields {
        name
      }
    }
    socialLoginToken
    socialIdentity {
      email
      fullName
    }
  }
}
```

## Product Page
On our platform, product pages are made up of product content, reviews and recommendations. These can all be accessed easily through the product query.

### Relationships
When it comes to products, we have the concept of a Product, and a Product Variant. On other platforms, these are sometimes referred to as Master/Child or Parent/Child. 

A Product Variant is something that can actually be purchased and many products only have a single variant. For products that have multiple variants, there are variant options that are grouped by the variant key e.g. flavour or size.

What is shown on a product page (and therefore has a distinct URL) and in search/list results is a Product, not a variant. In our tooling, we do have the concept of Linked SKUs where 2 Products can be linked together so they appear together as variants but also have distinct URLs and will appear distinctly in searches and lists. 

Content can be set on either the product or product variant, depending on whether this content should be shared across all variants or update when changing variants on site.

### Content
Product content is any content that can be added to a product that isn't mandatory data like a price, a title, a url and a name. Product content can be in one of these predefined types found [here](/docs/schema/productcontentvalue.doc.html).

As previously stated, this can be set on the Product or the Product Variant so is best to query this for both objects to choose which to show. 
### URLs and Strict flag
When using the product query, a strict flag can be supplied. This flag determines whether the server should handle any redirects based on the product. These could be because you have entered a variant SKU, so we will actually return the master for you. Or if there is a redirect in place for discontinued products for example.

If this is set to true, we will return null if a variant SKU is supplied. 

URLs are currently defined in our product stack and are used to service the website as well as performance marketing integrations like Google Shopping, hence why they are provided in the data.

### Images
Currently, images are returned in 3 possible sizes. In future, this will be changed to return a single image URL where the exact size is customisable based on URL parameters. 

### PAPs - Promotion Aware Products
As well as a promotion being aware of the products it applies to, on our platform, the inverse is also true. This means that for a product, if a promotion (or many) apply to it, the product is aware of this and promotion related information can be surfaced alongside the product.

This includes information like:
- Offer Title
- Offer Message
- Offer Link - A link to other products in this promotion
- Link Text

These are all customised and set up through our Content Management System that allow this information to be displayed beside a product.

When multiple promotions apply to a single product, the offer that provides the best value to the customer is the one that has the information displayed. 

This is seen in the `marketedSpecialOffer` object below.

### Example
```graphql
query ProductPage {
  product(sku: 10530421, strict: false) {
    title
    marketedSpecialOffer {
      title {
        content {
          type
          content
        }
      }
      description {
        content {
          type
          content
        }
      }
      landingPageLink {
        text
        url
      }
    }
    images {
      thumbnail
      largeProduct
      zoom
    }
    url
    options {
      key
      choices {
        optionKey
        key
        colour
        title
      }
    }
    content {
      key
      value {
        ... on ProductContentStringValue {
          stringValue: value
        }
        ... on ProductContentStringListValue {
          stringListValue: value
        }
        ... on ProductContentIntValue {
          intValue: value
        }
        ... on ProductContentIntListValue {
          intListValue: value
        }
        ... on ProductContentRichContentValue {
          richContentValue: value {
            content {
              type
              content
            }
          }
        }
        ... on ProductContentRichContentListValue {
          richContentListValue: value {
            content {
              type
              content
            }
          }
        }
      }
    }
    reviews {
      total
      averageScore
      maxScore
      count1Score
      count2Score
      count3Score
      count4Score
      count5Score
      reviews {
        reviews {
          id
          title
          authorName
          verifiedPurchase
          posted
          elements {
            ... on RatingReviewElement {
              key
              maxScore
              score
            }
            ... on TextReviewElement {
              key
              value
            }
          }
        }
        total
        hasMore
      }
    }
    fbt: recommendations(type: FREQUENTLY_BOUGHT_TOGETHER) {
      sku
      url
      title
      images {
        thumbnail
      }
      defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
        price(currency: GBP, shippingDestination: GB) {
          price {
            currency
            amount
            displayValue
          }
          rrp {
            currency
            amount
            displayValue
          }
        }
      }
    }
    personalised: recommendations(type: PERSONALISED) {
      sku
      url
      title
      images {
        thumbnail
      }
      defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
        price(currency: GBP, shippingDestination: GB) {
          price {
            currency
            amount
            displayValue
          }
          rrp {
            currency
            amount
            displayValue
          }
        }
      }
    }
    variants {
      sku
      title
      images {
        thumbnail
        largeProduct
        zoom
      }
      choices {
        optionKey
        key
        colour
        title
      }
    }
    platform
    sizeGuide {
      content {
        type
        content
      }
    }
    defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
      sku

      choices {
        optionKey
        key
        colour
        title
      }
      title
      inStock
      availabilityMessage
      price(currency: GBP, shippingDestination: GB) {
        price {
          currency
          amount
          displayValue
        }
        rrp {
          currency
          amount
          displayValue
        }
      }
      content {
        key
        value {
          ... on ProductContentStringValue {
            stringValue: value
          }
          ... on ProductContentStringListValue {
            stringListValue: value
          }
          ... on ProductContentIntValue {
            intValue: value
          }
          ... on ProductContentIntListValue {
            intListValue: value
          }
          ... on ProductContentRichContentValue {
            richContentValue: value {
              content {
                type
                content
              }
            }
          }
          ... on ProductContentRichContentListValue {
            richContentListValue: value {
              content {
                type
                content
              }
            }
          }
        }
      }
    }
  }
}
```

## Search
Our search system has 2 variants, Normal and Instant. 
Instant is to be used when the user types into the search bar at the top of the page before they press enter. This search is slightly different to normal as the searched performed is more fuzzy in terms of the data returned and also provides spelling corrections and suggested full search queries.

### Facets
Facets are used to filter down lists of products. The possible values are returned as part of a search/product list query and are then supplied back to filter the list down. 

### Normal

```graphql
query Search {
  search(options: {
    currency: GBP
    shippingDestination: GB
    limit: 30
    offset: 0
    sort: RELEVANCE
    facets: []
  }, query: "Protain" # purposefully spelt wrong to highlight corrections
  ) {
    total
    hasMore
    correctedQuery
    facets {
      ... on RangedFacet {
        facetName
        facetHeader
        options {
          displayName
          from
          to
          matchedProductCount
        }
      }
      ... on SimpleFacet {
        facetName
        facetHeader
        options {
          optionName
          displayName
          matchedProductCount
        }
      }
      ... on SliderFacet {
        facetName
        facetHeader
        minValue
        maxValue
      }
    }
    products {
      url
      title
      images {
        thumbnail
        largeProduct
        zoom
      }
      reviews {
        total
        averageScore
      }
      defaultVariant(options: {
      	currency:GBP,
        shippingDestination: GB,
      }) {
        title
        price(currency: GBP,
          shippingDestination: GB) {
          price {
            currency
            amount
            displayValue
          }
          rrp {
            currency
            amount
            displayValue
          }
        }
      }
    }
  }
}
```

### Instant

```graphql
query InstantSearch {
  instantSearch(
    currency: GBP
    shippingDestination: GB
    limit: 5
    query: "protei" # purposefully spelt wrong to highlight corrections
  ) {
    corrections {
      correction
      highlightedSearchCorrection
    }
    suggestedSearchQueries
    products {
      url
      title
      images {
        thumbnail
        largeProduct
        zoom
      }
      reviews {
        averageScore
      }
      defaultVariant(options: {
      	currency:GBP,
        shippingDestination: GB,
      }) {
        title
        price(currency: GBP,
          shippingDestination: GB) {
          price {
            currency
            amount
            displayValue
          }
          rrp {
            currency
            amount
            displayValue
          }
        }
      }
    }
  }
}
```

## Landing Page
Dynamic pages on site fall into 3 categories. Product pages, Search pages and Listing Pages. Listing pages come in 2 varieties, Landing pages and Product List pages. Both are just pages that display a list of widgets.

Listing pages are built using widgets which take data and display them based on how the Frontend widget is defined. This could be a simple image where the widget is just the image URL and a link or could be more complex like a carousel where there is a carosel widget which has an array of widgets as one of its values, each with images and links.

For carosels where widgets may have child widgets, so that the query doesnt need to become recursive, we flatten the child widgets which can then be matched based on the IDs.

```graphql
widgets {
  ... on MyParentWidgetType {
    banners: {
      id
    }
  }
}
flattenedChildWidgets {
  id
  ... on MyChildWidgetType {
    greeting
  }
}
```

New widgets are named and defined in the THG Tooling then become available to use in both the API and for assigning to pages. 

Landing pages are usually a list of different widgets like the homepage of a website, Product list pages are usually 1 ProductListWidget on the page.

Widgets can be defined on a per site basis depending on what the client supports. For THG built websites, we will support our global list of widgets on all of them.

```graphql
query LandingPage {
  page(path: "/") {
    title
    metaDescription
    metaSearchKeywords
    widgets {
      ... on GlobalPrimaryBanner {
        id
        imageSmall
        imageMedium
        imageLarge
        bannerURL
      }
      ... on GlobalBrandLogos {
        id
        itemOneURL
        itemOneImage
        itemTwoURL
        itemTwoImage
        itemThreeURL
        itemThreeImage
        itemFourURL
        itemFourImage
        itemFiveURL
        itemFiveImage
        itemSixURL
        itemSixImage
      }
      ... on GlobalSectionPeek {
        id
        title
        numberOfProducts
        url
      productList(input: {
        currency: GBP
        shippingDestination: GB
        limit: 30
        offset: 0
        sort: RELEVANCE
        facets: []          
      }) {
        total
        hasMore
        products {
          url
          title
          images {
            thumbnail
            largeProduct
            zoom
          }
          reviews {
            total
            averageScore
          }
          defaultVariant(options: {
            currency:GBP,
            shippingDestination: GB,
          }) {
            title
            price(currency: GBP, 
              shippingDestination: GB) {
              price {
                currency
                amount
                displayValue
              }
              rrp {
                currency
                amount
                displayValue
              }
            }
          }
        }
      }
      }
      ... on GlobalThreeItemEditorial {
        id
        widgetTitle
        itemOneUrl
        itemOneImage
        itemOneTitle
        itemTwoUrl
        itemTwoImage
        itemTwoTitle
        itemThreeUrl
        itemThreeImage
        itemThreeTitle
      }
      ... on GlobalGeneralImageBanner {
        id
        smallImage
        mediumImage
        largeImage
        linkUrl
      }
      ... on GlobalTwoItemEditorial {
        itemOneURL
        itemOneTitle
        itemOneDescription
        itemOneImage
        itemOneCTAText
        itemTwoURL
        itemTwoTitle
        itemTwoDescription
        itemTwoImage
        itemTwoCTAText
      }
    }
  }
}
```

## Product List

Product lists are very similar to search in the sense they display lists of products and can be filtered with facets but are normally pre-built pages. The only difference to a landing page is that they have the ProductListWidget which takes similar parameters to the search query.

```graphql
query ProductList {
  page(path: "/clothing/sale/mens-clothing-sale") {
    title
    metaDescription
    metaSearchKeywords
    widgets {
      ... on ProductListWidget {
        id
        title
        descriptionHtml {
          content {
            type
            content
          }
        }
        productList(input: {
          currency: GBP
          shippingDestination: GB
          limit: 30
          offset: 0
          sort: RELEVANCE
          facets: []          
        }) {
          total
          hasMore
          facets {
            ... on RangedFacet {
              facetName
              facetHeader
              options {
                displayName
                from
                to
                matchedProductCount
              }
            }
            ... on SimpleFacet {
              facetName
              facetHeader
              options {
                optionName
                displayName
                matchedProductCount
              }
            }
            ... on SliderFacet {
              facetName
              facetHeader
              minValue
              maxValue
            }
          }
          products {
            url
            title
            images {
              thumbnail
              largeProduct
              zoom
            }
            reviews {
              total
              averageScore
            }
            defaultVariant(options: {
              currency:GBP,
              shippingDestination: GB,
            }) {
              title
              price(currency: GBP, 
                shippingDestination: GB) {
                price {
                  currency
                  amount
                  displayValue
                }
                rrp {
                  currency
                  amount
                  displayValue
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## Navigation

See also [examples of navigation types](navigation-types.md).

Navigation refers to the header and footer shown on site. These can technically also include widgets. Widgets in navigation basically just mean a widget that appears at the top or bottom of every page below the header or above the footer.

Navigation can be as deep as you would like to support it but most of our existing navigations are only 3 deep.

### Header
```graphql
query Header{
  header {
    navigation {
      topLevel {
        displayName
        link {
          url
          openExternally
          noFollow
          noIndex
        }
        type
        image {
          url
        }
        subNavigation {
          displayName
          link {
            url
            openExternally
            noFollow
            noIndex
          }
          type
          image {
            url
          }
        subNavigation {
            displayName
            link {
              url
              openExternally
              noFollow
              noIndex
            }
            type
            image {
              url
            }
          }
        }
      }
    }
  }
}
```

### Footer
```graphql
query Footer{
  footer {
    navigation {
      topLevel {
        displayName
        link {
          url
          openExternally
          noFollow
          noIndex
        }
        type
        image {
          url
        }
        subNavigation {
          displayName
          link {
            url
            openExternally
            noFollow
            noIndex
          }
          type
          image {
            url
          }
        subNavigation {
            displayName
            link {
              url
              openExternally
              noFollow
              noIndex
            }
            type
            image {
              url
            }
          }
        }
      }
    }
  }
}
```

## Add to Basket

When adding to basket for the first time, the basket ID should be set as null which will generate a basket for you. This should then be provided back each time. Note: The basket id will update with each manipulation to the basket as it contains a last updated timestamp. The basket ID can also change during a login where a basket merge can occur.

Altering the quantity of the basket is a separate mutation that allows you to set the quantity of an item if you know its already there. 

```graphql
mutation AddToBasket {
  addProductToBasket(
    basketId: null
    sku: 10797927
    quantity: 8
    settings: { currency: GBP, shippingDestination: GB }
  ) {
    id
    items {
      product {
        title
      }
      chargePricePerUnit {
        currency
        amount
        displayValue
      }
      quantity
      appliedOffers {
        totalBasketDiscount {
          currency
          amount
          displayValue
        }
        removeable
        message
        info
      }
      freeGift
    }
    appliedOffers {
      totalBasketDiscount {
        currency
        amount
        displayValue
      }
      removeable
      message
      info
    }
    messages {
      type
      message
    }
    selectYourSample {
      id
      title
      message
      currentAmountSpent {
        currency
        amount
        displayValue
      }
      tiers {
        id
        thresholdAmountSpent {
          currency
          amount
          displayValue
        }
        products {
          sku
          title
          images {
            thumbnail
          }
        }
        selectedProducts {
          sku
        }
        maxSelectedProducts
      }
    }
  }
}
```

### Select Your Sample
Select your sample is a feature available on our promotions system. This allows a site to have a promotion that once the criteria is met, the customer can choose a free gift from a range of options. It is also possible to have multiple tiers with different spend thresholds in to incentivise customers to increase the average order values.



```graphql
selectYourSample {
      id
      title
      message
      currentAmountSpent {
        currency
        amount
        displayValue
      }
      tiers {
        id
        thresholdAmountSpent {
          currency
          amount
          displayValue
        }
        products {
          sku
          title
          images {
            thumbnail
          }
        }
        selectedProducts {
          sku
        }
        maxSelectedProducts
      }
    }
```
## View Basket
```graphql
query ViewBasket {
  basket(
    id: "2298e807-50ee-4120-abfa-3c0f0ef78fca:1611587680727",
    settings: {
      currency:GBP
      shippingDestination: GB
  	}
  ) {
    id
    items {
      product {
        title
      }
      chargePricePerUnit { # item price after offers have applied
        currency
        amount
        displayValue
      }
      totalChargePrice { # line price after offers have applied
        currency
        amount
        displayValue
      }
      quantity
      appliedOffers {
        totalBasketDiscount {
          currency
          amount
          displayValue
        }
        removeable # if applied through discount code, it can be removed
        message
        info
      }
      freeGift
    }
    appliedOffers {
      totalBasketDiscount {
        currency
        amount
        displayValue
      }
      removeable
      message
      info
    }
    messages {
      type
      message
    }
  }
}
```

## Checkout Handover

Once the user is finished on the basket, the basket can be handed over the checkout to complete. To do this, the customer must be logged in otherwise you will be presented with an error.

The response of the checkout handover is the checkout token and the URL. To get to check, the url format is ${checkoutUrl}?ct=${token}.

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

## Product Personalisation

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

### Validating Fields
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

### Validating Submissions
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

### Adding to Basket
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

## Customer Services
Built into our API is the ability to contact our customer services team, and see responses from them. This also include uploading attachments to them.

Each conversation is split into a `Discussion` that has a category, a status and a group of messages.

Each discussion can also have an optional order or list of products. This is because our customer services team can be contacted for general issues, issues related to a specific order, or issues related to specific products in an order.

### Message Categories

Message categories are used to help direct the customer's query to the correct CS team and also for analytics purposes to measure for an influx of a specific category.

The available categories can be found here: [here](/docs/schema/discussioncategory.doc.html).

If you are implementing your own client, you can implement as many or as little of these are you please, but we provide the full list of ones that are supported.

### Message Statuses

When submitting a message to our CS team, the message goes through multiple statuses during its lifetime.

- Outstanding: When a customer has sent a query in, but it hasn't yet been picked up by an agent
- In Progress: Moves here once an agent has been assigned to it
- Pending: Manually moved here if an agent is waiting for some other internal team to investigate the issue
- Complete: The agent has responded or if the agent doesn't need to respond e.g. if a customer just messages saying 'thanks'

### Attachments

When contacting customer services, it is possible to submit an attachment along with a message. This can be done by following these steps.

1. Get the upload config which is specific to a customer using the following query:
```graphql
query Config{
  attachmentUploaderConfig {
    uploadUrl
    authToken
  }
}
```
2. Make a request to the site's domain (not this APIs domain) e.g. www.myprotein.com followed by the `uploadUrl` in the response e.g. https://www.myprotein.com/account/documentupload/uploader/upload/38779254
3. The body of the request should have the `authToken` from the above config request as a form parameter called `authToken` and the attachments as a file attachment in the form parameter called `attachment`.
4. The request should look something like the below:
```bash
curl --location --request POST 'https://www.myprotein.com/account/documentupload/uploader/upload/38779254' \
--form 'uploadToken="145D6BC5-E617-454F-BFB2-MUM19BBC160F"' \
--form 'attachment=@"e6ac8e98-5ec7-44db-b303-5c5e48c4710f.jpeg"'
```
```json
{"successful":true,"token":"eTErS1ZBazJGZzNFR25TcGQzVkNxVzFCdUZDeE1yOXJDaXpVbUNweGhrMkhLVERsQjhUMXRKMjVDRHhiYUJUeVBsNUxCdHNhejB1TTV2SlR2YVNxM0E9PQ","mimeType":"image/jpeg","errorCode":0}
```
5. The token in the response should then be send with a new discussion, or a reply to an existing discussion in the `attachmentToken` field of the `AddDiscussionMessageInput`.

### Examples

```graphql
query GetMessages {
  customer {
    discussions {
      discussions {
        id
        # if the discussion is about a specific order and product
        selection {
          selectedOrder {
            orderNumber
          }
          selectedProducts {
            sku
            productVariant {
              title
            }
          }
        }
        category # category chosen during the submission
        status
        createdAt
        updatedAt
        read
        messages {
          messages {
            id
            type # This is QUERY or RESPONSE
            createdAt
            message
          }
          total
          hasMore
        }
      }
      total
      hasMore
    }
  }
}
```

```graphql
mutation CreateDiscussion{
  createDiscussion(input:{
    category: WEBSITE_ISSUES
    message: {
      message: "Hi, I cannot log in on site, please help."
      attachmentToken:"eTErS1ZBazJGZzNFR25TcGQzVkNxVzFCdUZDeE1yOXJDaXpVbUNweGhrMkhLVERsQjhUMXRKMjVDRHhiYUJUeVBsNUxCdHNhejB1TTV2SlR2YVNxM0E9PQ"
    }
  })
}
```

```graphql
mutation MarkDiscussionAsRead {
  markDiscussionMessagesAsRead(input: {
    discussionId: "13183370" # This discussion currently being read
    upToMessageId: "35100000" # The lastest message being read
  })
}
```

```graphql
mutation ReplyToDiscussion {
  replyToDiscussion(discussionId: "13183370", input:{
    message: "Hi, I still cannot log in on site, please help."
    attachmentToken:"eTErS1ZBazJGZzNFR25TcGQzVkNxVzFCdUZDeE1yOXJDaXpVbUNweGhrMkhLVERsQjhUMXRKMjVDRHhiYUJUeVBsNUxCdHNhejB1TTV2SlR2YVNxM0E9PQ"
  })
}
```


## Account Section

To power the account section of a site, we have included some example queries below on how do get certain data for these areas as we as the mutations for common actions. 

### Viewing Order History

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

### Cancelling Orders
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

### Returning Orders
Todo when we work out where this logic should live

### Marketing Preferences
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

### Account Details
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

### Addresses
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

### Payment Cards
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

### Social Links 
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

### Account Credit
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


