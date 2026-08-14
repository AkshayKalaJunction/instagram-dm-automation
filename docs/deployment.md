# Deployment

## Deployment Overview

AkshayKala Smart DM is designed to run as a self-hosted n8n workflow deployed on a Google Cloud VM using Docker.

```text
Google Cloud VM
       ↓
     Docker
       ↓
Self-hosted n8n
       ↓
AkshayKala Smart DM Workflow
```

## Deployment Components

### Google Cloud VM

The virtual machine provides the host environment for the self-hosted automation service.

### Docker

Docker is used to run the n8n service in an isolated and reproducible environment.

### n8n

The self-hosted n8n instance executes the Instagram DM workflow and maintains the workflow configuration and execution state.

### External Services

The workflow communicates with external services through authenticated API requests:

- Meta Graph API / Instagram Messaging API
- Configured LLM providers
- SMTP service for the scheduled token-refresh reminder

## Deployment Flow

```text
1. Provision Google Cloud VM
              ↓
2. Install / configure Docker
              ↓
3. Run self-hosted n8n
              ↓
4. Configure n8n credentials
              ↓
5. Import workflow JSON
              ↓
6. Verify API configuration
              ↓
7. Test workflow
              ↓
8. Activate workflow
```

## Docker Deployment

A deployment configuration can be maintained under:

```text
deployment/
├── docker-compose.yml
└── README.md
```

The deployment environment should keep persistent n8n data outside the workflow repository and should not commit production environment files or credentials.

## Environment Configuration

Production secrets must be supplied through the deployment environment or n8n's credential management system.

Example configuration is provided separately as:

```text
.env.example
```

The example file must contain placeholders only.

## n8n Workflow Import

After n8n is available:

1. Open the n8n instance.
2. Import `workflows/instagram-dm-automation.json`.
3. Configure the required credentials in n8n.
4. Verify the Meta Graph API endpoints and account configuration.
5. Verify the LLM provider configuration.
6. Test controlled Instagram messages.
7. Activate the workflow.

## Runtime Flow

Once deployed, the main workflow operates on a 5-minute schedule:

```text
Schedule
   ↓
Fetch changed conversations
   ↓
Filter new customer messages
   ↓
Classify intent
   ↓
Route response
   ↓
Send Instagram reply
```

The separate token-refresh chain runs independently according to its configured schedule.

## Deployment Considerations

- Keep production credentials outside Git.
- Use the sanitized workflow file in the repository.
- Configure credentials directly in the target n8n environment.
- Maintain persistent n8n data independently from the Git repository.
- Test the workflow before enabling automated execution.
