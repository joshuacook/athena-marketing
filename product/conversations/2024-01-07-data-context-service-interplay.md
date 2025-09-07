# Data Service vs Context Service Interplay
*Date: 2024-01-07*
*Participants: Josh, Claude*

## Current Situation

We have a Context Service that appears to handle both:
- Structured data (stored in Firestore)
- AI context/understandings (with embeddings)

But Josh mentioned we need a "Data Service" as well.

## The Question

What is the Data Service and how does it differ from what the Context Service already provides?

## Initial Thoughts

### What Context Service Currently Does
- Stores "contexts" (hierarchical containers)
- Stores "understandings" (knowledge with embeddings)
- Both seem to use Firestore as the backend
- Provides vector search capabilities

### What a Data Service Might Provide
- Pure CRUD operations on structured data?
- Traditional database-like operations without AI overhead?
- Schemas and validation?
- Relationships and constraints?

## Context Service Database Schema (Discovered)

The Context Service uses 4 Firestore collections:

1. **`context_kinds`** - Configurable context types
2. **`context_contexts`** - Hierarchical containers for organizing knowledge  
3. **`context_understandings`** - Atomic knowledge statements with embeddings
4. **`context_understanding_contexts`** - Many-to-many relationships

**Key Insight**: The Context Service is purely focused on knowledge management and semantic relationships. There are NO tables for traditional application data (users, products, orders, etc.).

## Josh's Vision - Key Insights

### Current Reality
- **Users ARE contexts** - "User" is a default context kind in Context Service
- **Structured user data** goes in the metadata field of the context
- **Todo items** would likely also be context kinds (they benefit from relationships/hierarchy)

### The Real Distinction
It's NOT "application data vs knowledge" - it's about what benefits from AI/context capabilities:

**Context Service handles:**
- Things that benefit from hierarchy (User → Projects → Tasks)
- Things that need semantic search (find all tasks about "marketing")
- Things with relationships (User context linked to Preference understandings)
- Entities where AI understanding adds value

**Data Service would handle:**
- Pure structured data that doesn't need embeddings
- High-volume transactional data
- Data where exact matching is all you need
- Lists/collections that are just data (grocery list items)

### Key Insight
They share the same Firestore! This opens possibilities:
- Could move structured aspects from Context Service metadata → Data Service
- Data Service could store the "facts", Context Service stores the "understanding"
- Clean separation: structure vs semantics

## Concrete Examples

### Todo App Example
**Context Service stores:**
- User context (with basic profile in metadata)
- Project contexts (hierarchical under User)
- Task contexts (hierarchical under Projects) 
- Understandings: "User prefers morning reminders", "User uses technical language"

**Data Service would store:**
- Task details (title, done, due_date, priority)
- Structured lists without AI needs
- Audit logs, activity history

### Recipe App Example
**Context Service stores:**
- User context with dietary restrictions
- Recipe contexts with relationships
- Understandings: "User is lactose intolerant", "Prefers spicy food"

**Data Service would store:**
- Ingredient quantities (2 cups flour, 1 tsp salt)
- Shopping lists
- Meal calendar data

### The Pattern
- **Context Service**: Identity, relationships, knowledge, preferences
- **Data Service**: Facts, quantities, states, transactions

## Key Design Questions

1. **Separation of Concerns**
   - Should structured data and AI context be in separate services?
   - Or is the power in having them unified?

2. **Use Cases**
   - When would a developer use Data Service vs Context Service?
   - Example: Todo app - todos in Data Service, user preferences in Context Service?

3. **Performance Considerations**
   - Is the Data Service for when you don't need embeddings/AI?
   - Faster, cheaper for pure structured data?

4. **API Design**
   - How do the two services reference each other?
   - Can Context Service "understand" data from Data Service?

## How They Work Together

Since both services share the same Firestore:

1. **Cross-referencing**: A record in Data Service can reference a context_id
   - Todo item → Task context
   - Shopping list → User context

2. **Best of both worlds**:
   - Query Data Service for exact matches: "Get all todos due today"
   - Query Context Service for semantic search: "Find all tasks about the marketing campaign"
   - Join the results using shared IDs

3. **Migration path**: 
   - Start with everything in Context Service metadata
   - Gradually move structured data to Data Service
   - Keep the context_id references

## API Design Implications

```python
# Get user's todos with context
todos = data_service.query("todos", {"user_id": "123", "done": False})
for todo in todos:
    # Get AI understanding about this todo
    context = context_service.get_context(todo.context_id)
    understandings = context_service.search_understandings(
        query="priority and deadlines",
        context_ids=[todo.context_id]
    )
```

## Service Write Permissions (Key Architecture Decision)

### Clear Separation of Write Access

**Context Service can ONLY write to:**
- `context_contexts` (minus the metadata field)
- `context_understandings`
- `context_understanding_contexts`
- `context_kinds`

**Data Service can ONLY write to:**
- The `metadata` field in `context_contexts`
- All collections prefixed with `data_*`

### Implications

1. **Context Service becomes purely about relationships and knowledge**
   - Creates contexts and their hierarchy
   - Creates understandings with embeddings
   - Links understandings to contexts
   - But CANNOT touch the actual data

2. **Data Service owns all structured data**
   - Even the metadata in contexts
   - All new data collections
   - Ensures data consistency

3. **This solves potential conflicts:**
   - No race conditions on metadata updates
   - Clear ownership of data
   - Single source of truth for structured information

### API Examples

```python
# Context Service - ALLOWED
context = context_service.create_context(
    name="John's Shopping List",
    kind="shopping_list",
    parent_id=user_context_id
    # Note: NO metadata here
)

# Context Service - NOT ALLOWED
# This would fail with permission error
context_service.update_context(
    context_id,
    metadata={"items": ["milk", "eggs"]}  # ❌ Cannot write metadata
)

# Data Service - ALLOWED
data_service.update_context_metadata(
    context_id,
    metadata={"items": ["milk", "eggs"]}  # ✅ Data service owns metadata
)
```

## The Core Distinction (Refined)

**Context Service** handles:
- **Identity**: WHAT something is (User, Project, Task)
- **Relationships**: HOW things relate (User owns Project contains Task)  
- **Knowledge**: UNDERSTANDING about things (User prefers mornings, Task is urgent)

**Data Service** handles:
- **Properties**: The actual data values (name="John", due_date="2024-01-15")
- **State**: Current status (done=true, count=5)
- **Transactions**: Changes to data over time

### The "What" vs "What About"

- Context Service: "This IS a User" (identity/essence)
- Data Service: "This user's name is John" (attributes/properties)

- Context Service: "This IS a Task that belongs to Project X" (identity/relationships)
- Data Service: "This task is due tomorrow and is 50% complete" (state/data)

## Next Steps

- Define Data Service API patterns
- Design service authentication to enforce these permissions
- Consider read permissions (both can read everything?)
- Create migration plan for existing metadata in Context Service