# Instagram DM Automation

Automated Instagram DM classification and response workflow for AkshayKala, built with **n8n**, the **Meta Graph API**, and **LLM-based intent classification**.

## Overview

Instagram DM Automation is an Instagram DM automation system designed to identify new customer messages, understand their intent, route them to the appropriate response path, and send a relevant reply through Instagram's messaging API.

The workflow is designed around a scheduled polling model and processes conversations incrementally rather than repeatedly processing the complete conversation history.

## Problem Statement

Instagram DMs can contain different types of customer requests, including:

- Product and pricing queries
- Catalogue requests
- Business and collaboration enquiries
- Appreciation and general feedback
- Post-purchase questions

Handling these messages manually can become repetitive and inconsistent. The project addresses this by introducing an automated workflow that filters new customer messages, classifies their intent, and routes them to predefined response branches.

## Solution

The system uses n8n as the workflow orchestration layer.

At a high level:

```text
Instagram
    ↓
Meta Graph API
    ↓
n8n scheduled workflow
    ↓
Fetch & Filter
    ↓
LLM Intent Classification
    ↓
Intent Router
    ↓
Response Formatting
    ↓
Instagram Messaging API
    ↓
Customer
```

## Key Capabilities

- Scheduled Instagram conversation polling
- Detection of conversations with new activity
- Incremental message processing
- Filtering of already-processed messages
- Exclusion of messages sent by the business account
- LLM-based intent classification
- Multiple LLM providers with fallback handling
- Intent-based response routing
- Duplicate-reply protection
- Automated Instagram message delivery
- Separate Instagram access-token refresh workflow
- Scheduled token-refresh reminder email

## Intent Classification

The classifier is instructed to return one of the following categories:

| Intent | Purpose |
|---|---|
| `Product_Query` | Product, pricing, availability, authenticity and related pre-purchase questions |
| `Catalogue` | Full catalogue, collection or recommendation requests |
| `Professional` | Collaboration, partnership, business, press or related enquiries |
| `Appreciation` | Compliments, thanks and positive feedback |
| `Post_Purchase` | Questions about an order already placed |
| `Unknown` | Messages that do not clearly belong to the supported categories |

Only clearly relevant customer messages are routed into the supported intent branches; ambiguous messages are classified as `Unknown`.

## Workflow

The main workflow is organized into three functional stages.

### 1. Fetch & Filter

- Runs on a 5-minute schedule.
- Tracks the previous polling timestamp.
- Fetches conversations through the Meta Graph API.
- Keeps conversations with activity since the previous check.
- Processes conversations individually.

### 2. Classify

- Retrieves messages for the selected conversation.
- Filters previously processed/replied messages.
- Excludes messages sent by the business account.
- Sends new customer messages to the classification layer.
- Uses an LLM to determine the customer's intent.

The classification layer uses a fallback sequence across configured LLM providers so that another provider can be attempted when a preceding provider fails.

### 3. Reply

- Routes the classified intent through the intent router.
- Selects the corresponding response branch.
- Applies duplicate-reply protection.
- Sends the response through Instagram's official messaging endpoint.
- Returns to the conversation-processing loop.

## Deployment

The workflow is intended to run on a self-hosted n8n instance deployed using Docker on a Google Cloud VM.

```text
Google Cloud VM
       ↓
     Docker
       ↓
Self-hosted n8n
       ↓
Instagram DM Workflow
```

The repository includes deployment documentation and example configuration without production credentials.

## Authentication & Security

Credentials are managed through n8n's credential system and environment-based configuration.

The repository does **not** contain production API keys, access tokens, passwords, or other secret credential values.

Before importing and running the workflow, the required Meta and LLM credentials must be configured in the target n8n environment.

## Access Token Refresh

The workflow also contains a separate scheduled token-refresh chain.

```text
Scheduled Token Refresh
        ↓
Meta Access Token Refresh
        ↓
Update n8n Credential
        ↓
Scheduled Reminder
```

The refresh workflow is separate from the main 5-minute DM polling workflow.

## Repository Structure

```text
akshaykala-smart-dm/
│
├── README.md
├── LICENSE
├── .gitignore
├── .env.example
│
├── workflows/
│   └── instagram-dm-automation.json
│
├── docs/
├── deployment/
├── examples/
└── assets/
```

Detailed architecture, workflow, deployment and authentication documentation is maintained under `docs/`.

## Workflow Import

1. Deploy or access the target n8n instance.
2. Configure the required credentials.
3. Import:

```text
workflows/instagram-dm-automation.json
```

4. Review the credential mappings and environment configuration.
5. Verify the Meta Graph API configuration.
6. Test the workflow with controlled Instagram DM inputs.
7. Activate the workflow after validation.

## Demo

The project demonstration is provided separately as part of the project submission.

> Demo link: To be added.

## Project Documentation

The repository documentation covers:

- System architecture
- Workflow stages
- Deployment
- Authentication
- API request flow
- Intent classification

See the `docs/` directory for the detailed project documentation.

---

**Instagram DM Automation**  
Instagram DM Automation — AkshayKala

