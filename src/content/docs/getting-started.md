---
title: "Getting Started"
description: "Learn how to authenticate and make your first API call to Athena services"
section: "Getting Started"
order: 1
draft: false
---

# Getting Started

Learn how to authenticate and make your first API call to Athena services.

## Step 1: Get Your API Key

First, you'll need an API key to authenticate your requests:

1. Go to [API Keys](https://athena-developer-portal.vercel.app/dashboard/api-keys) in your dashboard
2. Click "Create API Key"
3. Give your key a descriptive name
4. Copy and save your key securely - you won't be able to see it again!

Your API key should look like: `athena_abc123...`

## Step 2: Authentication

Include your API key in the Authorization header of all requests:

```
Authorization: Bearer YOUR_API_KEY
```

Replace `YOUR_API_KEY` with your actual API key from Step 1.

## Step 3: Available Services

Choose a service to connect to:

### LLM Service
Access GPT-5 and other AI models

```
https://athena-llm-570639954118.us-central1.run.app
```

### Context Service
Manage contexts and understandings with vector search

```
https://athena-context-570639954118.us-central1.run.app
```

## Step 4: Make Your First Request

Test your API key with a simple health check:

```bash
curl -X GET https://athena-llm-570639954118.us-central1.run.app/api/v1/models \
  -H "Authorization: Bearer YOUR_API_KEY"
```

If successful, you'll see a list of available models:

```json
{
  "models": [
    {"id": "gpt-5-mini", "name": "GPT-5 Mini"},
    {"id": "gpt-5", "name": "GPT-5"},
    {"id": "gpt-4o", "name": "GPT-4 Optimized"}
  ]
}
```

## Common Issues

### 401 Unauthorized
- Check that your API key is valid
- Ensure you included "Bearer " before your key
- Verify the key hasn't been revoked

### 429 Too Many Requests
- You've exceeded the rate limit (60 req/min)
- Wait a moment before retrying
- Consider implementing exponential backoff

## Next Steps

- [Explore LLM Service](/docs/llm-service) - Access GPT-5 and other models
- [Learn Context Service](/docs/context-service) - Manage knowledge with vector search
- [View API Reference](/docs/api-reference) - Complete endpoint documentation