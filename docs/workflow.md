# Workflow

## Main Workflow

The main Instagram DM automation workflow is organized into four functional batches:

1. Fetch & Filter
2. Classify
3. Reply
4. Continue Processing

The workflow runs on a 5-minute schedule and processes conversations incrementally.

## Batch 1 — Fetch & Filter

The workflow first establishes the polling window using workflow static data.

- Records the previous polling timestamp.
- Records the current polling timestamp.
- Requests the Instagram conversation list through the Meta Graph API.
- Filters conversations whose `updated_time` is newer than the previous polling timestamp.
- Sends active conversations into the conversation-processing loop.

This prevents every execution from repeatedly processing the complete conversation list.

## Batch 2 — Classify

For each active conversation, the workflow retrieves its messages.

The message filtering stage:

- Checks message timestamps.
- Tracks processed message IDs.
- Tracks replied message IDs.
- Excludes messages sent by the business account.
- Keeps genuinely new customer messages.

The remaining message is passed to the classification layer.

### LLM Classification

The workflow constructs a classification prompt containing the supported intent definitions and the customer message.

The LLM family attempts classification through a fallback sequence:

```text
Gemini
  ↓ failure
Gemma
  ↓ failure
Groq
  ↓ failure
Mistral
```

The returned value is normalized against the supported intents.

If the message cannot be confidently mapped to a supported category, the workflow returns `Unknown`.

## Batch 3 — Reply

The intent router sends the classified message to the corresponding response branch.

Supported response branches are:

```text
Product_Query
Catalogue
Professional
Appreciation
Post_Purchase
```

Each branch:

1. Assigns the appropriate response.
2. Applies duplicate-reply protection.
3. Sends the response through the Instagram Messaging API.

### Response Routing

```text
                 ┌── Product Query
                 │
                 ├── Catalogue
LLM Intent ──────┼── Professional
                 │
                 ├── Appreciation
                 │
                 └── Post-Purchase
```

## Batch 4 — Continue Processing

After a response is sent, the workflow returns to the conversation-processing loop.

This allows the workflow to continue through the remaining active conversations in the current execution rather than stopping after the first response.

## Message State Management

The workflow uses n8n workflow static data to maintain processing state.

Tracked values include:

- `lastPollTimestamp`
- `lastChecked`
- `processedMessageIds`
- `repliedMessageIds`

Message IDs are retained in bounded lists to provide duplicate protection across executions.

## Main Processing Flow

```text
Schedule Trigger
      ↓
Get Last Poll Timestamp
      ↓
List Changed Conversations
      ↓
Filter Active Conversations
      ↓
Process Conversations Loop
      ↓
Fetch Conversation Messages
      ↓
Filter Unprocessed Messages
      ↓
Classification Prompt
      ↓
LLM Family
      ↓
Intent Router
      ↓
Response Branch
      ↓
Send Instagram Reply
      ↓
Process Next Conversation
```

## Separate Token Refresh Workflow

The same exported workflow also contains a separate scheduled access-token maintenance chain.

```text
Schedule Trigger
      ↓
Refresh Instagram Access Token
      ↓
Update n8n Credential
      ↓
Send Reminder Email
```

This maintenance chain is separate from the main 5-minute DM-processing workflow.

