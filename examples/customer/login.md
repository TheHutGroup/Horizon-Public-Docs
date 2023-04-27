---
layout: default
title: Login
parent: Customer & Authentication
grand_parent: Examples & Concepts
nav_order: 5
---

# Login

Login is much simpler than registration in that only the username and password are required. Currently, our platform only supports email addresses as usernames, but there are plans to remove this restriction in the long-term.

```graphql
mutation Login {
  login(
    input: {
      username: "HorizonTestUser1@thehutgroup.com"
      password: "HorizonTestUser1@thehutgroup.com"
    }
  ) {
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

# Social Login

On our platform, we also support logging in users using supported SSO providers like Facebook, Google, and a few other popular social media companies.

## Viewing Available Providers and Logging In

Providers need to be set up and configured on the THG side. This is because, after a successful login/registration, we need to be able to pull data from the provider like the customer's email address.

Once the provider is set up by THG, the following query can be used to view the providers that are currently configured:

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

This response can be used to power the buttons on the login page as it will return the text, the button color, the ordering, and the link to be used when the button is pressed.

To login, the `loginUrl` can be used but should have a `returnTo` parameter appended.

Example: https://loginservice.thehut.net/google?site=83&subsite=en&shield=true&returnUrl=https://www.myprotein.com/socialAuthentication.account

This return to URL provided will need to support HTTP POST requests. More on this in the next section.

## Handling Responses

Once the customer has logged in to the social provider, the social provider will actually return to our system first to validate the response. Once validated, we then pass a token to the URL provided in the `returnTo` as an HTTP POST request.

This will be submitted as a form body with a single parameter called `token`.

Once you have received the POST request with the token, this should be submitted to us using the following GraphQL request. This will return the Opaque token as a cookie, as with the login and register mutations.

Note: We also support social login being performed by a query instead of a mutation. This is in case you also want to request other data in the same request, which is possible in queries but not mutations.

```graphql
mutation SocialLogin {
  socialLogin(
    input: { socialAuthenticationToken: "oqehb6cryq44j1-64cscpxvwye00jwfipde" }
  ) {
    authenticationResponse {
      error
    }
  }
}

query SocialLogin {
  exchangeSocialAuthenticationToken(
    input: { socialAuthenticationToken: "oqehb6cryq44j1-64cscpxvwye00jwfipde" }
  ) {
    authenticationResponse {
      error
    }
  }
}
```

## Edge Cases

Social login has some edge cases that need to be handled, including:

- Mandatory information needed to create an account that is not provided by social providers
- Attempting to perform a social login for an account that already exists

If mandatory information is missing, the `AuthenticationResponse` will be `INVALID_DATA` because not all the data required is provided or valid. In this case, the form field should be checked and a form presented for the user to provide the missing information. To do this, the same mutation can be made with the `missingInformation` fields populated in the input and the `socialLoginToken` returned used instead.

If attempting to social login to an account that already exists, and the provider does not perform email verification before allowing social login, the error will be `SOCIAL_LINK_PENDING`. In this case, the name and email of the customer being linked to will be revealed as `socialIdentity`.

To handle this, a `requestSocialLinkVerificationEmail` mutation should be made with the `socialAuthenticationToken`. This will trigger an email verification email to the owner of the account. The steps to redeem this email can be found in the [Social Link section](/account-data.md#social-links).

Here's an example GraphQL mutation for social login:

```graphql
mutation SocialLogin {
  socialLogin(
    input: {
      socialAuthenticationToken: "oqer41ofdp8q7dpu9he22k-g2n-1pb_yqud"
      missingInformation: { phoneNumber: "07989658965" }
    }
  ) {
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
