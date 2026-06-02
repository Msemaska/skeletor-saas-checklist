# MCP Server

---

## What It Is

An MCP (Model Context Protocol) server exposes your product's capabilities as a first-class integration target for AI agents and LLM clients such as Claude, Cursor, and any MCP-compatible host. Instead of a user clicking through your UI, an AI agent calls your tools directly and reads your resources. The MCP server is not a separate product — it is a thin protocol layer built on top of the existing public API (phase-4-public-api/01-api-foundation.md). It reuses the same authentication, rate limiting, and data access stack already in place.

Build this after your public REST API is stable. MCP is a post-launch channel for expanding the developer surface area of your product to the AI tooling ecosystem.

---

## Transport

Use Streamable HTTP transport exclusively. Deploy the MCP server as a Next.js API route under `app/api/mcp/` running on Railway alongside the rest of the application — no separate deployment. Streamable HTTP is stateless at the protocol layer, works behind Railway's round-robin load balancer without sticky sessions, and is the transport the spec has standardised on. stdio transport is for local development and CLI tooling only — never for production remote servers.

The MCP endpoint lives at a single URL (`/api/mcp`). The MCP TypeScript SDK handles the Streamable HTTP framing, method routing, and the `Mcp-Method` / `Mcp-Name` headers the spec now requires on every request. Do not implement this transport layer manually.

---

## Authentication

Authenticate MCP requests using the same API keys that authenticate the REST API. The MCP client passes the key as a Bearer token in the `Authorization` header — identical to the REST API contract. Unkey validates the key on every request before any tool handler executes. The workspace scope extracted from the key determines what data all tools in that session can access. There is no separate MCP credential system.

For clients that use the MCP OAuth 2.1 flow (the spec defines this for hosted consumer integrations), delegate entirely to Better Auth. The OAuth flow issues a scoped access token that is then passed as the Bearer token — the same Unkey validation applies from that point forward. Do not implement a custom OAuth server for MCP; Better Auth's OAuth provider handles PKCE, token issuance, and revocation.

Key scopes work exactly as they do for the REST API. If a key is read-only, all tools that write data return a 403. Define scopes on the key at creation time, not at the tool invocation level.

---

## Tool Design

A tool is a function an AI agent can call. Tools should map directly to actions a user would take in the product — not to database operations. Name tools as agent-legible verbs: `list_projects`, `create_task`, `get_invoice`, `archive_member`. Snake case. No abbreviations. The tool name is the contract — rename only with a deprecation cycle.

Every tool must include a clear description field. The description is read by the LLM to decide whether to call the tool. A vague description produces wrong calls; an accurate description produces correct calls. Write descriptions from the agent's perspective: what does calling this tool accomplish?

Set tool annotations on every tool:
- `readOnlyHint: true` on all read tools (list, get, search). Only set this if the handler genuinely has no side effects — updating a `last_accessed_at` timestamp disqualifies it.
- `destructiveHint: true` on delete and archive tools.
- `idempotentHint: true` on update tools where calling twice with the same arguments produces the same result.

These hints are not enforced by the server — they are used by MCP hosts to decide whether to prompt the user for confirmation. Missing or wrong annotations are the leading cause of rejection from AI platform directories. Set them correctly from the start.

Expose only tools that are safe and meaningful for an AI agent to call autonomously. Do not expose admin tools, billing mutations, or anything that cannot be reversed or is scoped to internal operations.

---

## Resource Design

A resource is a piece of data an AI agent can read, addressable by URI. Resources complement tools — tools act, resources inform. Expose resources for the core entities in your product using a consistent URI scheme:

```
your-product://workspace/{workspaceId}/projects
your-product://workspace/{workspaceId}/project/{projectId}
your-product://workspace/{workspaceId}/members
```

Resources are read-only by definition. Every resource URI is scoped to the authenticated workspace — agents cannot construct a URI that crosses tenant boundaries.

Keep resource payloads focused. Return the fields an agent needs to reason about the entity. Do not return full database rows with internal fields, foreign keys, or audit metadata. If the full shape is needed, the agent can call the REST API directly.

---

## Rate Limiting

