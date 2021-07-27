---
layout: default
title: Navigation
parent: Examples & Concepts
nav_order: 7
---

# Navigation

See also [examples of navigation types](navigation-types.md).

Navigation refers to the header and footer shown on site. These can technically also include widgets. Widgets in navigation basically just mean a widget that appears at the top or bottom of every page below the header or above the footer.

Navigation can be as deep as you would like to support it but most of our existing navigations are only 3 deep.

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