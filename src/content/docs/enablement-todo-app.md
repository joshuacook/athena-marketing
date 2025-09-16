---
title: "Enablement Plan: Todo App on Athena"
description: "Step-by-step enablement guide for building a Todo application with Athena's GraphQL platform."
section: "Enablement Guides"
order: 1
draft: false
---

# Enablement Plan — Build a Todo App with Athena

This guide converts ticket **TICKET-ENB-004** into a complete enablement path for developers who want to build a production-ready Todo application on Athena. Follow the sections below sequentially to go from zero to a deployed app that uses Athena's GraphQL gateway, handles attachments, and can optionally generate AI summaries for your tasks.

## Audience & Outcomes

**Who:** Frontend or full-stack engineers who are comfortable with TypeScript/React and want to add Athena as the data and AI layer.

**What you'll accomplish:**

- Bootstrap a Next.js application (App Router) and configure environment variables for Athena.
- Authenticate with Athena using Unkey-issued API keys from the Developer Portal.
- Model todo data using the `Row(kind, id, metadata)` pattern with `kind = "todo"` and shape the `SubjectInput` context.
- Execute core GraphQL operations: `createRow`, `patchRow`, and `row`.
- Store and retrieve attachments through signed upload/download URLs.
- (Optional) Call `chatCompletion` to produce AI-generated summaries of todo lists.
- Understand current limitations (no server-side listing) and temporary workarounds.

## Prerequisites

| Requirement | Why it matters | Notes |
| --- | --- | --- |
| Node.js 20+ (or 22 LTS) | Next.js App Router features and fetch API support | Confirm with `node -v` |
| Package manager | Install dependencies | `pnpm` recommended, `npm` or `yarn` also works |
| Athena Developer Portal access | Create and manage API keys | https://app.athena.radicalsymmetry.com |
| API key + organization context | Authenticate to Athena GraphQL | Capture `API Key`, `organizationId`, optional `userId` |
| GraphQL endpoint URL | Target for all queries/mutations | Provided in Developer Portal (e.g., `https://athena-gateway.example.com/graphql`) |
| Optional storage bucket | Where attachments will live | Signed URL mutations produce upload/download URLs for your bucket |

## High-Level Architecture

```
Next.js App Router (frontend + optional route handlers)
  ├─ React Server Actions / Route Handlers (optional proxy)
  │    └─ fetch -> Athena GraphQL Gateway (single endpoint)
  ├─ Client Components (forms, list, detail views)
  │    └─ Uses shared GraphQL client util (fetch wrapper)
  └─ Local state or lightweight index for todo IDs

Athena GraphQL Gateway
  ├─ createRow / patchRow / row mutations & queries
  ├─ createUploadUrl / createDownloadUrl for attachments
  └─ chatCompletion for AI summaries (optional)
```

### Zero-Backend vs. Thin Backend

- **Client-only:** Use browser `fetch` with a user-specific, scoped API key stored in client-side runtime config. Only do this if the key's permissions are narrow and you trust the execution context.
- **Thin proxy (recommended):** Implement Next.js Route Handlers or API routes that inject the API key server-side. This keeps secrets off the client and lets you enforce rate limits or audit logging. The rest of the app remains static/edge friendly.

## Data Model & Subjects

- **Kind:** `todo`
- **Row metadata structure:**

  ```json
  {
    "title": "string",
    "completed": false,
    "description": "optional string",
    "dueDate": "optional ISO date string",
    "attachmentPath": "optional string for file references"
  }
  ```

- **SubjectInput example:**

  ```json
  {
    "organizationId": "org_123",
    "userId": "user_789" // optional but recommended for per-user views
  }
  ```

  - `organizationId` **must** match the org that owns your API key.
  - `userId` can be omitted if the app is purely organizational, but including it enables per-user filtering in future list APIs or when you implement your own index.
  - Treat `SubjectInput` as context metadata passed on every mutation to capture who performed the action.

