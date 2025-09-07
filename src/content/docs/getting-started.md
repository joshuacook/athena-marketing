---
title: "Getting Started with Athena"
description: "Learn how to set up and start using the Athena AI platform"
section: "Getting Started"
order: 1
draft: false
---

# Getting Started with Athena

Welcome to Athena, the complete AI data platform that unifies structured data, AI context, and intelligent file management.

## Quick Start

Get up and running with Athena in minutes:

### 1. Sign Up and Get Your API Key

1. Visit [Athena Developer Portal](https://athena-developer-portal.vercel.app)
2. Create your account
3. Generate your API key from the dashboard

### 2. Install the SDK

```bash
npm install @athena/sdk
```

### 3. Initialize Athena

```javascript
import { Athena } from '@athena/sdk';

const athena = new Athena({
  apiKey: 'your-api-key-here'
});
```

### 4. Create Your First Context

```javascript
// Store structured data
const user = await athena.data.create('users', {
  id: 'user-123',
  name: 'John Doe',
  email: 'john@example.com'
});

// Add contextual knowledge
const context = await athena.context.create({
  type: 'user-preferences',
  content: 'User prefers dark mode and compact layouts',
  metadata: {
    userId: 'user-123',
    category: 'ui-preferences'
  }
});

// Upload and process files
const file = await athena.files.upload('./document.pdf', {
  context: 'project-documentation',
  extract: true // Automatically extract and index content
});
```

## Next Steps

- [Explore the API Reference](/docs/api-reference)
- [Learn about Context Types](/docs/context-types)
- [Understand Knowledge Graphs](/docs/knowledge-graphs)
- [Build Your First App](/docs/tutorials/first-app)