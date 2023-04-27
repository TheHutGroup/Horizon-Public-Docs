---
layout: default
title: Customer Services
parent: Customer & Authentication
grand_parent: Examples & Concepts
nav_order: 4
---

# Customer Services

Our API allows you to contact our customer services team and receive responses from them. You can also upload attachments during the conversation.

Each conversation is a "Discussion" that has a category, a status, and a group of messages.
Each discussion can also have an optional order or a list of products.
This is because our customer services team can be contacted for general issues, issues related to a specific order, or issues related to specific products in an order.

## Message Categories

Message categories are used to direct the customer's query to the correct CS team and measure an influx of specific categories for analytics purposes.

Here's the [list](https://api.thehut.net/lfint/en/docs#DiscussionCategory) of available categories. You can implement as many or as few of these as you please. However, we provide the full list of supported categories.

## Message Statuses

When submitting a message to our CS team, the message goes through multiple statuses during its lifetime:

- **Outstanding**: When a customer has sent a query, but an agent hasn't picked it up yet.
- **In Progress**: Once an agent has been assigned to the query.
- **Pending**: Manually moved here if an agent is waiting for some other internal team to investigate the issue.
- **Complete**: The agent has responded or if the agent doesn't need to respond, for example, if a customer just messages saying 'thanks.'

## Attachments

When contacting customer services, you can submit attachments along with a message.  
Here's how to do it:

1. Get the upload configuration that's specific to a customer using the following query:

   ```graphql
   query Config {
     attachmentUploaderConfig {
       uploadUrl
       authToken
     }
   }
   ```

2. Make a request to the site's domain (not this APIs domain) e.g. www.myprotein.com followed by the `uploadUrl` in the response.  
   e.g. https://www.myprotein.com/account/documentupload/uploader/upload/38779254
3. The body of the request should have the `authToken` from the above config request as a form parameter called `authToken` and the attachments as a file attachment in the form parameter called `attachment`.
4. The request should look something like the following:

   ```bash
   curl --location --request POST 'https://www.myprotein.com/account/documentupload/uploader/upload/38779254' \
   --form 'uploadToken="145D6BC5-E617-454F-BFB2-MUM19BBC160F"' \
   --form 'attachment=@"e6ac8e98-5ec7-44db-b303-5c5e48c4710f.jpeg"'
   ```

   The response will be in JSON format, such as:

   ```json
   {
     "successful": true,
     "token": "eTErS1ZBazJGZzNFR25TcGQzVkNxVzFCdUZDeE1yOXJDaXpVbUNweGhrMkhLVERsQjhUMXRKMjVDRHhiYUJUeVBsNUxCdHNhejB1TTV2SlR2YVNxM0E9PQ",
     "mimeType": "image/jpeg",
     "errorCode": 0
   }
   ```

5. The token in the response should then be sent with a new discussion, or as a reply to an existing discussion in the `attachmentToken` field of the `AddDiscussionMessageInput`.

## Examples

```graphql
query GetMessages {
  customer {
    discussions {
      discussions {
        id
        # if the discussion is about a specific order and product
        selection {
          selectedOrder {
            orderNumber
          }
          selectedProducts {
            sku
            productVariant {
              title
            }
          }
        }
        category # category chosen during the submission
        status
        createdAt
        updatedAt
        read
        messages {
          messages {
            id
            type # This is QUERY or RESPONSE
            createdAt
            message
          }
          total
          hasMore
        }
      }
      total
      hasMore
    }
  }
}
```

```graphql
mutation CreateDiscussion {
  createDiscussion(
    input: {
      category: WEBSITE_ISSUES
      message: {
        message: "Hi, I cannot log in on site, please help."
        attachmentToken: "eTErS1ZBazJGZzNFR25TcGQzVkNxVzFCdUZDeE1yOXJDaXpVbUNweGhrMkhLVERsQjhUMXRKMjVDRHhiYUJUeVBsNUxCdHNhejB1TTV2SlR2YVNxM0E9PQ"
      }
    }
  )
}
```

```graphql
mutation MarkDiscussionAsRead {
  markDiscussionMessagesAsRead(
    input: {
      discussionId: "13183370" # This discussion currently being read
      upToMessageId: "35100000" # The lastest message being read
    }
  )
}
```

```graphql
mutation ReplyToDiscussion {
  replyToDiscussion(
    discussionId: "13183370"
    input: {
      message: "Hi, I still cannot log in on site, please help."
      attachmentToken: "eTErS1ZBazJGZzNFR25TcGQzVkNxVzFCdUZDeE1yOXJDaXpVbUNweGhrMkhLVERsQjhUMXRKMjVDRHhiYUJUeVBsNUxCdHNhejB1TTV2SlR2YVNxM0E9PQ"
    }
  )
}
```
