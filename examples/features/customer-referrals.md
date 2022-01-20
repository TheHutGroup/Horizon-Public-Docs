---
layout: default
title: Customer Referrals
parent: Features
grand_parent: Examples & Concepts
nav_order: 1
---

# Customer Referrals
Customer referrals are where one customer, can refer another customer and both of the customers get a rewards. For the rest of this document we will call these the referrer (person referring a friend) and referee (the friend being referred).


## Referral Process
When the referrer creates an account, they will be automatically generated a referral code, as is the process for all customer. This can be accessed using the following query:

```graphql
query CustomerReferralCode {
    customer {
        referralCode # The code itself to be shared
        referralCount # Number of referee's that have signed up using this referrers code
    }
}
```

The `referralCode` can then be shared to other customers through social media or manually.

We support the option for us to email a list of recipients, which will link them to `/referrals.list?applyCode=${CODE}` on the website. The expected behaviour here is that this page is used to display more information to the referee about the referral program and that the application consuming this data stores this code in the session until the customer goes to the registration screen.

Note: The below mutation is rate limited. 

```graphql
mutation SendReferralEmail {
  sendReferralEmail(emailAddresses: ["email1@thehutgroup.com", "email2@thehutgroup.com"])
}
```

## Referee Process

Once a customer has access to the referral code, they should use this when creating an account in the referral code field. This will link their account to the referrers account so that both of them receive their rewards.

## Rewards

Rewards are configured in the THG offer system and can differ based on the requirements of the site. In general, the referrer receives account credit for any successful order but referee over a specific value e.g. £5 credit if they spend more than £30 and the referee received a discount off their order e.g. 20%. 

These exact reward is not exposed through this API and this is something that should be manually added to the site as marketing information based on how the site is configured. 