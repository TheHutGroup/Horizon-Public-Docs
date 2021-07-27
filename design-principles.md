---
layout: default
title: Schema Design Principles
nav_order: 2
---

# Schema Design Principles

**Remember that any information included in the schema (even in descriptions) should be considered public. These docs are also public.**

## Recommended reading

[Shopify's GraphQL design tutorial](https://github.com/Shopify/graphql-design-tutorial/blob/master/TUTORIAL.md) makes good reading, though we do some things (e.g. pagination) very differently from their recommendations.

This [article on nullability](https://medium.com/expedia-group-tech/nullability-in-graphql-b8d06fbd8a3c) on Expedia's blog is also well worth reading, as is a lot of the other content on that blog.

## Formatting rules

* All field names should be `camelCased`.
* Enum values should be in `ALL_CAPS_WITH_UNDERSCORES`.
* Types should be `CamelCasedWithALeadingCapital`, a.k.a. [Pascal case](https://en.wikipedia.org/wiki/Camel_case).
* Use descriptions where sensible. Avoid `#` comments as it's not clear how they carry through to the introspected schema.

These rules are, where possible, enforced by schema linting.

## Language

We have standardized the use of British English in this schema.
This is mainly to avoid mixing `color` and `colour` in different places.
When querying the schema, you may use aliases to get `color`, for example:

```graphql
query {
  socialLoginProviders {
    code
    name
    loginUrl
    color:colour
  }
}
```

## Field names

* Timestamps should be named along the lines of `createdAt`, `updatedAt` etc.

## Pagination and filters

Pagination is normally handled as in the example below.

```graphql
type Bar {
   # some data fields
}

type Bars {
    bars: [Bar!]!
    total: Int!
    hasMore: Boolean!
}

input BarFilter {
    barId: ID
    servesFood: Boolean
    servesCaskBeer: Boolean
}

extend type City {
    bars(filter: BarFilter, offset: Int! = 0, limit: Int! = 10): Bars!
}
```

The `offset` parameter must be zero or more (negative values will error) and gives the index of the first record to be included, starting at zero. The `limit` parameter must also be zero or more, though a zero value will not be very useful as no records will be returned, and sets the maximum number of records to be returned. `total` gives the total number of records matching the filter. `hasMore` is true unless the last record returned is the final record available, or there are no records available at all. If `offset` is greater than `total`, no records will be returned but the system will not error.

Filters can be used to narrow down the records returned. All records returned will match the value provided in each non-null filter field, i.e. they are combined with an *and* rather than an *or*. To return records matching different filters, use field aliasing. A null filter parameter, or a non-null filter with all fields null, are equivalent and do not affect the results.

## Sub-queries

Sometimes additional data may be needed from a root query in reaction to the presence of a particular GraphQL type (resolved from a union or interface).  In this case, these types may have a field of type Query to allow further data to be queried.

An example of this would be with Widgets.  The client will want to request Query.socialLinks ONLY when the SocialLinksWidget is present.  Unfortunately, they couldn't normally make that decision efficiently, as they would need to wait for the response to be returned and then make a second GraphQL query.  We can solve that by giving the SocialLinksWidget a field of type Query.  Then the client can just query as follows:

```
widgets {
  ... on SocialLinksWidget {
    query {
      socialLinks {
        name
        url
      }
    }
  }
}
```

These sub-queries can be neatly pulled out as fragments and re-used if needed.

## Potentially Failing Queries

If a query field makes a call to a downstream microservice, and that call may fail, but the field should be Non-null as far as the domain is concerned, we have the issue of a non-null field returning null, and thus causing the response to be inconsistent with the schema.  Instead, if a query makes a call to a microservice or may fail, that field should be nullable to avoid this.

For example:

```graphql
type Query {
  customer: Customer # Nullable because it comes from an Account Service
}

type Customer {
  email: String! # Non-null, because if the account service call for the customer succeeded, 
                 # we already have the email

  creditAccounts: [CreditAccount!] # Nullable as this causes a new call to Account Service which might fail
}

type CreditAccount {
  currency: Currency! # Non-null, because if the account service call for credit accounts succeeded
                      # We already have the currency
}
```

This does leak some of our internal architecture but, as covered in the Expedia blog post from the top of this page, this is realistically unavoidable.
