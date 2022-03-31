---
layout: default
title: Reviews
parent: Features
grand_parent: Examples & Concepts
nav_order: 5
---

# Reviews
Product reviews is something that is a standard feature on THGs platform and comes with features like verified purchases, review moderation (before they are published on site), average rating and number of reviews at each score level.

Reviews can be accessed by querying a product:
```graphql
query ProductReviews {
  product(sku: 10530743, strict: false) {
    sku
    reviews {
      total
      averageScore
      maxScore
      count1Score
      count2Score
      count3Score
      count4Score
      count5Score
      reviews(limit: 10) {
        reviews {
          id
          title
          authorName
          verifiedPurchase
          posted # Date posted
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
          positiveVotes
          negativeVotes
        }
        total
        hasMore
      }
    }
  }
}
```

Reviews can contain ratings for different qualities e.g. overall score, taste, effectiveness which are `RatingReviewElement`s or can be purely written content which is `TextReviewElement`.

## Rating/Reporting a Review
Logged in customers can also rate the helpfulness of a review and can report reviews if they are inappropriate in any way. 

Reviews store the positive and negative votes against them which can be displayed when rendering a review with `positiveVotes` and `negativeVotes`.

To rate or report a review, these authenticated mutations can be used:

```graphql
mutation Report {
  reportReview(input: {
    sku: 10530743
    reviewId: "480081"
  })
}

mutation VoteUp {
  voteReviewPositive(input: {
    sku: 10530743
    reviewId: "480081"
  })
}

mutation VoteDown {
  voteReviewNegative(input: {
    sku: 10530743
    reviewId: "480081"
  })
}
```

## Adding a Review

Similar to registration and other "forms", the data submitted when creating a review can differ by SKU so is dynamic. This reuses the `Form` object from other areas but means before submitting a review, the data that is required needs to be determined.

To do this, the form and fields should be queried for the SKU as below.

```graphql
query ReviewForm {
  product(sku: 11140463, strict: false) {
    reviews {
      newReviewForm {
        fields {
          name
          type
          validators {
            name
            argument
          }
          required
          answerOptions {
            optionKey
            translation
          }
          offerOtherTextInput
          defaultValue
        }
      }
    }
  }
}
```

Once the fields are known, a review can be submitted as below where the field names are the name field above, and the value is a text String (or optionKey if the option is a single selection option):

```graphql
mutation AddReview {
  addReview(
    input: {
      sku: 11140463
      fields: [
        { name: "score", value: "5" }
        { name: "synopsis", value: "Product Tastes Great" }
        { name: "reviewContent", value: "Really enjoyed the product and is definitely worth while" }
        { name: "nickname", value: "James" }
        { name: "ageRange", value: "25-34" }
        { name: "gender", value: "Male" }
      ]
    }
  ) {
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