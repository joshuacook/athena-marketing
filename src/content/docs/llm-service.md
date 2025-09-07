---
title: "LLM Service"
description: "Access GPT-5 and other OpenAI models with advanced features"
section: "Services"
order: 1
draft: false
---

# LLM Service

Access GPT-5 and other OpenAI models with advanced features.

## Overview

The Athena LLM Service provides access to state-of-the-art language models with additional features like conversation management, token tracking, and usage analytics.

**Base URL:**
```
https://athena-llm-570639954118.us-central1.run.app
```

## Available Models

### gpt-5-mini
Fast and cost-effective for most tasks
- **Input:** $3.00 / 1M tokens
- **Output:** $12.00 / 1M tokens
- **Best for:** Quick responses, simple queries, high-volume applications

### gpt-5
Most capable model for complex reasoning
- **Input:** $15.00 / 1M tokens
- **Output:** $60.00 / 1M tokens
- **Best for:** Complex analysis, creative writing, advanced reasoning

### gpt-4o
Optimized version of GPT-4
- **Input:** $10.00 / 1M tokens
- **Output:** $30.00 / 1M tokens
- **Best for:** Balanced performance and cost, production applications

## API Endpoints

### Chat Completions

Create a chat completion with one or more messages.

```bash
curl -X POST https://athena-llm-570639954118.us-central1.run.app/api/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Explain quantum computing in simple terms"}
    ],
    "model": "gpt-5-mini",
    "temperature": 0.7,
    "max_tokens": 500
  }'
```

**Parameters:**
- `messages` (required): Array of message objects with `role` and `content`
- `model` (required): Model ID to use
- `temperature` (optional): Sampling temperature (0-2, default: 1)
- `max_tokens` (optional): Maximum tokens to generate
- `top_p` (optional): Nucleus sampling (0-1)
- `frequency_penalty` (optional): Frequency penalty (-2 to 2)
- `presence_penalty` (optional): Presence penalty (-2 to 2)

### Conversation Management

Create and manage multi-turn conversations.

#### Create Conversation
```bash
curl -X POST https://athena-llm-570639954118.us-central1.run.app/api/v1/conversations \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5-mini",
    "system_message": "You are a helpful coding assistant.",
    "metadata": {"project": "my-app", "purpose": "code-review"}
  }'
```

#### Continue Conversation
```bash
curl -X POST https://athena-llm-570639954118.us-central1.run.app/api/v1/conversations/{conversation_id}/messages \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {"role": "user", "content": "Review this Python function"}
  }'
```

## Python SDK Example

```python
import requests

class AthenaLLM:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://athena-llm-570639954118.us-central1.run.app"
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def chat(self, messages, model="gpt-5-mini", **kwargs):
        response = requests.post(
            f"{self.base_url}/api/v1/chat/completions",
            headers=self.headers,
            json={"messages": messages, "model": model, **kwargs}
        )
        return response.json()

# Usage
client = AthenaLLM("your_api_key")
response = client.chat([
    {"role": "user", "content": "Hello!"}
])
print(response["content"])
```

## Usage Tracking

All API calls are tracked for usage analytics:
- Token consumption per request
- Model usage distribution
- Response times and latencies
- Error rates and types

Access your usage dashboard at the [Developer Portal](https://athena-developer-portal.vercel.app/dashboard).

## Best Practices

1. **Model Selection**: Start with `gpt-5-mini` for testing, upgrade to `gpt-5` only when needed
2. **Temperature**: Use lower values (0.3-0.7) for factual tasks, higher (0.8-1.2) for creative tasks
3. **Token Limits**: Set appropriate `max_tokens` to control costs and response length
4. **Error Handling**: Implement exponential backoff for rate limit errors
5. **Conversation Context**: Use conversation endpoints for multi-turn interactions to maintain context

## Rate Limits

- **gpt-5-mini**: 100 requests/minute
- **gpt-5**: 60 requests/minute
- **gpt-4o**: 80 requests/minute

Rate limits are per API key and reset every minute.