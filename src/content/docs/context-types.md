---
title: "Understanding Context Types"
description: "Learn how to design and use context types effectively in Athena"
section: "Guides"
order: 1
draft: false
---

# Understanding Context Types

Context types are the foundation of Athena's knowledge management system. They allow you to structure and organize contextual information in a way that's both flexible and searchable.

## What are Context Types?

Context types define the schema and behavior for different kinds of contextual knowledge in your application. Think of them as templates that ensure consistency while maintaining flexibility.

## Built-in Context Types

Athena provides several built-in context types:

### user-context
For storing user-specific information, preferences, and history.

```yaml
type: user-context
schema:
  userId: string
  category: string
  importance: number
searchable: true
embeddable: true
```

### document-context
For document summaries, key points, and metadata.

```yaml
type: document-context
schema:
  documentId: string
  section: string
  tags: array
searchable: true
relations:
  - user-context
  - project-context
```

### conversation-context
For chat histories and conversation summaries.

```yaml
type: conversation-context
schema:
  sessionId: string
  participants: array
  timestamp: datetime
retention: 30d
```

## Creating Custom Context Types

Define custom context types to match your application's needs:

```javascript
const customType = await athena.contextTypes.create({
  name: 'meeting-notes',
  description: 'Context from team meetings',
  schema: {
    meetingId: { type: 'string', required: true },
    attendees: { type: 'array', items: 'string' },
    actionItems: { type: 'array', items: 'object' },
    decisions: { type: 'array', items: 'string' },
    nextSteps: { type: 'string' }
  },
  searchable: true,
  embeddable: true,
  relations: ['user-context', 'project-context']
});
```

## Best Practices

### 1. Design for Searchability
Structure your context types to maximize the effectiveness of semantic search:

```javascript
// Good: Descriptive, searchable content
{
  type: 'feature-request',
  content: 'User wants ability to export data as CSV for financial reporting',
  metadata: {
    feature: 'data-export',
    format: 'csv',
    useCase: 'financial-reporting'
  }
}

// Avoid: Vague, unsearchable content
{
  type: 'note',
  content: 'Export feature',
  metadata: {
    id: '12345'
  }
}
```

### 2. Establish Relationships
Connect related contexts to build a knowledge graph:

```javascript
// Create related contexts
const project = await athena.context.create({
  type: 'project-context',
  content: 'Q4 2024 Product Roadmap'
});

const feature = await athena.context.create({
  type: 'feature-context',
  content: 'AI-powered search functionality',
  metadata: {
    projectId: project.id
  }
});

// Link contexts
await athena.context.link(project.id, feature.id, 'contains');
```

### 3. Use Consistent Schemas
Maintain consistency within each context type while leveraging flexibility across types:

```javascript
// Define once, use everywhere
const PRIORITY_LEVELS = ['low', 'medium', 'high', 'critical'];

const taskContextType = {
  name: 'task-context',
  schema: {
    priority: { 
      type: 'string', 
      enum: PRIORITY_LEVELS 
    },
    // ... other fields
  }
};
```

## Context Type Patterns

### Hierarchical Contexts
Build parent-child relationships:

```javascript
// Organization -> Team -> User
const orgContext = { type: 'org-context', content: 'ACME Corp policies' };
const teamContext = { 
  type: 'team-context', 
  content: 'Engineering team standards',
  metadata: { parentOrg: 'ACME Corp' }
};
```

### Temporal Contexts
Track context over time:

```javascript
const timeContext = {
  type: 'status-update',
  content: 'Project on track for Q4 delivery',
  metadata: {
    timestamp: new Date().toISOString(),
    validUntil: '2024-12-31'
  }
};
```

### Composite Contexts
Combine multiple aspects:

```javascript
const compositeContext = {
  type: 'decision-context',
  content: 'Chose PostgreSQL for better JSON support',
  metadata: {
    decision: 'database-selection',
    factors: ['json-support', 'performance', 'cost'],
    alternatives: ['MongoDB', 'DynamoDB'],
    madeBy: 'tech-team',
    date: '2024-01-15'
  }
};
```

## Next Steps

- [API Reference](/docs/api-reference) - See context endpoints
- [Knowledge Graphs](/docs/knowledge-graphs) - Build connected knowledge
- [Search & Retrieval](/docs/search) - Query your contexts effectively