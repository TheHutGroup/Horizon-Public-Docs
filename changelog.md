---
layout: default
title: Breaking Changes
nav_order: 5
---

# Breaking Changes

This page lists any backwards-incompatible changes made to the Horizon schema. 

## Change in registration input fields (Released on 27/10/2021)

The field "lastName" has been renamed as "surname" to make it compatible with the "surname" field returned from the "accountCreationForm" query.

## Change in supersize variant (Released on 23/03/2022)

The supersize variants query has changed. The previous query has been deprecated and will be removed from the schema in 6 months.
. [See here](examples/features/supersize.md)
