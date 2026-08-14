# Deployment

This directory contains the deployment skeleton for running the workflow with self-hosted n8n using Docker.

## Quick Start

1. Provision the target Google Cloud VM.
2. Install Docker and Docker Compose.
3. Copy `.env.example` to a deployment environment file.
4. Set the n8n host/protocol values.
5. Start n8n with:

```bash
docker compose up -d
```

6. Open the n8n instance.
7. Configure production credentials in n8n.
8. Import `workflows/instagram-dm-automation.json`.
9. Test the workflow before activation.

Production secrets must not be committed to Git.

