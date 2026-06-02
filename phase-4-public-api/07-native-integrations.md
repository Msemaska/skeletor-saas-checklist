# Native Integrations

---

## What It Is

Native integrations are pre-built connectors that let your users link your product to the tools they already use — Slack, HubSpot, Google Sheets, Notion — without writing a single line of code. Instead of exposing only an API and leaving the wiring to developers, you publish your app to integration platforms (Zapier, Make, n8n) so any user can connect your product in minutes through a visual workflow builder. This dramatically expands your product's reach: every Zap template and Make scenario becomes a discovery surface, and every supported platform is a distribution channel you would otherwise have to build yourself.

Native integrations are built on top of the public API (phase-4/01) and webhook system (phase-4/03). They are not separate infrastructure — they are a packaging layer that exposes your existing API capabilities to non-technical users through platform-native UIs.

## Architecture

Your integration platforms sit outside your infrastructure entirely. When a Zapier trigger fires or a Make scenario runs, the platform calls your public API using OAuth tokens or API keys that the user granted during connection setup. You do not run any integration-platform SDK inside your own server. Your side of the contract is:

- A public REST API (phase-4/01) that the platform can call to read data and perform actions
- Webhook events (phase-4/03) delivered via Svix that the platform can subscribe to as trigger sources
- An OAuth 2.0 application registered per platform so users authenticate without sharing their API keys

The integration platform handles scheduling, retries, error logging, and user notifications. You handle API correctness, webhook delivery, and OAuth token validation.

## Triggers vs. Actions

Every integration is composed of triggers and actions.

**Triggers** start a workflow when something happens in your product. Map these directly to your existing webhook event types — `project.created`, `task.updated`, `member.added`, and so on. Always prefer webhook-based (instant) triggers over polling triggers. Polling triggers on Zapier check for new data every 1–15 minutes and require a deduplicated list endpoint that returns recent items in reverse chronological order. Webhook triggers fire within seconds and require no polling endpoint — the platform registers a URL with your webhook system and waits. Start with webhook triggers wherever the event type already exists in phase-4/03. Only build polling triggers for event types that do not map cleanly to a single webhook event.

**Actions** let users perform operations in your product from another tool's workflow — creating a record, updating a field, sending a notification. Map actions directly to your existing POST/PUT/PATCH API endpoints. Actions must be idempotent where possible; use the idempotency key support built into phase-4/01 to prevent duplicate records when the integration platform retries a failed action.

Keep the initial trigger and action set small: cover the five or six most common automation patterns your users ask for, not every endpoint. You can expand after launch.

## OAuth App Setup

Register one OAuth 2.0 application per integration platform — one app for Zapier, one for Make, one for n8n (if self-hosted users request it). Each platform gets its own `client_id` and `client_secret`.

Use the OAuth 2.0 Authorization Code flow. Request only the scopes the integration actually needs — never request admin or billing scopes for a general-purpose integration. Define scopes at the resource-action level: `projects:read`, `tasks:write`, `members:read`. Map these scope strings to your existing API key permission model.

When a user authorises your app on the integration platform, the platform exchanges the code for an access token and refresh token and stores them on its side. You never see the refresh token again after the initial exchange — the platform handles renewal. Your side validates the access token on every API call exactly as it validates API keys in phase-4/01, enforcing workspace scope and rate limits identically.

OAuth token metadata — workspace ID, granted scopes, expiry — is stored in your database and associated with the user's workspace. When a workspace is deleted or a user revokes access in your product's connected-apps settings, invalidate the OAuth token server-side immediately. This is separate from the integration platform's own revocation.

## Integration Platform Registration and Review

Each platform has a developer portal where you register your app, define triggers and actions, configure OAuth, and submit for review before your integration is publicly listed.

**Zapier** requires a minimum of 10 published Zap templates and 50 active users before a public listing is approved. The 50-user threshold can be waived if you embed the Zapier integration widget inside your product's UI. Build the embed — it removes the gating requirement and puts integration setup inside your product where users already are. Zapier's review process checks that every trigger and action works end-to-end, that error messages are human-readable, and that OAuth scopes match what is documented.

**Make** (formerly Integromat) reviews apps submitted via their developer hub. The process is similar but tends to move faster. Make's user base skews toward power users who build complex multi-step scenarios; this makes Make the right investment for products with richer data models where users want branching logic and data transformation.

**n8n** uses a community node model — you publish a Node.js package to npm and n8n users install it. There is no central approval gate for self-hosted instances. For n8n Cloud, community nodes with external dependencies require n8n team approval. n8n's audience is technical teams and developers who self-host for data security reasons; treat it as a lower-priority investment unless your users are developers or the product handles sensitive data that precludes cloud automation platforms.

## Embedded Integration UI

