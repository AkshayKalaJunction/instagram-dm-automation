# API Request Flow

## Overview

The workflow uses the Meta Graph API to retrieve Instagram conversation activity and send automated responses.

The main request path is:

```text
Instagram Customer
        ↓
Meta Graph API
        ↓
n8n
        ↓
Message Filtering
        ↓
LLM Classification
        ↓
Intent Router
        ↓
Response Branch
        ↓
Meta Instagram Messaging API
        ↓
Customer
```

## 1. Conversation Polling

The Schedule Trigger starts the main workflow every 5 minutes.

The workflow first obtains its polling timestamps and requests the conversation list from the Instagram Graph API.

The conversation request retrieves:

- Conversation participants
- Conversation update timestamps

Only conversations whose activity is newer than the previous polling timestamp are retained.

## 2. Message Retrieval

For each active conversation, the workflow requests the conversation's messages.

The message request retrieves:

- Message text
- Sender information
- Creation timestamp

The workflow then filters the returned messages before sending anything to the classification layer.

## 3. Message Filtering

The filtering stage checks:

- Message timestamp
- Processed message IDs
- Replied message IDs
- Sender ID

Messages already processed or replied to are ignored.

Messages originating from the business account are also excluded so the workflow does not respond to its own messages.

## 4. LLM Classification

A new customer message is combined with the classification prompt and passed to the LLM classification layer.

The classifier is instructed to return one intent.

The implementation attempts providers in a fallback sequence:

```text
Gemini
   ↓
Gemma
   ↓
Groq
   ↓
Mistral
```

The resulting text is normalized against the supported intent names.

## 5. Intent Routing

The normalized intent is passed to the n8n Switch node.

```text
                     ┌── Product_Query
                     │
                     ├── Catalogue
LLM Result ──────────┼── Professional
                     │
                     ├── Appreciation
                     │
                     └── Post_Purchase
```

Each supported intent connects to its corresponding response branch.

## 6. Response Generation

The selected branch assigns a predefined response template.

Before delivery, the workflow applies duplicate-reply protection using the message ID.

## 7. Instagram Response

The response is sent through the Instagram Messaging API using an authenticated HTTP POST request.

The request contains:

```json
{
  "recipient": {
    "id": "<customer_id>"
  },
  "message": {
    "text": "<response>"
  }
}
```

The actual production recipient and authentication values are supplied at runtime and are not stored as secret values in the repository.

## 8. Continue Processing

After the response is sent, the workflow returns to the conversation-processing loop.

This allows subsequent active conversations to be processed during the same execution.

## Complete Flow

```text
Schedule Trigger
      │
      ▼
Polling Timestamp
      │
      ▼
GET Conversations
      │
      ▼
Filter Changed Conversations
      │
      ▼
Process Conversation
      │
      ▼
GET Messages
      │
      ▼
Filter New Customer Messages
      │
      ▼
LLM Classification
      │
      ▼
Intent Router
      │
      ▼
Response Template
      │
      ▼
POST Instagram Message
      │
      ▼
Next Conversation
```

## Authentication Boundary

API authentication is handled through n8n credentials and runtime environment configuration.

The repository contains workflow definitions and sanitized configuration only; production authentication material must be configured in the deployed n8n instance.
