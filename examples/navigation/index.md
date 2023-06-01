---
layout: default
title: Navigation
parent: Examples & Concepts
has_children: true
nav_order: 5
---

# Navigation

See also [examples of navigation types](navigation-types.md).

Navigation refers to the header and footer shown on site. These can technically include widgets. Widgets in navigation mean widgets that appear at the top or bottom of every page below the header or above the footer.

Navigation can be as deep as you would like to support but most of our existing navigations are only 3 levels deep.

## Header
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

## Footer
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