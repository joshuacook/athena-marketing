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

1. Go to [API Keys](https://app.athena.radicalsymmetry.com/dashboard/api-keys) in your dashboard
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

## Step 3: GraphQL Endpoint (Single API)

Athena exposes a single GraphQL endpoint:

```
POST https://athena-graphql-6ivigdfuua-uc.a.run.app/graphql
```

## Step 4: Make Your First Request (GraphQL)

Try a simple query to verify auth and connectivity:

```bash
curl -s \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_API_KEY' \
  -d '{"query":"{ requestInfo { requestId hasAuthorization } }"}' \
  https://athena-graphql-6ivigdfuua-uc.a.run.app/graphql | jq
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

- Read the full GraphQL introduction in the Developer Portal:
  - https://athena-developer-portal.vercel.app/docs/graphql/overview
  - https://athena-developer-portal.vercel.app/docs/graphql/api
  - https://athena-developer-portal.vercel.app/docs/graphql/examples
