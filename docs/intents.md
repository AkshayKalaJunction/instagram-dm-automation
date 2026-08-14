# Intent Classification

## Overview

The workflow classifies new customer Instagram messages into a defined set of intents before routing them to the corresponding response branch.

The classifier is instructed to return exactly one category.

## Supported Intents

### Product Query

`Product_Query`

Handles questions about specific products and pre-purchase information, including:

- Pricing
- Product availability
- Product authenticity
- Product-related requests
- Shipping time for a product not yet ordered
- Requests for the main website/product link

### Catalogue

`Catalogue`

Handles requests for the broader product range or collection, including:

- Full catalogue requests
- Collection requests
- Product recommendations
- Requests for a special or exclusive product list

### Professional

`Professional`

Handles business-related enquiries, including:

- Collaborations
- Partnerships
- Business enquiries
- Press enquiries
- Requests to join the team
- Requests to learn a skill

### Appreciation

`Appreciation`

Handles messages that primarily contain:

- Compliments
- Thanks
- Praise for products or the brand

If the message contains another clear customer request, it should be classified according to that request instead.

### Post-Purchase

`Post_Purchase`

Handles issues relating to an order that has already been placed, including:

- Shipping status
- Delivery delays
- Orders that have not arrived
- Returns
- Order-related questions

### Unknown

`Unknown`

Used when the message does not clearly belong to one of the supported business intents.

Examples include:

- Unrelated questions
- Random conversation
- General small talk without a relevant request
- Trust or legitimacy questions
- Ambiguous messages

## Classification Rule

The workflow uses a conservative classification rule:

> Only classify a message as `Product_Query`, `Catalogue`, `Professional`, `Appreciation`, or `Post_Purchase` when the message is clearly and specifically related to the business, its products, or an existing order.

When the intent is unclear, the classifier should return:

```text
Unknown
```

The purpose of this rule is to avoid guessing the customer's intent and routing unrelated messages into an incorrect automated response branch.

## Routing

After classification, the result is passed to the n8n intent router.

```text
Product_Query  → Product Response
Catalogue      → Catalogue Response
Professional   → Professional Response
Appreciation   → Appreciation Response
Post_Purchase → Post-Purchase Response
Unknown        → No supported intent branch
```

The workflow's router contains explicit branches for the five supported response categories.