Apply the same Upstash sliding window rate limit used by the REST API (phase-4-public-api/01-api-foundation.md). Rate limiting applies per API key, not per tool invocation. This means an agent calling ten tools in rapid succession consumes ten requests from the same window as REST API calls on the same key. AI agents are bursty by nature — a single LLM reasoning step may invoke three or four tools in a few seconds. Set a separate, higher per-minute limit for MCP keys if the 100 req/min REST default is too tight for agent workflows. Document the limit clearly in the MCP integration guide so developers can design agents accordingly.

When a rate limit is hit, the MCP server returns an error result with a message instructing the agent to wait and retry. The agent's host application surfaces this to the user. Include the seconds-until-reset in the error message text — MCP tool error responses do not have standard retry headers.

---

## Relationship to the Public REST API

The MCP server calls the same service functions that the REST API calls. It does not query the database directly and does not bypass RLS. The layering is:

```
MCP Client
    │
    ▼
/api/mcp  (MCP TypeScript SDK, Streamable HTTP)
    │  auth: Unkey key validation
    │  rate limit: Upstash (shared quota with REST API)
    ▼
Service functions (services/)
    │
    ▼
Neon Postgres (RLS enforced)
```

This means the MCP server inherits the REST API's security model, data access patterns, and error handling for free. Do not duplicate logic from the service layer into tool handlers. If a service function does not exist yet, create it — do not put the query inside the tool handler.

---

## Versioning

The MCP server does not use path-based versioning like the REST API. The MCP protocol itself is versioned via the SDK and the `MCP-Version` header negotiation that the SDK handles automatically. Within your tool set, use the same breaking-change rules as the REST API: adding tools and adding optional parameters to existing tools is non-breaking; removing or renaming a tool is breaking. When a tool must be renamed or removed, keep the old name operational and return a deprecation notice in the tool's response for a minimum of 90 days before removal.

---

## Security Considerations

- Every tool invocation is authenticated via the same Unkey API key validation used by the REST API — no request reaches a tool handler without a valid, workspace-scoped key
- All data returned by tools and resources is scoped to the authenticated workspace — cross-tenant access is impossible at the database level via RLS
- Destructive tools are annotated with `destructiveHint: true` so MCP hosts can surface user confirmation prompts before execution
- Tool input schemas are validated with Zod inside the MCP SDK registration — invalid inputs are rejected before the handler runs
- The MCP endpoint is exposed over HTTPS only (Railway enforces this) — no plaintext transport
- Tool handlers must not accept arbitrary strings as identifiers without validating that the referenced resource belongs to the authenticated workspace — an agent can be prompted with a resource ID from a different tenant
- Never expose internal IDs, service account keys, or infrastructure details in tool descriptions, resource URIs, or error messages

---

## Deliverables / Implementation Checklist

**Foundation**
- [ ] MCP TypeScript SDK installed (`@modelcontextprotocol/sdk`)
- [ ] MCP server instance created using `McpServer` high-level API
- [ ] Streamable HTTP transport configured at `app/api/mcp/route.ts`
- [ ] Unkey key validation wired into the MCP request handler before any tool executes
- [ ] Upstash rate limiting applied per API key on every MCP request
- [ ] Workspace scope extracted from validated key and passed to all tool handlers

