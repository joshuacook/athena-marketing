---
title: "Context Service"
description: "Build knowledge graphs with semantic search and structured storage"
section: "Services"
order: 2
draft: false
---

# Context Service (Concepts)

Build knowledge graphs with semantic search and structured storage.

> Note: GraphQL is the primary integration surface. For schema & examples, see the Developer Portal:
> - https://athena-developer-portal.vercel.app/docs/graphql/overview
> - https://athena-developer-portal.vercel.app/docs/graphql/examples

## Overview

The Athena Context Service allows you to create hierarchical contexts, store understandings with automatic embeddings, and perform both keyword and vector searches using Firestore.

**Integration:** Use the GraphQL gateway at a single endpoint instead of service-specific REST URLs.

## Core Concepts

### Contexts
Hierarchical containers that organize your knowledge:
- **Organization** - Top-level entity
- **Team** - Groups within organizations
- **Project** - Specific initiatives
- **Custom Types** - Define your own context types

### Understandings
Knowledge statements with automatic embeddings:
- Stored with semantic meaning
- Searchable via keywords or vectors
- Can belong to multiple contexts
- Include metadata for filtering

## API Endpoints (Legacy)

### Context Management

#### Create Context
```bash
curl -X POST https://athena-context-570639954118.us-central1.run.app/api/v1/contexts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Machine Learning Project",
    "kind": "Project",
    "description": "Context for ML research and development",
    "metadata": {
      "team": "ai-research",
      "status": "active"
    }
  }'
```

#### Create Hierarchical Contexts
```bash
# Create parent context
curl -X POST https://athena-context-570639954118.us-central1.run.app/api/v1/contexts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "AI Organization",
    "kind": "Organization"
  }'

# Create child context
curl -X POST https://athena-context-570639954118.us-central1.run.app/api/v1/contexts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Computer Vision Team",
    "kind": "Team",
    "parent_id": "PARENT_CONTEXT_ID"
  }'
```

### Understanding Management

#### Create Understanding
Automatically generates embeddings for semantic search:

```bash
curl -X POST https://athena-context-570639954118.us-central1.run.app/api/v1/understandings \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "statement": "Convolutional neural networks excel at image recognition tasks",
    "context_ids": ["CONTEXT_ID_1", "CONTEXT_ID_2"],
    "metadata": {
      "source": "research-paper",
      "confidence": "high"
    }
  }'
```

### Search Operations

#### Keyword Search
Traditional text-based search:

```bash
curl -X POST https://athena-context-570639954118.us-central1.run.app/api/v1/search/understandings \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "neural networks",
    "search_type": "keyword",
    "limit": 10
  }'
```

#### Vector Search
Semantic search using Firestore vector capabilities:

```bash
curl -X POST https://athena-context-570639954118.us-central1.run.app/api/v1/search/understandings \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "how do machines learn from images",
    "search_type": "vector",
    "context_ids": ["CONTEXT_ID"],
    "limit": 5
  }'
```

## Python SDK Example

```python
import requests
from typing import List, Optional

class AthenaContext:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://athena-context-570639954118.us-central1.run.app"
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def create_context(self, name: str, kind: str, **kwargs):
        """Create a new context"""
        data = {"name": name, "kind": kind, **kwargs}
        response = requests.post(
            f"{self.base_url}/api/v1/contexts",
            headers=self.headers,
            json=data
        )
        return response.json()
    
    def add_understanding(self, statement: str, context_ids: List[str]):
        """Add an understanding to contexts"""
        response = requests.post(
            f"{self.base_url}/api/v1/understandings",
            headers=self.headers,
            json={
                "statement": statement,
                "context_ids": context_ids
            }
        )
        return response.json()
    
    def search(self, query: str, search_type: str = "vector"):
        """Search understandings"""
        response = requests.post(
            f"{self.base_url}/api/v1/search/understandings",
            headers=self.headers,
            json={
                "query": query,
                "search_type": search_type,
                "limit": 10
            }
        )
        return response.json()

# Usage example
client = AthenaContext("your_api_key")

# Create a project context
project = client.create_context(
    name="AI Assistant", 
    kind="Project",
    description="Building an intelligent assistant"
)

# Add knowledge
understanding = client.add_understanding(
    "LLMs can be fine-tuned for specific tasks",
    [project["context"]["id"]]
)

# Search for relevant knowledge
results = client.search("fine-tuning language models")
```

## Advanced Features

### Context Types
Available context types:
- `Organization` - Top-level entities
- `Team` - Organizational units
- `Project` - Specific initiatives
- `Topic` - Knowledge domains
- `Custom` - User-defined types

### Metadata Schema
Flexible metadata for filtering and organization:

```json
{
  "metadata": {
    "tags": ["machine-learning", "computer-vision"],
    "priority": "high",
    "department": "research",
    "created_by": "user@example.com",
    "custom_field": "custom_value"
  }
}
```

### Search Filters
Filter search results by context or metadata:

```json
{
  "query": "neural networks",
  "search_type": "vector",
  "filters": {
    "context_ids": ["CONTEXT_ID_1", "CONTEXT_ID_2"],
    "metadata": {
      "priority": "high",
      "tags": ["machine-learning"]
    }
  }
}
```

## Best Practices

1. **Context Hierarchy**: Design your context structure before adding understandings
2. **Metadata Standards**: Define consistent metadata schemas for your organization
3. **Search Strategy**: Use keyword search for exact matches, vector search for concepts
4. **Batch Operations**: Group related understandings for better performance
5. **Context Isolation**: Use context IDs to scope searches appropriately

## Performance Considerations

- **Embedding Generation**: Happens automatically, typically takes 100-200ms
- **Vector Search**: Powered by Firestore, scales to millions of documents
- **Keyword Search**: Full-text indexing for fast retrieval
- **Context Limits**: No hard limits, but deep hierarchies may impact performance
