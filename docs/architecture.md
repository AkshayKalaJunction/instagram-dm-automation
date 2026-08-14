# System Architecture

## Overview

AkshayKala Smart DM is an Instagram DM automation workflow built around self-hosted n8n, the Meta Graph API, and an LLM-based intent classification layer.

The system periodically checks Instagram conversations for new activity, retrieves relevant messages, classifies customer intent, routes the message to the corresponding response branch, and sends the selected response back through Instagram's messaging API.

## High-Level Architecture

```text
┌─────────────────────┐
│     Instagram      │
│    Customer DM     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Meta Graph API    │
│ Conversations/API │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────┐
│      Google Cloud VM        │
│                             │
│  ┌───────────────────────┐  │
│  │        Docker         │  │
│  │                       │  │
│  │  ┌─────────────────┐  │  │
│  │  │ Self-hosted n8n │  │  │
│  │  │                 │  │  │
│  │  │ Fetch & Filter  │  │  │
│  │  │       ↓         │  │  │
│  │  │ LLM Classifier  │  │  │
│  │  │       ↓         │  │  │
│  │  │ Intent Router   │  │  │
│  │  │       ↓         │  │  │
│  │  │ Reply Branches  │  │  │
│  │  └─────────────────┘  │  │
│  └───────────────────────┘  │
└──────────────┬──────────────┘
               │
               ▼
       Meta Messaging API
               │
               ▼
          Customer Reply
```

## Core Components

### Instagram / Meta Graph API

Provides the conversation and message interface used by the workflow to retrieve Instagram DM activity and send responses.

### Google Cloud VM

Hosts the self-hosted automation environment.

### Docker

Runs the n8n service in the deployed environment.

### n8n

Acts as the workflow orchestration layer. It coordinates API requests, filtering, processing logic, LLM classification, routing, response formatting, and message delivery.

### LLM Classification Layer

The workflow sends the customer message to an LLM-based classifier. The implementation contains a fallback sequence across configured providers and normalizes the result into a supported intent.

### Intent Router

The classified intent is routed into the corresponding response branch.

### Response Branches

The workflow contains separate branches for:

- Product Query
- Catalogue
- Professional
- Appreciation
- Post-Purchase

## Processing Model

The main DM workflow uses scheduled polling.

The workflow:

1. Runs on a 5-minute schedule.
2. Records the previous polling timestamp.
3. Retrieves conversations with recent activity.
4. Filters conversations that changed since the previous check.
5. Processes conversations individually.
6. Retrieves messages for the selected conversation.
7. Filters previously processed messages and messages sent by the business account.
8. Classifies the new customer message.
9. Routes the resulting intent.
10. Sends the corresponding response through the Meta messaging endpoint.

## State and Duplicate Protection

Workflow static data is used to retain processing information between executions.

The implementation tracks values including:

- Last polling/check timestamp
- Processed message IDs
- Replied message IDs

This prevents the workflow from repeatedly processing the same customer message or sending duplicate responses.

## Secondary Token Refresh Flow

The exported workflow also contains a separate scheduled access-token refresh chain.

```text
Scheduled Trigger
       ↓
Refresh Instagram Access Token
       ↓
Update n8n Credential
       ↓
Reminder Email
```

This chain is independent of the main 5-minute DM polling path.

## Security Boundary

Production credentials are not stored as plain values in the repository. The workflow references n8n credentials and environment variables for the required authentication material.

The repository contains the workflow definition and sanitized configuration examples; credentials must be configured in the target n8n environment before execution.