Do not rely solely on users discovering your app on Zapier's marketplace. Embed the Zapier partner widget or the Make OAuth connect flow directly inside your product's integration settings page. This means users authorise and create workflows without leaving your product, which improves conversion and allows Zapier to waive the 50-user public listing requirement.

The integration settings page in your product should show: connected platforms, connection status (active / token invalid / revoked), the scopes granted, and a disconnect button that revokes the OAuth token.

## Rate Limiting Integration Traffic

Integration platform calls are indistinguishable from direct API calls at the transport layer. A busy Zap running every minute for every one of your users can generate significant API traffic. Rate limits from phase-4/01 apply in full. The 100 requests/minute-per-key limit is per OAuth token — if one user's Zap hits the limit, only that user's automation is throttled. The integration platform receives the 429 response and backs off according to your `Retry-After` header. Document the rate limits in your integration's help text so users with high-frequency Zaps can contact you for elevated limits rather than assuming the integration is broken.

## Versioning and Deprecation

Integration platforms cache your trigger/action schemas. When you rename a field, remove an endpoint, or change a response shape, existing Zaps and scenarios break silently for users who never update them. Treat integration schema changes the same as API breaking changes (phase-4/01): only make changes when a new API version is live and the old version is still running. When the old API version is deprecated, surface a warning in the integration platform's developer portal so affected users are prompted to reconnect or update their Zap.

---

## Providers

| Platform | Audience | Trigger Model | Auth | Review Process | Distribution |
|---|---|---|---|---|---|
| **Zapier** | Non-technical business users | Webhook (instant) + polling | OAuth 2.0 | Formal review, 10 templates + 50 users | Zapier marketplace + embeddable widget |
| **Make** | Power users, ops teams | Webhook (instant) + polling | OAuth 2.0 | Formal review, faster turnaround | Make app directory |
| **n8n** | Developers, self-hosters | Webhook (instant) + polling | OAuth 2.0 or API key | npm publish (no gate for self-hosted) | npm registry + n8n community |

Lead with Zapier for reach — its marketplace has the largest installed base of non-technical users and the embeddable widget removes the listing barrier. Add Make for power users who want complex multi-step automation. Add n8n only if your users are developers or explicitly request it; it is a lower-effort build but a smaller audience.

---

## Security Considerations

- Register OAuth applications with the narrowest possible scopes — never request permissions the integration does not actively use; scope creep expands the blast radius if a token is compromised on the platform's side
- OAuth tokens stored on the integration platform (Zapier, Make) are outside your security perimeter — treat them as long-lived credentials and ensure your token revocation endpoint works instantly; a user who disconnects their Zapier account in your product must have their token invalidated server-side immediately
- The integration platform does not sign its outbound requests to your API — your API validates the OAuth bearer token on every request, exactly as it does for API keys in phase-4/01; there is no additional request signature to verify on inbound integration traffic
- Rate limits in phase-4/01 apply to integration-platform traffic — a misconfigured Zap that loops and fires thousands of requests per minute is still rate-limited per token; never whitelist integration platform IP ranges from rate limiting
- Scopes granted during OAuth authorisation must be enforced server-side — the platform requesting a `projects:read` scope must only be able to read projects, even if it constructs API calls that would normally allow writes; enforce scopes at the middleware layer the same way workspace isolation is enforced
- Audit log every OAuth authorisation and revocation against the workspace so workspace admins can see which integration platforms have access and when they were connected

---

## Deliverables / Implementation Checklist

**OAuth App Registration**
- [ ] OAuth 2.0 application registered in Zapier developer portal (client_id, client_secret stored in environment variables)
- [ ] OAuth 2.0 application registered in Make developer hub
- [ ] OAuth scopes defined at resource-action granularity and mapped to API permission model
- [ ] OAuth token metadata (workspace_id, granted scopes, expiry, platform) stored in Neon database
- [ ] Token revocation endpoint built — invalidates token server-side immediately
- [ ] Connected-apps page in workspace settings (lists active integrations, scopes, disconnect button)

**Triggers**
- [ ] Webhook-based triggers defined for all major event types from phase-4/03 (project.created, task.updated, etc.)
- [ ] Webhook trigger registration endpoint — stores platform-provided URL as a Svix webhook endpoint scoped to the workspace
- [ ] Polling trigger fallback endpoints built where event type does not map to a webhook (returns deduplicated list, reverse chronological, last 100 items)
- [ ] Trigger schemas registered in Zapier developer portal and Make developer hub

**Actions**
- [ ] Actions mapped to existing POST/PUT/PATCH endpoints from phase-4/01
- [ ] Idempotency key forwarding — pass Zapier/Make's request ID as the `Idempotency-Key` header where applicable
- [ ] Human-readable error messages returned on 4xx responses (integration platforms surface these to users)
- [ ] Action schemas registered in Zapier developer portal and Make developer hub

