---
title: "API Reference"
description: "Complete technical documentation for all Athena API endpoints"
section: "API Reference"
order: 1
draft: false
---

# API Reference (GraphQL)

Athena exposes a single GraphQL API endpoint. Use your API key to authenticate and send queries/mutations to the gateway.

## Authentication

All API requests must include an Authorization header with a Bearer token:

```
Authorization: Bearer YOUR_API_KEY
```

API keys can be generated from your dashboard. Keys are validated using Unkey.

## Endpoint

```
POST https://<your-graphql-service>/graphql
```

## Response Format

All API responses follow a consistent JSON format:

### Success Response
```json
{
  "data": { ... },      // Response data
  "status": "success"   // Optional status field
}
```

### Error Response
```json
{
  "error": "Error message",
  "detail": "Detailed error information",  // Optional
  "status_code": 400                       // HTTP status code
}
```

## Status Codes

- **200 OK** - Request succeeded
- **201 Created** - Resource created successfully
- **400 Bad Request** - Invalid request data
- **401 Unauthorized** - Missing or invalid API key
- **403 Forbidden** - Insufficient permissions
- **404 Not Found** - Resource not found
- **429 Too Many Requests** - Rate limit exceeded
- **500 Internal Server Error** - Server error

## Rate Limiting

- Default: 60 requests per minute per API key
- Rate limit information is included in response headers:
  - `X-RateLimit-Limit`
  - `X-RateLimit-Remaining`
  - `X-RateLimit-Reset`

## Common Headers

### Request Headers
- `Authorization: Bearer YOUR_API_KEY` (required)
- `Content-Type: application/json` (for POST/PUT requests)
- `Accept: application/json`

### Response Headers
- `Content-Type: application/json`
- `X-Request-ID: unique-request-id`
- `X-Response-Time: milliseconds`

## Error Handling

The API uses standard HTTP status codes and returns detailed error messages:

```json
{
  "error": "Validation failed",
  "detail": "The 'model' field is required",
  "status_code": 400,
  "field_errors": {
    "model": ["This field is required"]
  }
}
```

## Pagination

List endpoints support pagination using query parameters:

- `page` - Page number (default: 1)
- `per_page` - Items per page (default: 20, max: 100)

Response includes pagination metadata:

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 156,
    "pages": 8
  }
}
```

## Full Schema & Examples

- GraphQL schema (SDL), examples, and detailed guides live in the Developer Portal:
  - https://athena-developer-portal.vercel.app/docs/graphql/overview
  - https://athena-developer-portal.vercel.app/docs/graphql/api
  - https://athena-developer-portal.vercel.app/docs/graphql/examples
