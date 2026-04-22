# API Documentation

---

## What It Is

Developer-facing documentation for the public API. Includes endpoint reference, authentication guide, code examples, and an interactive explorer where developers can make live requests. The docs are auto-generated from the OpenAPI spec so they stay in sync with the actual API.

## Design Decisions

**Documentation framework:** Fumadocs. It is a Next.js-native docs framework, so it runs in the same codebase — no separate static site generator, no separate deployment. Host the docs at `docs.yourdomain.com` via a subdomain rewrite.

**OpenAPI spec:** Auto-generate the OpenAPI 3.1 spec from route definitions. Use `zod-openapi` to annotate the existing Zod request/response schemas with OpenAPI metadata (descriptions, examples, tags). Serve the generated spec at `/api/openapi.json` as a public endpoint. This spec is the single source of truth — the docs, the interactive explorer, and the SDK all consume it.

**Interactive API explorer:** Scalar. Embed it in the Fumadocs site so developers can read the docs and try requests without leaving the page. Scalar reads the OpenAPI spec from `/api/openapi.json` and renders a try-it-out interface with request/response previews. Developers paste their API key into the explorer and make real requests against the API.

**Documentation structure:**
- **Getting Started** — overview, authentication, base URL, rate limits, error handling, pagination
- **Endpoint Reference** — auto-generated from the OpenAPI spec, grouped by resource (workspaces, members, billing, etc.)
- **Guides** — multi-step tutorials for common workflows (create a workspace, invite members, set up webhooks)
- **Changelog** — dated entries for API changes, new endpoints, deprecations

**Writing process:** The endpoint reference is auto-generated from the spec — never write it manually. The Getting Started page and Guides are written manually in MDX files within the Fumadocs content directory. The Changelog is a manually maintained MDX file updated with every API release.

**Keeping docs in sync:** The OpenAPI spec is generated at build time from the Zod schemas that the API routes already use for validation. If a schema changes, the spec changes, and the docs update automatically on the next deploy. No manual step required for endpoint reference updates.

## Security Considerations

- API documentation is public (no auth required to read docs)
- The interactive API explorer requires an API key to make requests — it does not use a shared test key
- Documentation does not expose internal implementation details (database schema, service function names, internal error codes)
- OpenAPI spec served at a public endpoint — does not include internal/admin-only endpoints

## What to Test

- OpenAPI spec is valid (passes OpenAPI 3.1 validation)
- `/api/openapi.json` endpoint returns the spec with correct content type
- Scalar explorer loads and renders endpoints from the spec
- Scalar explorer can make authenticated requests with a test API key
- Fumadocs pages render without errors
- Every API endpoint appears in the generated spec
- Request/response examples in the spec match actual API behavior
- Internal-only endpoints are excluded from the public spec

## Edge Cases

- **Spec generation fails** — a Zod schema is missing OpenAPI metadata. The build should fail loudly rather than generating an incomplete spec. Add a CI check that validates the spec has entries for every public route.
- **Developer uses the explorer against production** — this is expected. The explorer makes real requests with the developer's real API key. Document this clearly so developers know they are modifying real data.
- **Spec grows very large** — hundreds of endpoints make the JSON file large. Scalar handles large specs fine. If the file exceeds 1MB, split by tag into separate spec files and configure Scalar to load them selectively.
- **Deprecated endpoints** — mark deprecated endpoints in the Zod schema's OpenAPI metadata with `deprecated: true`. Scalar renders them with a visual indicator. Keep them in the spec until the version is sunset.

## Implementation Checklist

- [ ] Fumadocs installed and configured in the Next.js project
- [ ] Docs hosted at `docs.yourdomain.com`
- [ ] `zod-openapi` added to annotate existing Zod schemas with OpenAPI metadata
- [ ] OpenAPI spec generation script built (runs at build time)
- [ ] `/api/openapi.json` endpoint serving the generated spec
- [ ] Scalar embedded in the Fumadocs site as the interactive explorer
- [ ] Getting Started page written (auth, base URL, rate limits, errors, pagination)
- [ ] Endpoint reference auto-generated from the spec
- [ ] Guides written for common workflows
- [ ] Changelog page created
- [ ] CI check validates the OpenAPI spec on every PR
- [ ] Internal/private endpoints excluded from the public spec
- [ ] Navigation sidebar organized by resource and guide category

## Common Mistakes

- Writing endpoint documentation manually instead of generating from the spec (docs drift from reality within weeks)
- Using a separate documentation platform that requires manual deploys (Notion, GitBook) instead of keeping docs in the codebase
- No interactive explorer (developers have to switch to Postman or curl to test — friction kills adoption)
- Forgetting to document error responses (developers only see success examples and have no idea what 400/401/403 responses look like)
- Hosting docs at a completely different domain with no link from the main site (developers can't find them)
- Not including authentication instructions in the Getting Started page (developers' first API call fails and they give up)
- Generating the spec at runtime instead of build time (slow first request, unnecessary computation)
- Including internal endpoints in the public spec (leaks implementation details)