## Step 0 — Verify Access & Keys

1. In the Developer Portal, create an API key via **Dashboard → API Keys**. The key is issued by Unkey; copy it immediately because it is shown only once.
2. Record the `organizationId` associated with the key. If the portal allows scoping to specific users, also capture the desired `userId`.
3. Test the key with a minimal request:

   ```bash
   curl -s \
     -H 'Content-Type: application/json' \
     -H 'Authorization: Bearer YOUR_API_KEY' \
     -d '{"query":"{ requestInfo { requestId hasAuthorization } }"}' \
     https://your-athena-graphql-url/graphql | jq
   ```

   Successful responses should include `hasAuthorization: true`.

## Step 1 — Bootstrap the Next.js Project

1. Scaffold the project:

   ```bash
   npx create-next-app@latest athena-todo \
     --typescript --app --tailwind --eslint
   cd athena-todo
   ```

2. Install any supporting packages you prefer (e.g., `clsx`, `react-hook-form`).
3. Set up `.env.local` with the required secrets:

   ```bash
   ATHENA_GRAPHQL_URL="https://your-athena-graphql-url/graphql"
   ATHENA_API_KEY="athena_xxx" # keep secret, never commit
   ATHENA_ORG_ID="org_123"
   ATHENA_DEFAULT_USER_ID="user_789" # optional helper for server-side proxies
   ```

4. Update `next.config.js` to expose only **non-secret** variables to the browser via `env` or `publicRuntimeConfig` if you implement a proxy.

## Step 2 — Implement a GraphQL Client Utility

Create a shared fetch helper in `lib/athena-client.ts` (or similar):

