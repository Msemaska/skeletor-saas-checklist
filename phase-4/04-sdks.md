# SDK Generation

---

## What It Is

A TypeScript SDK published to npm that wraps your public API with typed methods. Generated from the OpenAPI spec so it stays in sync with the API automatically. Developers install the package, pass their API key, and call typed methods instead of constructing HTTP requests manually.

## Design Decisions

**Generation tool:** `openapi-typescript` to generate TypeScript types from the OpenAPI spec. Use `openapi-fetch` as the runtime HTTP client — it consumes the generated types and provides typed methods for every endpoint. These two packages work together: `openapi-typescript` generates the types at build time, `openapi-fetch` uses them at runtime for type-safe API calls.

**Package name:** `@yourproduct/sdk` published to npm. Replace `yourproduct` with your actual product name. Use npm's organization scope to own the namespace.

**SDK structure:**
- `src/client.ts` — creates the configured API client with base URL and API key
- `src/types.ts` — auto-generated types from the OpenAPI spec (never edit manually)
- `src/index.ts` — exports the client factory and all types

**Authentication helper:** The SDK exports a `createClient` function that accepts an API key and returns a configured client instance. The client automatically adds the `Authorization: Bearer <key>` header to every request. No manual header management for the developer.

**Error handling:** The client throws typed errors for non-2xx responses. Error objects include `status`, `code`, and `message` matching the API's error response format. Developers catch errors and inspect the typed fields instead of parsing raw response bodies.

**Regeneration workflow:** When the API changes, regenerate types by running `openapi-typescript` against the latest `/api/openapi.json` spec. This happens as a CI step: after the API deploys, a GitHub Action fetches the new spec, regenerates types, runs tests, and publishes a new SDK version to npm automatically.

**Versioning:** The SDK version follows semver independently from the API version. A non-breaking API change (new field added) bumps the SDK's minor version. A breaking API change (new API version) bumps the SDK's major version. The SDK's major version matches the API version it targets (SDK v1.x targets API v1, SDK v2.x targets API v2).

**Package contents:** The published npm package includes compiled JavaScript (ESM), TypeScript declaration files, and the source maps. No source TypeScript files in the published package.

## Security Considerations

- SDK stores the API key in memory, never writes it to disk or logs
- SDK documentation instructs users to store their API key in environment variables, not in source code
- SDK never includes the API key in error messages or stack traces
- SDK uses HTTPS for all API calls — no HTTP fallback

## What to Test

- `createClient` returns a configured client instance
- Client adds the Authorization header to every request
- Typed methods exist for every public API endpoint
- Success responses are correctly typed
- Error responses throw typed errors with status, code, and message
- Pagination helpers work (fetching next page with cursor)
- Generated types match the current OpenAPI spec
- Package builds without errors
- Package installs from npm and imports correctly in a fresh project

## Edge Cases

- **OpenAPI spec has an endpoint the SDK doesn't cover** — this means the regeneration step was skipped. The CI pipeline should fail if the generated types are out of date compared to the deployed spec. Add a check that compares the spec hash.
- **Developer uses SDK v1 against API v2** — the SDK targets a specific API version in its base URL (`/api/v1/`). It will continue working as long as API v1 is live. When v1 is deprecated, SDK v1 stops working and developers must upgrade to SDK v2.
- **Network error vs API error** — network errors (timeout, DNS failure) are different from API errors (4xx, 5xx). The client should throw distinct error types so developers can handle them differently. Network errors don't have a status code or response body.
- **Rate limited** — the SDK receives a 429 response. The error object includes the `Retry-After` value from the response header. The SDK does not automatically retry — the developer decides how to handle rate limiting in their application.

## Implementation Checklist

- [ ] npm organization scope registered (`@yourproduct`)
- [ ] SDK repository created (monorepo package)
- [ ] `openapi-typescript` configured to generate types from `/api/openapi.json`
- [ ] `openapi-fetch` configured as the runtime HTTP client
- [ ] `createClient` function implemented with API key authentication
- [ ] Typed error classes created for API errors and network errors
- [ ] `package.json` configured with ESM output, TypeScript declarations, and exports map
- [ ] Build script produces compiled JS + declaration files
- [ ] README written with quickstart: install, authenticate, make first call
- [ ] GitHub Action: on API deploy, regenerate types, run tests, publish to npm
- [ ] Semver versioning strategy documented
- [ ] SDK major version matches target API version
- [ ] Test suite covers client creation, typed requests, and error handling
- [ ] `files` field in package.json excludes source TypeScript and test files from published package

## Common Mistakes

- Writing the SDK by hand instead of generating from the spec (SDK drifts from the API within days)
- Not publishing TypeScript declarations (developers lose type safety — the whole point of a TS SDK)
- Auto-retrying rate-limited requests in the SDK (the developer's application should control retry behavior, not the SDK)
- Bundling the entire OpenAPI spec in the published package (bloats the package, leaks internal details)
- No automated publish pipeline (someone forgets to publish after an API change and the SDK is stale for weeks)
- Pinning the SDK to a specific API version without documenting it (developers don't know which API version they're targeting)
- Not including a README with a quickstart (developers install the package and have no idea how to use it)
- Publishing source TypeScript files to npm (unnecessary, increases package size)
