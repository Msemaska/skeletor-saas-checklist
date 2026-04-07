# Cache Layer Strategy

---

## What It Is

Caching stores the result of expensive operations (database queries, computations, API calls) so they don't need to be repeated on every request. In a Next.js app, caching happens at multiple levels — most of it is built-in and free. An external cache (Redis) is not needed for a standard SaaS product.

## Cache Layers

```
User Request
  │
  ▼
┌─────────────────────────────────┐
│  Layer 1: CDN / Edge Cache      │  ← Static assets, ISR pages
│  (handled by hosting provider)  │     You configure, don't build
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 2: Next.js Server Cache  │  ← use cache, cacheLife, cacheTag
│  (Server Components, functions) │     The main caching layer you control
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 3: Client Cache          │  ← TanStack Query
│  (browser, per-user)            │     Handles client-side reads
└───────────────┬─────────────────┘
                ▼
┌─────────────────────────────────┐
│  Layer 4: Database Cache        │  ← Postgres query cache
│  (automatic, don't touch)       │     Managed by the database engine
└─────────────────────────────────┘
```

## Layer 1: CDN / Edge Cache

Handled automatically by your hosting provider (Vercel, Cloudflare, etc.). You don't build this — you configure it.

**What gets cached at the edge:**
- Static assets (images, fonts, CSS, JS bundles) — cached indefinitely with content hashing
- ISR pages — cached at the edge, revalidated on the schedule you set
- Public API responses with cache headers — if you set Cache-Control headers, the CDN respects them

**Design decisions:**
- Static assets are handled by Next.js build output — no configuration needed
- For ISR pages (marketing site, blog posts, pricing page), set a revalidation period that makes sense for how often the content changes
- The app itself (dashboard, authenticated pages) is never edge-cached — it's dynamic per-user

## Layer 2: Next.js Server Cache

This is the primary caching layer you control. Next.js provides the `use cache` directive for caching Server Component output and async function results. This requires enabling `cacheComponents: true` in your next.config — it is not on by default.

**Development vs production:** Next.js disables most caching in dev mode. If caching appears broken locally, it's not — it's working as intended. Always test caching behaviour in a production build or preview deployment, not in `next dev`.

