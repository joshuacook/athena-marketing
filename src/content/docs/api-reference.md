---
title: "API Reference"
description: "Complete API documentation for the Athena platform"
section: "API Reference"
order: 1
draft: false
---

# API Reference

The Athena API provides a unified interface for managing structured data, AI context, and files.

## Base URL

```
https://api.athena.ai/v1
```

## Authentication

All API requests require authentication using your API key:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  https://api.athena.ai/v1/contexts
```

## Core Endpoints

### Data Management

#### Create Data Entry
```http
POST /data/{collection}
```

Store structured data in a collection.

```json
{
  "id": "unique-id",
  "field1": "value1",
  "field2": "value2"
}
```

#### Query Data
```http
GET /data/{collection}?filter={query}
```

Query structured data with filtering and pagination.

### Context Management

#### Create Context
```http
POST /contexts
```

Create a new context entry with embeddings.

```json
{
  "type": "context-type",
  "content": "The context content to store",
  "metadata": {
    "key": "value"
  }
}
```

#### Search Contexts
```http
POST /contexts/search
```

Semantic search across all contexts.

```json
{
  "query": "search query",
  "limit": 10,
  "filter": {
    "type": "specific-type"
  }
}
```

### File Management

#### Upload File
```http
POST /files/upload
```

Upload and process a file.

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "file=@document.pdf" \
  -F "extract=true" \
  https://api.athena.ai/v1/files/upload
```

### AI Generation

#### Generate Response
```http
POST /generate
```

Generate an AI response using your contexts.

```json
{
  "prompt": "User question or prompt",
  "context": {
    "include": ["context-ids"],
    "search": "additional context search"
  }
}
```

## Rate Limits

- **Free Tier**: 100 requests/minute
- **Pro Tier**: 1,000 requests/minute
- **Enterprise**: Custom limits

## Error Handling

Athena uses standard HTTP response codes:

- `200 OK` - Success
- `400 Bad Request` - Invalid request
- `401 Unauthorized` - Invalid API key
- `404 Not Found` - Resource not found
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error