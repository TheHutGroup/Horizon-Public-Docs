---
layout: default
title: Loyalty
parent: Features
grand_parent: Examples & Concepts
nav_order: 2
---

# Loyalty Overview
Loyalty is a feature that can be enabled on any site. Users will be rewarded points for their purchases as well as completing various actions around the site such as leaving reviews for products they have purchased.
There are 3 types of Loyalty Programmes:

- Multi-Tiered: Customers are rewarded with more account benefits as their points total increases
- Redeemable: Customers can convert their points for set amounts of Account Credit
- Hybrid: A combination of the above where two types of points are tracked, total points for tier benefits, and spendable points for Account credit redemptions

## Ways of earning points
There are two ways for customers to earn points:
 - purchasing products 
 - various interaction on the site: reviews, profile completion, account creation, social media interactions 


## Loyalty Account Dashboard Example
The Loyalty account dashboard query is part of the customer query. It contains information about the customer's points, about their points history, and about the loyalty programme.
All of these queries are available only if the LOYALTY feature flag is enabled.

```graphql
query LoyaltyAccountDashboard {
    customer {
        loyaltyAccountDashboard {
            summary {
                optIn   # Customer opt in status in the loyalty programme
                tierOrder
                totalPoints
                nonSettledPoints
                spendablePoints
                pointsToNextTier
                socialMediaInteractionTypesUsage {      # This object is populated only if there are social media interactions setup for the loyalty programme
                    interactionType     # ENUM InteractionAwardType
                    usage
                }

            }
            pointsBreakdown {
                nonSettledPoints
                pointsPendingInfo {
                    product {
                        title
                        images {
                            thumbnail
                        }
                    }
                    pointsPending
                    earnDate
                    quantity
                }
                pointsHistory {
                    interactionType     # ENUM LoyaltyHistoryInteractionType can be ORDER or MISC
                    orderNumber       
                    miscSubtype     # ENUM LoyaltyMiscInteractionType
                    earnDate
                    earnedPoints
                }
            }
            loyaltyInteractionAwards {
                type    # ENUM InteractionAwardType
                earnablePoints
                socialMediaLink     # This is only populated for social media interactions
            }
            defaultTierPromotionTax     # Points that are automatically deducted when a customer moves from base tier to the 2nd tier
            redemptionRates(currency: GBP) {
                credit {        # Credit obtained when redeeming this redemption rate
                    currency
                    amount
                    displayValue
                }
                pointsRequired      # Points required to redeem this redemption rate
                redemptionRateId
            }
            tiers {
                name
                order       # Tier order or level starting from 1 (base tier)
                threshold   # Number of points needed for reaching the tier
            }
        }
    }
}

enum LoyaltyHistoryInteractionType {
    ORDER
    MISC
}

enum InteractionAwardType {
    REVIEW,
    ACCOUNT_CREATION,
    PROFILE_COMPLETION,
    FACEBOOK,
    INSTAGRAM,
    TIKTOK,
    YOUTUBE,
    TWITTER,
    LINKEDIN
}

enum LoyaltyMiscInteractionType @if(feature: LOYALTY) {
    REVIEW
    ACCOUNT_CREATION
    PROFILE_COMPLETION
    FACEBOOK
    INSTAGRAM
    TIKTOK
    YOUTUBE
    TWITTER
    LINKEDIN
    OTHER

}

  
```

The `defaultTierPromotionTax` is a nullable integer that if provided in the `loyaltyAccountDashboard` it means that an auto-redemption of the customer points will occur when they move from the base tier to the 2nd tier. 
This feature has been introduced so that the customer isn't eligible to see the `loyaltyAccountDashboard` until they are in tier order 2. This feature is only available for the `Hybrid` programmes.

A `Redeemable` Programme has the `tiers` object null.
A `Multi-Tiered` Programme has the `redemptionRates` object null.
A `Hybrid` Programme has both `redemptionRates` and `tiers` objects populated.


## Loyalty Opt In Mutation

A Loyalty Programme can have 3 types of opt ins: AUTO, SOFT, HARD.
An AUTO opt in means that the customer is auto opted-in in the loyalty programme as soon as they create an account.
A SOFT opt in means that customer has to manually opt-in in the loyalty programme otherwise they cannot access their points and rewards. However, all the points transactions are still registered behind the scenes and when they decide to opt in all of their points will be available to them.
A HARD opt in means that customer has to manually opt-in in the loyalty programme and none of their points are registered.


An example of the mutation to opt in a customer into the Loyalty Programme.

```graphql
mutation UpdateLoyaltyOptIn {
  updateLoyaltyOptIn(newValue: True) {
    error
    fieldErrors
    customer
  }
} 
```

## Loyalty Redeem Points Mutation

To redeem points into credit the following mutation should be called with the correct `redemptionId`. 
The `redemptionId` can be retrieved from the `redemptionRates` object from `loyaltyAccountDashboard`.

```graphql
mutation RedeemPoints { 
    redeemPoints(redemptionId: 1) 
}

enum RedemptionRateSubmissionStatus {
"Success, Points were redeemed."
SUCCESS

    "Insufficient balance or invalid request"
    INSUFFICIENT_FUNDS_OR_BAD_REQUEST

    "Redemption rate was not found"
    REDEMPTION_RATE_NOT_FOUND

    "Failed to redeem the points"
    ERROR_REDEEMING
}

```

## Loyalty Customer Interaction with Social Media Links Mutation

If the loyalty programme contains social media interactions, then a set of social media icons should be presented to the customer. Each icon should redirect the customer to the `socialMediaLink` provided in the `loyaltyInteractionAwards` from the `loyaltyAccountDashboard`.
After the customer clicks on the interaction the following mutation should be called with the corresponding `InteractionAwardType`.

```graphql
mutation CustomerInteractionSocialMediaLink { 
    customerInteractionSocialMediaLink(interactionType: FACEBOOK)  
}
```

The `socialMediaInteractionTypesUsage` object from the `loyaltyAccountDashboard.summary` object can be used to determin which social media interaction awards has the customer interacted with by checking the `usage` boolean.

## Loyalty Points on Product & Basket Pages

Loyalty points can be displayed on the product page and is calculated from the product's list price. The query is part of the `productVariant` object.

```graphql
query ProductPage { 
    product(sku: 10530421, strict: false) {
        defaultVariant(options: { currency: GBP, shippingDestination: GB }) {
             earnableLoyaltyPoints(settings: { currency: GBP, shippingDestination: GB }) 
        }
    }
}
```

The basket page can also display the total number of loyalty points currently in the basket.

```graphql
query ViewBasket { 
  basket(
    id: "2298e807-50ee-4120-abfa-3c0f0ef78fca:1611587680727",
    settings: {
      currency:GBP
      shippingDestination: GB
  	}
    acknowledgeMerge: true
  ) {
         earnableLoyaltyPoints
 }
}
```