**What to cache:**
- Data that is the same for all users in a workspace (workspace settings, plan details, shared project data)
- Data that rarely changes (reference tables, categories, feature flags)
- Expensive computations (aggregations, reports, analytics summaries)
- External API responses (third-party data that doesn't change every second)

**What NOT to cache:**
- User-specific data that changes frequently (notifications, real-time counters, activity feeds)
- Data that must be up-to-the-second accurate (billing state during checkout, usage limits mid-action)
- Anything after a mutation — revalidate instead of serving stale data

**Cache profiles — match the duration to how often the data actually changes:**

| Data Type | Duration | Revalidation |
|---|---|---|
| Marketing pages (pricing, features) | Hours | On content change via revalidateTag |
| Blog posts | Hours to days | On publish/edit via revalidateTag |
| Workspace settings | Minutes | On settings change via revalidateTag |
| Dashboard data (project lists, team members) | 30 seconds | On mutation via revalidatePath |
| Reference data (plans, categories) | Hours | On admin change via revalidateTag |
| User profile | Do not cache | Always fresh from database |
| Billing/subscription state | Do not cache | Always fresh from database |

**Revalidation:**
- Use `revalidateTag` when a specific piece of data changes (e.g., tag workspace settings with the workspace ID, revalidate that tag when settings are updated)
- Use `revalidatePath` when an entire page's data has changed (e.g., after creating a project, revalidate the project list page)
- Revalidation happens in Server Actions after the mutation succeeds — this is covered in 04-data-access.md

**Private vs shared cache:**
- Most cached data in a SaaS is workspace-scoped, not user-scoped. Tag cache entries with the workspace ID so different workspaces don't see each other's cached data.
- For data that is truly the same for everyone (public pricing, marketing content), use shared cache with no workspace scoping.
- For data that varies per user within a workspace (user preferences, notification settings), do not cache server-side — fetch fresh every time or cache client-side with TanStack Query.

## Layer 3: Client Cache (TanStack Query)

TanStack Query manages all client-side caching. See 04-data-access.md for the full TanStack Query patterns (query keys, invalidation after mutations, placeholderData). The caching-specific decisions are:

**Stale time — how long before cached data is considered stale and triggers a background refetch:**

| Data Type | Stale Time |
|---|---|
| Real-time data (messages, activity feeds) | 0 seconds (always refetch) |
| Dashboard data (project lists, team members) | 30 seconds |
| Settings and preferences | 60 seconds |
| Reference data (plans, categories) | 5 minutes |

**Cache time (gcTime) — how long data stays in memory after the component unmounts:**
- 5 minutes for most data
- 15 minutes for reference data that rarely changes
- When the user navigates away and back within cache time, data shows instantly from cache while a background refetch happens

## Layer 4: Database Cache

Postgres has its own internal caching (shared buffers, query plan cache). You do not configure or manage this directly.

**The only thing you control:**
- Write efficient queries so Postgres can cache and reuse query plans
- Use indexes properly so queries hit cache instead of doing full scans
- Avoid `SELECT *` — select only the columns you need so more rows fit in the buffer cache

This is covered in 01-database-schema.md under indexing strategy.

## When You Actually Need Redis

Redis (or Upstash) is NOT part of the default stack. Add it only if you hit one of these specific needs:

- **API rate limiting at scale** — if you're offering a public API (Phase 16) and need per-key rate limiting across distributed serverless functions. For auth rate limiting on a standard SaaS, database-backed counters are sufficient.
- **Job queues** — if Trigger.dev/Inngest aren't suitable and you need a self-hosted queue (BullMQ requires Redis)
- **Shared state across serverless functions** — if you need to share state between concurrent serverless invocations that can't use the database (rare)
- **Real-time leaderboards or counters** — if you need atomic increments at thousands of operations per second

If you're not hitting one of these, you don't need Redis. Next.js cache + TanStack Query + Postgres handles everything a standard SaaS needs.

## What to Test

- Cached data is scoped by workspace ID (no cross-tenant cache leakage)
- Revalidation clears the correct cache entries after mutations
- User-specific data (profile, billing) is never served from shared cache
- Cache miss falls through to database correctly
- Stale cache is refreshed on revalidation, not served indefinitely

## Edge Cases

- **Stale cache after deployment** — a new deployment may change the shape of cached data. Next.js handles this by invalidating the server cache on deploy. TanStack Query client cache persists across navigations but not across page reloads (full deploy triggers a reload).
- **Cache stampede** — many users request the same uncached data simultaneously, causing a flood of identical database queries. Next.js deduplicates identical fetch requests within a single render, but cross-request deduplication requires cache-aware data fetching (which `use cache` handles).
- **Cached data with user-specific content** — if you accidentally cache a Server Component that renders user-specific data (e.g., "Welcome, John"), all users see the same cached version. Never cache components that include user-specific content unless you're using private cache scoped to the user.
- **Revalidation doesn't propagate to client cache** — you revalidate the server cache after a mutation, but the user's browser still shows TanStack Query's stale data. Always invalidate both caches after mutations.
- **Over-caching billing state** — a user upgrades their plan but the cached plan name still shows "Free" for the next 5 minutes. Never cache billing or subscription state — always fetch fresh.
- **Cache key collision across workspaces** — if you cache workspace data without including the workspace ID in the cache tag, one workspace's data leaks to another. Always scope cache tags by workspace ID.

## Implementation Checklist

- [ ] ISR configured for marketing/public pages with appropriate revalidation periods
- [ ] `use cache` directive used for expensive shared data (not user-specific)
- [ ] Cache tags include workspace ID for workspace-scoped data
- [ ] Revalidation happens after every mutation (revalidateTag or revalidatePath)
- [ ] TanStack Query stale times configured per data type
- [ ] TanStack Query keys structured hierarchically for granular invalidation
- [ ] Both server cache and client cache invalidated after mutations
- [ ] User-specific data (profile, notifications, billing) is never server-cached
- [ ] No Redis in the stack unless a specific need arises (API rate limiting, job queues)

## Common Mistakes

- Adding Redis to the stack before you have a problem that requires it (unnecessary complexity and cost)
- Caching user-specific data in shared server cache (data leaks between users)
- Caching billing/subscription state (stale plan data causes access control bugs)
- Revalidating server cache but forgetting to invalidate TanStack Query client cache (UI shows stale data)
- Not scoping cache tags by workspace ID (cross-tenant data leakage)
- Setting cache duration too long for data that changes frequently (users see stale data)
- Setting cache duration too short for data that rarely changes (defeats the purpose of caching)
- Using `unstable_cache` in new projects (replaced by `use cache` directive in Next.js 16)
- Caching everything by default (caching is an optimization — add it where you have a measured performance problem, not preemptively)
