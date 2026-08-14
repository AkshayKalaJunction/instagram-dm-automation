# Authentication & Security

## Overview

The workflow communicates with Meta's Instagram Graph API and configured LLM providers using authenticated requests.

Production credentials are not stored as plain-text secret values in the repository.

## Credential Architecture

```text
                    ┌─────────────────────┐
                    │   n8n Credentials   │
                    └──────────┬──────────┘
                               │
             ┌─────────────────┼─────────────────┐
             ▼                 ▼                 ▼
       Meta / Instagram     LLM APIs          SMTP
       API credentials     API credentials    credentials
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ▼
                        n8n Workflow
```

## Meta / Instagram Authentication

The workflow uses n8n's credential mechanism for authenticated requests to the Instagram Graph API.

The workflow contains credential references rather than production access-token values.

Before execution, the required Meta credential must be configured in the target n8n instance.

## LLM Authentication

The classification code retrieves configured provider keys from the execution environment.

The workflow supports a fallback sequence across configured providers:

```text
Gemini
   ↓
Gemma
   ↓
Groq
   ↓
Mistral
```

Provider keys are expected to be supplied through environment configuration and are not committed to the repository.

## SMTP Authentication

The separate token-refresh maintenance chain uses an n8n SMTP credential for its scheduled reminder email.

SMTP credentials are configured in n8n rather than stored in the repository.

## Repository Security

The repository should never contain:

- API keys
- Instagram access tokens
- Passwords
- SMTP passwords
- Private keys
- Production `.env` files
- n8n credential exports containing secret values

The repository uses sanitized workflow data and example configuration placeholders.

## Deployment Security

Production secrets should be configured directly in the deployed n8n environment.

Recommended separation:

```text
Git Repository
     │
     ├── Workflow definition
     ├── Documentation
     └── Example configuration

Production n8n
     │
     ├── Actual API credentials
     ├── Environment secrets
     └── Persistent workflow data
```

This separation allows the workflow definition to be version-controlled without committing production authentication material.

## Credential Rotation

If an access token or provider credential is rotated, update the corresponding credential in the deployed n8n environment rather than modifying the repository workflow to contain the new secret.

The workflow also includes a separate scheduled Instagram access-token refresh chain.

## Security Verification Before Deployment

Before activating the workflow:

1. Confirm required n8n credentials are configured.
2. Confirm no production secrets are present in the repository.
3. Verify Meta API access with a controlled test.
4. Verify LLM provider configuration.
5. Verify the SMTP credential if the reminder branch is enabled.
6. Review workflow execution logs for authentication failures.