```ts
const ATHENA_URL = process.env.ATHENA_GRAPHQL_URL!;
const ATHENA_KEY = process.env.ATHENA_API_KEY!;

export type GraphQLResponse<T> = {
  data?: T;
  errors?: Array<{ message: string; path?: string[]; extensions?: Record<string, unknown> }>;
};

export async function gql<T>(query: string, variables?: Record<string, any>): Promise<GraphQLResponse<T>> {
  const res = await fetch(ATHENA_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${ATHENA_KEY}`,
    },
    body: JSON.stringify({ query, variables }),
    cache: 'no-store',
  });

  if (!res.ok) {
    const text = await res.text();
    throw new Error(`Athena request failed (${res.status}): ${text}`);
  }

  return res.json() as Promise<GraphQLResponse<T>>;
}
```

- For client-side usage, wrap this utility in route handlers or server actions to avoid exposing the API key.
- Centralize error logging here so you can correlate GraphQL errors with UI failures.

## Step 3 — Core GraphQL Operations

### Create a Todo

GraphQL mutation:

```graphql
mutation CreateTodo($metadata: JSON!, $subject: SubjectInput!) {
  createRow(kind: "todo", subject: $subject, metadata: $metadata) {
    id
    kind
    metadata
    createdAt
  }
}
```

Usage example:

```ts
const createTodo = async (input: { title: string; description?: string; dueDate?: string }) => {
  const metadata = { ...input, completed: false };
  const subject = { organizationId: process.env.ATHENA_ORG_ID, userId: currentUserId };

  const { data, errors } = await gql<{ createRow: { id: string; metadata: any } }>(CREATE_TODO, {
    metadata,
    subject,
  });

  if (errors) throw new Error(errors.map(e => e.message).join('\n'));
  return data!.createRow;
};
```

### Update (Patch) a Todo

```graphql
mutation UpdateTodo($id: ID!, $metadata: JSON!, $subject: SubjectInput!) {
  patchRow(kind: "todo", id: $id, subject: $subject, metadata: $metadata) {
    id
    metadata
    updatedAt
  }
}
```

- Use this to toggle `completed`, edit the `title`, or attach files.
- Send **only** the fields that changed; Athena merges them into existing metadata.

### Fetch a Todo by ID

```graphql
query GetTodo($id: ID!) {
  row(kind: "todo", id: $id) {
    id
    metadata
    createdAt
    updatedAt
  }
}
```

- Keep a lightweight cache of IDs locally (see Step 4) so you know which IDs to fetch.
- Handle `row = null` gracefully (missing or unauthorized).

### Error Handling Pattern

Wrap every call in a helper that normalizes GraphQL error envelopes:

```ts
function assertNoErrors<T>(response: GraphQLResponse<T>): T {
  if (response.errors?.length) {
    throw new Error(response.errors.map(e => e.message).join('\n'));
  }
  if (!response.data) {
    throw new Error('Athena response missing data');
  }
  return response.data;
}
```

## Step 4 — Work Around the Missing List API

Athena's current GraphQL schema lacks a `rows` or `listRows` field. Use one (or combine several) of these interim tactics:

1. **Client-maintained index (simplest):** Store created todo IDs in browser `localStorage` or React state. When the session reloads, read the ID list and fetch each todo via `row`.
2. **Edge KV / lightweight backend:** Persist an array of todo IDs per `userId` in an external store (e.g., Vercel KV, Upstash Redis, Supabase). Route handlers maintain this index while Athena remains the source of truth for todo metadata.
3. **Hybrid fallback:** For server-rendered lists, keep a JSON file or database that maps `userId` → `[todoId]` while you wait for the future `rows` query.

Document whichever workaround you pick and encapsulate it so you can swap in the official list query when it ships.

## Step 5 — UI Implementation Checklist

- **Pages/Routes:**
  - `/` — list view that iterates over your todo ID index.
  - `/todos/new` — form to create a todo (title, optional description, due date, attachment upload, AI summary toggle).
  - `/todos/[id]` — detail view with edit controls.

- **Components:**
  - Todo form using controlled inputs and client-side validation.
  - Toggle button to mark complete/incomplete via `patchRow`.
  - Attachment uploader (see Step 6) and preview/download links.
  - Error boundary to display GraphQL error messages elegantly.

- **UX Considerations:**
  - Use optimistic updates when toggling `completed` to keep the UI responsive.
  - Show loading skeletons while fetching `row` data.
  - If AI summaries are enabled, show rate-limit feedback and caching to avoid repeated requests.

## Step 6 — Attachments via Signed URLs

1. Request an upload URL whenever a user attaches a file:

   ```graphql
   mutation GetUploadUrl($path: String!, $contentType: String!) {
     createUploadUrl(path: $path, contentType: $contentType) {
       uploadUrl
       filePath
     }
   }
   ```

   - Construct `path` using your org and todo ID to keep storage structured, e.g., `"org_123/todos/${todoId}/${filename}"`.

2. Perform an HTTP `PUT` to `uploadUrl` with the raw file bytes and `Content-Type` header.
3. Save the returned `filePath` on the todo via `patchRow`:

   ```ts
   await gql(PATCH_TODO, {
     id: todoId,
     subject,
     metadata: { attachmentPath: filePath },
   });
   ```

4. When rendering attachments, request a download URL:

   ```graphql
   mutation GetDownloadUrl($path: String!, $filename: String!) {
     createDownloadUrl(path: $path, filename: $filename) {
       downloadUrl
       expiresAt
     }
   }
   ```

5. Use the signed `downloadUrl` in an `<a>` tag or fetch to stream the file to the browser.

**Security tips:**

- Signed URLs are short-lived; refresh on demand rather than storing them.
- Validate file types and sizes before requesting an upload URL to prevent unnecessary storage consumption.

## Step 7 — Optional AI Summaries with `chatCompletion`

Use Athena's AI gateway to summarize todos or suggest next actions.

```graphql
mutation SummarizeTodos($messages: [LLMMessageInput!]!) {
  chatCompletion(model: "gpt-5-mini", messages: $messages) {
    content
  }
}
```

Recommended prompt structure:

```ts
const messages = [
  { role: 'system', content: 'You summarize todo lists. Keep it under 120 words.' },
  {
    role: 'user',
    content: JSON.stringify({
      todos: todos.map(todo => ({
        title: todo.metadata.title,
        completed: todo.metadata.completed,
        dueDate: todo.metadata.dueDate,
      })),
    }),
  },
];
```

Display the AI response in the UI and cache it alongside the todo list to minimize repeat calls. Be mindful of model quotas and rate limits; surface helpful error messages when the mutation fails.

## Step 8 — Observability & Error Handling

- **Logging:** Wrap the GraphQL client to log latency, operation names, and error codes. Include `todoId`, `userId`, and `orgId` context in logs (without leaking the API key).
- **Retries:** Implement exponential backoff for network failures. Do **not** blindly retry mutations that may have side effects without idempotency (e.g., double create).
- **Rate Limits:** Watch `429` headers and surface "Too Many Requests" guidance in the UI. For background jobs, throttle proactively.
- **Security:** Store secrets in platform-specific secret managers (Vercel Environment Variables, Cloud Run secrets, etc.). Never ship the API key in client bundles unless intentionally scoped and revocable.

## Step 9 — Testing Checklist

| Layer | What to test | Suggested tooling |
| --- | --- | --- |
| GraphQL operations | Unit tests for helper functions that call `gql` | Vitest/Jest with mocked fetch |
| Forms & validation | Required fields, date formatting, optimistic UI | React Testing Library + Playwright |
| Attachment flow | Upload URL retrieval, PUT success, metadata patch | Integration test hitting a staging bucket |
| AI summaries | Happy path and error states | Mock `chatCompletion` responses |
| Deployment checks | Secrets configured, logs visible, key rotation plan | Platform-specific preflight scripts |

Before shipping to production, run through the end-to-end flow in a staging environment using a separate Athena API key.

## Deployment Notes

1. **Build & deploy:**
   - `pnpm build` to create the production bundle.
   - Deploy to Vercel/Netlify or your preferred host.
2. **Environment variables:** Configure `ATHENA_GRAPHQL_URL`, `ATHENA_API_KEY`, and `ATHENA_ORG_ID` in your deployment platform. Use separate keys for staging vs. production.
3. **Monitoring:** Set up alerts for failed GraphQL calls. Track success/error counts per mutation so you can diagnose issues quickly.
4. **Key rotation:** Document the rotation process. Because keys are issued via Unkey, you can create a new key, update secrets, and revoke the old one with minimal downtime.

## Known Limitations & Future Enhancements

- **No list query yet:** Track todo IDs yourself for now. When Athena introduces `rows(kind: String!, filter: JSON, limit: Int)`, migrate your index logic to call it directly.
- **Subject consistency:** Always pass the same `organizationId`/`userId` combination for reads and writes; mismatches will return `null` or authorization errors.
- **Schema validation:** If you extend metadata with new fields, ensure your Context Service schema (if managed separately) allows them; otherwise `createRow`/`patchRow` may fail with validation errors.
- **Attachments storage:** Signed URLs assume your bucket permissions align with Athena's IAM role. Verify bucket policies early.
- **AI cost management:** Cache summaries and offer manual refresh controls to avoid unnecessary spend.

## Quick Reference Summary

- **GraphQL endpoint:** `POST ${ATHENA_GRAPHQL_URL}` with `Authorization: Bearer <API_KEY>`.
- **Core operations:** `createRow(kind: "todo")`, `patchRow(kind: "todo")`, `row(kind: "todo")`.
- **Attachments:** `createUploadUrl`, `createDownloadUrl`, store `filePath` in todo metadata.
- **AI:** `chatCompletion(model: "gpt-5-mini")` with structured `messages` payload.
- **Workaround:** Maintain local/external index of todo IDs until the list API exists.

With these steps, a developer can stand up a fully functional Todo application on Athena, complete with secure data storage, optional attachment support, and AI-powered insights.
