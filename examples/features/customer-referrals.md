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

The `referralCode` can then be shared to other customers through social media or manually 

```graphql
mutation SendReferralEmail {
  sendReferralEmail(emailAddresses: ["email1@thehutgroup.com", "email2@thehutgroup.com"])
}
```

## Referee Process