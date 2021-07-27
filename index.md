---
layout: page
title: User's Guide
nav_order: 1
---

# User Guide

Horizon is the GraphQL API powering THG's enterprise e-commerce platform.

This guide assumes a reasonable level of GraphQL knowledge. If you are just starting out with GraphQL then there is an excellent tutorial at [howtographql.com](https://www.howtographql.com/). The [official specification](https://spec.graphql.org/) is also worth reading, and is not too obscure.

## API types

Horizon is designed to be used by both API clients (e.g. mobile apps, scripts) and by browser-based web clients.

These two classes of clients have, in some areas, quite different requirements and are therefore served from different URLs. For the rest of this section, these will be referred to as the **app** API and **web** API.

### Web API

The Web API uses the HTTP `Host` header (i.e. the domain part of the URL) to determine the site for which data is requested. Domains are in the format `https://horizon-api.[full site url]/graphql` e.g. `https://horizon-api.www.myprotein.com/graphql`. Only POST requests are supported. CORS headers are set to allow access from known THG properties.

Request metadata (such as authentication tokens and session IDs) are to be provided as cookies. This decision was taken for two reasons:

* Backwards compatibility: The cookie will be shared with our old e-commerce platform.
* Security: When logging in and being granted an authentication token, this needs to be stored securely with the customer in their browser. The most secure way of doing this, when working with a JS frontend is as a HttpOnly cookie. If it is not stored in this way, any malicious JS code could potentially steal it. As it is, such cookies are invisible to the frontend application.

The cookies currently supported are:

* `Opaque_<site code>_<subsite code>` (e.g. `Opaque_myprotein_en`, used for customer authentication)
* `chumewe_user` Cookie placed on the device which lasts 5 years for tracking and personalisation
* `chumewe_sess` Cookie placed on the device which lasts 4 hours for session tracking and A/B testing

### App API

The App API determines the site based on the path rather than the domain like above. The format is `https://api.thehut.net/[site code]/[subsite code]/graphql` e.g. `https://api.thehut.net/myprotein/en/graphql`. Only POST requests are supported.

When sending data like authentication tokens etc, these are sent as headers with the request and not as cookies. This fits a more API based model to be used by Mobile Apps etc.

The headers currently supported are:

* `Authorization: Opaque <token>` (used for customer authentication)
* `X-Chumewe-User` (optional) - see above
* `X-Chumewe-Session` (optional) - see above
* `X-Forwarded-For` (will be accepted from trusted clients only)

## Feature switching

Features are a way of enabling or disabling entire parts of the backend for different sites, so that each site can have a GraphQL API that fulfils the requirements of that site, without extra unused fields and types.

The list of currently active features on a given site can be queried using the `features` field on the `Query` type.  The full list of possible features can be found in the schema documentation.

The features affect the avaiable schema through GraphQL directives, which are applied to different types, fields and arguments in the master schema found in this document.

## Graceful degradation (flags)

The extensions of each GraphQL response may come with a list of `flags`.  This will be a list of values as defined in the Flag enum, which can be found in the schema documentation.

These flags are used by Horizon to signal information about the server's current status, and the status of the user's session.  Currently, the following flags are supported:

| Flag Name   | Description |
|-------------|-------------|
| `LOGGED_IN` | This flag is present when the client-provided authentication token (Or cookie, for web clients) is valid.  While in this state, the client is able to access any `@authenticated` fields, and will get a non-null customer from the `customer` query field.|
| `REGISTRATION_UNAVAILABLE` | This flag is present if the server is currently unable to fulfil registration requests.  While the server is in this state, the client may want to improve the user journey by offering alternatives to registration, such as guest checkout, where applicable.|

Example:

```json

{
  "data": {
    "check": "OK"
  },
  "extensions": {
    "flags": [
      "LOGGED_IN"
    ]
  }
}
```

Each flag will appear in the response at most once.
## Rate limiting and CAPTCHA

### Concepts

* *Any* mutation or query *may* be rate limited, at the discretion of the API.
* Some mutations / queries are declared to be rate limited in the schema.
  * These are associated with a rate limiting bucket, which is a set of operations that grouped together for rate-limiting purposes, e.g. `AUTHENTICATION`.
  * For some (not all) rate limited operations, the rate limiter may be bypassed if a valid CAPTCHA response is submitted with the request.
  * In this case, CAPTCHA is used as an umbrella term to also cover various "security check" mechanisms. See the CaptchaType enum for the canonical list of supported systems.

### Am I being rate limited?

If so you will receive a response along the lines of the following.

```json
{
  "errors": [
    {
      "message": "Rate limited: Please try again later",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "extensions": {
        "captchaBypassAvailable": [
          {
            "type": "V2_INVISIBLE",
            "siteKey": "6Lf4fiMUAAAAAGRkNt_wJnf79ra2LSdFBlTL-Wcf"
          }
        ],
        "rateLimited": true,
        "rateLimitingBucket": "AUTHENTICATION"
      }
    }
  ],
  "data": {
    "login": null
  },
  "extensions": {
    "ray": "xbPm2eNuEemolweCGcyDHg==",
    "experiments": {},
    "weight": {
      "maxWeight": 1000,
      "weight": 600
    },
    "rateLimitersFiring": [
      {
        "captchaBypassAvailable": [
          {
            "type": "V2_VISIBLE",
            "siteKey": "6Lct1QYUAAAAAFIB-ZaUrFV1YI5fZs0dOL3FgfaY"
          },
          {
            "type": "V2_INVISIBLE",
            "siteKey": "6Lf4fiMUAAAAAGRkNt_wJnf79ra2LSdFBlTL-Wcf"
          }
        ],
        "rateLimitingBucket": "AUTHENTICATION"
      }
    ]
  }
}
```

If the current operation is rate limited, the extensions `rateLimited` and `rateLimitingBucket` will be set on the error for that operation. If CAPTCHA bypass is available then those details will also be supplied here, see below.

If a rate limiter is firing or about to fire (i.e. the next such operation will be rate limited) then `rateLimitersFiring` will be set in the top-level extensions.

### How do I handle CAPTCHA?

Configuration is available for a given site in extensions and errors where applicable, see first code block above.

When wishing to bypass a rate limiter, the CAPTCHA type should be sent as the `X-Captcha-Type` request header, and the response string (from Google, or an attestation system) as the `X-Captcha-Response` header.

If these are valid the operation will work, if invalid you will simply get the same rate limited exception back as if nothing was sent.

Clients that want to gracefully show CAPTCHA before it is needed (i.e. on login, if the next login will be rate limited) should reference `rateLimitersFiring` as above.

## Weight restrictions

Some queries and mutations consume large amounts of server resources, and so there are limits in place as to how "heavy" each inbound request can be. Schema items can have a `@weight` directive attached, if there is no such directive then they are weightless, i.e. zero weight. The total weight for the received query and the maximum allowed weight are given in the extensions section of each response. Requests that exceed the max weight will be rejected.

```json
{
  "extensions": {
    "weight": {
      "weight": 245,
      "maxWeight": 1000
    }
  }
}
```

This system operates entirely independently of rate limiting.

## Pagination and query filters

See the [Schema Design Principles](schema-information/design-principles.md) for an explanation of these fields.

## Examples

* [Example queries and mutations for common e-commerce pages and flows](schema-information/index.md)
* [Examples of navigation types and how they are rendered](schema-information/examples/navigation-types.md)