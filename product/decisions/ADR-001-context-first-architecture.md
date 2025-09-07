# ADR-001: Context-First Architecture

**Status**: Proposed  
**Date**: 2024-01-07  
**Decision Makers**: Josh, Claude

## Context

We are building Athena as a unified platform for AI application development. The current architecture has multiple services (Context, Data, Storage, LLM) that developers must understand and coordinate. This creates cognitive overhead and doesn't reflect how developers think about their applications.

## Decision

**Athena will adopt a Context-First Architecture where:**

1. **Context Types are the primary abstraction** - Every application begins by defining its context types
2. **Services become invisible** - Developers interact with contexts, not services
3. **Everything revolves around contexts** - Data, files, AI, and memory are all capabilities of contexts

## Rationale

### The Flip

Traditional development follows this pattern:
```
Database Schema → Business Logic → Features → AI (bolted on)
```

Context-First development flips this:
```
Context Types → Everything else follows automatically
```

### What is a Context?

A context is any entity that needs:
- **Identity** - It IS something (User, Document, Conversation)
- **Relationships** - It connects to other things
- **Memory** - It can learn and remember
- **Understanding** - It benefits from AI comprehension

### What is NOT a Context?

Simple data that only needs storage:
- Items in a shopping list (the LIST is the context)
- Individual measurements or readings
- Simple key-value pairs

**Rule of thumb**: If it needs relationships, memory, or understanding, it's a context. If it's just data, it belongs to a context.

## Implementation

### Developer Experience

The first thing a developer does is define their context types:

```python
app = athena.Application("recipe-app")

app.define_context_types({
    "user": {
        "schema": {"name": str, "dietary_restrictions": list},
        "capabilities": ["memory", "preferences"]
    },
    "recipe": {
        "parent": "user",  # recipes belong to users
        "schema": {"title": str, "cuisine": str, "difficulty": int},
        "capabilities": ["memory", "search", "files"]
    },
    "cooking_session": {
        "parent": "recipe",
        "schema": {"date": datetime, "rating": int},
        "capabilities": ["memory", "temporal"]
    }
})
```

### Unified Interface

Once contexts are defined, all operations happen through them:

```python
# Not service-oriented:
# ❌ athena.context_service.create(...)
# ❌ athena.data_service.update(...)
# ❌ athena.llm_service.generate(...)

# But context-oriented:
# ✅ Everything through the context
user = User.create(name="Josh")
user.remember("Prefers vegetarian recipes")
user.set_data({"last_login": datetime.now()})

recipe = Recipe.create(title="Pad Thai", parent=user)
recipe.remember("User found this too spicy last time")
recipe.attach_file("photo.jpg")
recipe.generate_variations(constraint="vegetarian")

# Context-constrained operations
similar_recipes = recipe.find_similar(within=user.context)
user_sessions = CookingSession.list(parent=user, since="30d")
```

### Service Mapping

Under the hood, contexts map to services:

| Context Capability | Service | Purpose |
|-------------------|---------|---------|
| `.create()` | Context Service | Create identity & relationships |
| `.remember()` | Context Service | Add understandings |
| `.set_data()` | Data Service | Store structured data |
| `.attach_file()` | Storage Service | Manage files |
| `.generate()` | LLM Service | AI generation |
| `.search()` | Context Service | Vector/semantic search |

But developers never see this mapping.

## Consequences

### Positive

1. **Intuitive Development** - Developers think in terms of their domain, not infrastructure
2. **Automatic Integration** - Services work together automatically through contexts
3. **Progressive Disclosure** - Simple apps stay simple, complexity is opt-in
4. **Natural Boundaries** - Context types create clear architectural boundaries

### Negative

1. **Hidden Complexity** - Service boundaries still exist but are hidden
2. **SDK Dependency** - This experience requires a well-designed SDK (doesn't exist yet)
3. **Migration Path** - Existing service-oriented users need a transition strategy

### Neutral

1. **Opinionated** - This is a strong opinion about how to build AI apps
2. **Context-Centric** - Everything must be modeled as contexts or data-on-contexts

## Example: Hello World Apps

### Better RAG
```python
# Define context for a document Q&A system
app.define_context_types({
    "knowledge_base": {
        "capabilities": ["memory", "search", "files"]
    },
    "query_session": {
        "parent": "knowledge_base",
        "capabilities": ["memory", "generation"]
    }
})

# Build RAG that remembers
kb = KnowledgeBase.create(name="Product Docs")
kb.attach_files("docs/*.pdf")  # Automatic parsing & embedding

session = QuerySession.create(parent=kb)
session.remember("User prefers technical details")

# Context-aware retrieval
answer = session.answer("How do I authenticate?")
# → Automatically retrieves from kb context
# → Remembers this query for future improvements
```

### Chat that Learns
```python
app.define_context_types({
    "user": {
        "capabilities": ["memory", "preferences"]
    },
    "conversation": {
        "parent": "user",
        "capabilities": ["memory", "generation", "temporal"]
    }
})

user = User.create(email="josh@example.com")
chat = Conversation.create(parent=user)

chat.message("I prefer concise answers")
# → Stored as understanding

response = chat.message("Explain Python")
# → Returns concise explanation
# → Remembers this interaction

# Even in a new conversation...
new_chat = Conversation.create(parent=user)
response = new_chat.message("Explain JavaScript")
# → Still returns concise explanation (learned preference)
```

## Status

This ADR is **proposed**. Next steps:
1. Validate with more use cases
2. Design SDK interface in detail
3. Plan migration from service-oriented approach
4. Build proof of concept

## References

- [Product Conversation: Unified Platform for Development](../conversations/2024-01-07-unified-platform-for-development.md)
- [Product Conversation: Data/Context Service Interplay](../conversations/2024-01-07-data-context-service-interplay.md)