**Publishing**
- [ ] 10 Zap templates created and published covering the most common automation patterns
- [ ] Zapier partner widget embedded in product's integration settings page
- [ ] Zapier integration submitted for review
- [ ] Make app submitted for review
- [ ] Integration help documentation written (scopes, trigger list, action list, rate limits)

**Ongoing**
- [ ] Version compatibility monitored — integration schema pinned to current API version; deprecated API version triggers integration schema update
- [ ] Rate limit headers documented in integration help text
- [ ] Webhook subscription cleanup — when a workspace is deleted, all platform-registered webhook endpoints are removed from Svix

---

## What to Test

- OAuth authorisation flow completes and stores token metadata correctly for each platform
- OAuth token is invalidated server-side when user disconnects from the connected-apps page
- Webhook trigger fires within seconds of a resource event being dispatched via Svix (phase-4/03)
- Webhook trigger registration stores the platform URL as a Svix endpoint scoped to the correct workspace
- Polling trigger endpoint returns results in reverse chronological order with no duplicates
- Action endpoints return human-readable error messages on 4xx (not raw Zod error objects)
- Idempotency key passed by the integration platform prevents duplicate record creation on retry
- Rate limit (100 req/min) applies to OAuth token traffic and returns 429 with Retry-After
- Scope enforcement — an OAuth token granted `projects:read` cannot call a tasks write endpoint
- Workspace deletion triggers cleanup of all Svix endpoints registered by integration platforms
- Revoking OAuth token server-side causes the next integration platform API call to return 401
- Polling trigger deduplicated list does not return the same item twice across consecutive calls
- Connected-apps page accurately reflects which platforms have active tokens and their granted scopes

---

## Edge Cases

- **User revokes access on your side but the Zap still runs** — the OAuth token is invalidated in your database; the next API call from Zapier returns 401, which Zapier surfaces as a connection error to the user and pauses the Zap; no action needed beyond instant server-side revocation.
- **Trigger fires before related data is consistent** — a `task.created` webhook fires immediately but a consumer Zap fetches the task via API while a linked project record is still being written in the same transaction; use database transactions to ensure all related records are committed before dispatching the Svix event (phase-4/03), never dispatch from within an open transaction.
- **Rate limits hit by a busy Zap** — a Zap running every minute for a large workspace generates 1,440 API calls per day from one token; the 429 with `Retry-After` causes Zapier to back off and retry; document the rate limit in integration help text and provide a support contact for users who legitimately need a higher limit rather than leaving them to debug a "failing" Zap.
- **Integration uses a deprecated API version** — the Zapier integration schema is pinned to `/api/v1/` endpoints; when v1 is deprecated, existing Zaps calling v1 break silently; add the `Deprecation` response header to v1 routes (phase-4/01) and notify registered integration platform apps via their developer portals before the removal deadline.
- **Duplicate actions from retries** — Make retries a failed action after a network timeout; without idempotency, this creates two records; pass the integration platform's request ID as the `Idempotency-Key` header on all action requests so the second attempt replays the stored response without re-executing the handler (phase-4/01).
- **Webhook subscription left orphaned after workspace deletion** — the platform-registered Svix endpoint remains active and receives events addressed to a deleted workspace; delete all Svix endpoints for a workspace when the workspace is deleted, using the same cleanup logic described in phase-4/03.

---

## Common Mistakes

- Building polling triggers for events that already have a webhook equivalent — polling is slower, consumes more API quota, and produces a worse user experience
- Creating a separate auth system for integration platforms instead of reusing the OAuth layer on top of the existing API key model in phase-4/01
- Requesting broad OAuth scopes (admin, billing) when the integration only needs read access to two resource types
- Not embedding the Zapier widget in-product and then failing to reach the 50-user threshold for public listing
- Returning raw validation error objects (Zod messages) as action error responses — integration platforms surface these to non-technical users verbatim
- Not passing integration platform request IDs as idempotency keys, leading to duplicate records when a scenario retries
- Registering OAuth applications with the same client credentials across platforms — each platform needs its own application so tokens can be revoked per platform
- Neglecting to clean up webhook subscriptions when a workspace is deleted, leaving orphaned endpoints that receive and discard events indefinitely
- Assuming the integration platform verifies webhook delivery — Svix handles delivery to the platform, but you must still confirm the platform's registered URL accepted the payload
- Launching on n8n first because there is no review gate — n8n's audience is the smallest and most technical; Zapier delivers user reach, which is the primary goal of native integrations
- Not documenting rate limits in integration help text, causing users to file support tickets when a high-frequency Zap gets throttled
- Pinning the integration to an API version and never updating it — when that version is deprecated, every user's automations break at once