**Tools**
- [ ] Tool list defined (map to existing service functions — no new DB logic in handlers)
- [ ] Every tool has a clear, agent-legible description
- [ ] Every tool has correct annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`)
- [ ] Every tool input schema defined and validated via Zod
- [ ] Read tools return paginated results with a cursor when result sets can be large
- [ ] Destructive tools include a `confirm` boolean parameter that defaults to `false` and returns a dry-run summary when `false`

**Resources**
- [ ] Resource URI scheme defined and documented
- [ ] Core product entities exposed as resources
- [ ] Resource payloads stripped of internal fields

**Integration**
- [ ] MCP server tested end-to-end with Claude Desktop using a real API key
- [ ] MCP integration guide written: endpoint URL, auth method, tool list, rate limits
- [ ] Tool and resource list published in developer docs

---

## What to Test

- Valid API key allows tool invocation; missing or revoked key returns an auth error before any tool executes
- Rate limit applies per key; exceeding it returns a clear retry message with seconds-until-reset
- Each tool handler returns the correct result shape for a valid request
- Each tool handler returns a structured error (not an unhandled exception) for invalid input
- Tool input schema rejects missing required fields and invalid types
- Read tools respect workspace scope — a key for workspace A cannot retrieve workspace B's data
- Destructive tool called with `confirm: false` returns a dry-run summary, not a mutation
- Destructive tool called with `confirm: true` executes and returns the result
- Resource URIs return the correct entity for the authenticated workspace
- Resource URI with a foreign workspace ID returns a 404 or permission error
- Tool annotations match actual tool behaviour (`readOnlyHint: true` tool produces no side effects)
- Large list results are paginated and do not exceed the context-safe payload limit
- Tool invocations use the service layer, not direct database queries (verified by testing the service function independently)
- The MCP server connects successfully with Claude Desktop and all tools appear in the tool list

---

## Edge Cases

- **Tool returns a massive result set** — a list tool called without filters on a large workspace returns thousands of items, blowing the LLM context window. Enforce a hard cap of 50 items per tool response by default; expose a `limit` parameter capped at 100 and return a `next_cursor` for pagination identical to the REST API pattern.

- **Rate limit hit mid-session** — an agent invokes several tools in sequence and hits the rate limit partway through a multi-step task. Return a structured error result with the retry-after duration in the message text so the agent's host can surface it to the user and pause gracefully rather than crashing the session.

- **Destructive tool called without confirmation** — an agent autonomously calls `delete_project` without a user prompt. The tool must default `confirm: false`, return a dry-run description of what would be deleted, and require the agent to call again with `confirm: true` — making accidental deletion a two-step action the LLM must explicitly reason through.

- **API key scope does not permit a requested tool** — a read-only key attempts to call a write tool. The key validation and scope check must happen before the handler executes, returning a clear permission error rather than a partial execution or ambiguous failure.

- **Agent passes a resource ID from a different workspace** — an agent has been primed with data from a different context and constructs a tool call referencing a foreign workspace's entity ID. The service layer must validate that the ID belongs to the authenticated workspace; returning a generic 404 is correct — do not confirm the resource exists in another workspace.

- **Tool annotation mismatch causes host to skip confirmation** — a tool labelled `readOnlyHint: true` has a side effect (e.g., marks a notification as read). The host skips the confirmation prompt because it trusts the annotation. Audit annotations against actual handler behaviour before publishing. When in doubt, annotate conservatively.

- **MCP client does not send `Mcp-Method` / `Mcp-Name` headers** — older clients pre-dating the current spec do not send these headers. The SDK handles this gracefully — do not write custom header-parsing logic that rejects requests without them.

- **Same API key used for both REST and MCP** — rate limit quota is shared. An agent in a long session can exhaust the key's request window mid-session, breaking both the MCP session and any concurrent REST integrations on the same key. Document this clearly; recommend developers create dedicated MCP keys with elevated limits.

---

## Common Mistakes

- Querying the database directly inside tool handlers instead of calling service functions (bypasses RLS, duplicates logic, makes the tool untestable in isolation)
- Exposing every REST endpoint as a tool (agents become confused by large tool lists — expose only the 10–15 tools that cover the core workflows)
- Writing vague tool descriptions ("manages projects") instead of agent-legible action descriptions ("lists all projects in the workspace with their current status and owner")
- Omitting tool annotations entirely (MCP hosts assume the worst — destructive, non-idempotent, open-world — and prompt the user for confirmation on every tool call)
- Setting `readOnlyHint: true` on tools that have any side effect, including audit logging, timestamp updates, or notification marking
- Not capping list tool results (unbounded results blow the LLM context window and produce truncated or hallucinated reasoning)
- Building a separate auth system for MCP instead of reusing Unkey and the existing API key infrastructure
- Not testing with a real MCP client before publishing (tool descriptions that seem clear in isolation are ambiguous to an LLM in context)
- Treating tool renames as non-breaking (any agent prompt that references the old tool name by description will invoke the wrong tool or fail silently)
- Putting the MCP server on a separate Railway service instead of co-locating it with the API (doubles infrastructure cost for no benefit — Streamable HTTP works on the same service)
