# Webhook System

---

## What It Is

An event delivery system that notifies your customers' servers when things happen in your product. When an entity is created, updated, or deleted, your system sends an HTTP POST request to a URL the customer has registered. Powered by Svix, which handles signing, delivery, retries, and logging.

## Design Decisions

**Webhook provider:** Svix. It manages the entire webhook delivery pipeline — endpoint registration, payload signing, retry logic, and delivery logs. You send events to Svix via their API, and Svix delivers them to your customers' registered URLs. This removes the need to build retry queues, signing infrastructure, and delivery tracking from scratch.

**Event types:** Define a fixed set of event types that customers can subscribe to:
- `entity.created` — a new resource was created in the workspace
- `entity.updated` — a resource was modified
- `entity.deleted` — a resource was deleted
- `subscription.changed` — the workspace's billing plan changed (upgrade, downgrade, cancellation)
- `member.added` — a new member joined the workspace
- `member.removed` — a member was removed from the workspace

Replace `entity` with your actual resource names (e.g., `project.created`, `task.updated`). Each event type is a dot-separated string following the `resource.action` pattern.

**Payload format:** Every webhook payload includes:
- `id` — unique event ID (UUID v4), used for idempotency
- `type` — event type string (e.g., `entity.created`)
- `workspace_id` — which workspace this event belongs to
- `created_at` — ISO 8601 timestamp of when the event occurred
- `data` — the full resource object at the time of the event

**Signing:** Svix signs every payload with HMAC-SHA256. Each customer endpoint gets a unique signing secret. The signature is included in the `svix-signature` header. Customers verify the signature before processing to confirm the payload came from your system and was not tampered with.

**Retries:** Svix retries failed deliveries (non-2xx responses) with exponential backoff. 5 retry attempts over approximately 8 hours. After all retries are exhausted, the event is marked as failed in the delivery log. Svix handles this automatically.

**Endpoint registration:** Customers register webhook URLs in their workspace settings. Each workspace can register up to 5 endpoints. Each endpoint can subscribe to specific event types or all event types. Registration is stored in Svix via their API — your database only stores a reference to the Svix endpoint ID.

**Delivery logs:** Svix provides delivery logs showing every attempt for every event — timestamp, response status code, response body, and retry count. Expose these logs in the workspace settings UI so customers can debug failed deliveries themselves.

**Test events:** Customers can send a test event from the dashboard for any event type. This sends a sample payload to their registered URL so they can verify their server is receiving and processing webhooks correctly before going live.

## Security Considerations

- Webhook payloads are signed with HMAC-SHA256 — customers must verify the signature before processing
- Each workspace has its own webhook signing secret — compromising one customer's secret doesn't affect others
- Webhook payloads contain only data the workspace has access to — no cross-tenant data leakage
- Webhook endpoint URLs are validated on registration (must be HTTPS)
- Failed webhook deliveries are logged but webhook payload data is not stored indefinitely (purge delivery logs after 30 days)

## What to Test

- Creating a webhook endpoint in workspace settings stores it in Svix
- Deleting a webhook endpoint removes it from Svix
- Events are sent to Svix when resources are created, updated, or deleted
- Payload includes all required fields (id, type, workspace_id, created_at, data)
- Event ID is unique per event
- Test event delivery sends a sample payload to the registered URL
- Delivery logs are visible in workspace settings
- Customers can filter endpoints by event type
- Maximum of 5 endpoints per workspace is enforced
- Events are only sent for event types the endpoint is subscribed to

## Edge Cases

- **Customer's server is down** — Svix retries 5 times with exponential backoff. If all retries fail, the event appears as failed in the delivery log. No further action needed — the customer checks their logs and replays manually if needed.
- **Customer's server returns 200 but doesn't process the event** — not your problem. The delivery log shows a successful delivery. Idempotency keys let the customer reprocess safely.
- **Duplicate event delivery** — Svix may deliver the same event more than once (e.g., network timeout where the server processed but didn't respond in time). Every event has a unique `id` field. Document that customers must deduplicate by event ID.
- **Event ordering** — webhooks may arrive out of order. Two rapid updates to the same resource may arrive with the second update first. Include `created_at` on every event so customers can determine the correct order. Document that ordering is not guaranteed.
- **Workspace deleted but has registered endpoints** — delete all Svix endpoints for the workspace when the workspace is deleted. Use Svix's API to remove them.
- **Payload too large** — a resource with many nested relations produces a large payload. Cap payload size at 256KB. If the resource exceeds this, send a slim payload with the resource ID and type, and let the customer fetch the full resource via the API.

## Implementation Checklist

- [ ] Svix account created and API key stored in environment variables
- [ ] Svix SDK installed (`svix` npm package)
- [ ] Webhook endpoint CRUD in workspace settings UI (create, list, delete)
- [ ] Endpoint subscription filtering by event type
- [ ] Maximum 5 endpoints per workspace enforced
- [ ] Event dispatch function created — accepts event type and payload, sends to Svix
- [ ] Event dispatch called in all relevant service functions (create, update, delete)
- [ ] Payload includes: id (UUID v4), type, workspace_id, created_at, data
- [ ] Delivery logs displayed in workspace settings (fetched from Svix API)
- [ ] Test event button in workspace settings sends a sample payload
- [ ] Signing secret displayed to customers for verification (shown once on endpoint creation)
- [ ] Documentation: payload format, event types, signature verification, idempotency

## Common Mistakes

- Building webhook delivery infrastructure from scratch instead of using Svix (retry queues, signing, and delivery tracking are harder than they look)
- Not signing payloads (customers can't verify the webhook came from you — security risk)
- No retry logic (a single network blip means lost events)
- Sending webhooks synchronously in the request path (slows down the API response — always dispatch asynchronously)
- No delivery logs (customers have no way to debug why their integration isn't working)
- No test event feature (customers deploy to production to test their webhook handler)
- Guaranteeing event ordering (you can't — network conditions make this impossible without significant complexity)
- Not documenting idempotency (customers process the same event twice and create duplicate records)
- Unlimited endpoints per workspace (a single workspace registers hundreds of endpoints and floods your delivery pipeline)
