# Product Analytics & Feature Flags

---

## Provider

Use PostHog. It handles product analytics, funnels, session replay, and feature flags in one platform. Self-hostable if needed.

**AI interface:** PostHog MCP.

## Product Analytics

### What to Track

Track events that answer: "Are users getting value from the product?"

**Core events every SaaS tracks:**

| Event | When | Why |
|---|---|---|
| User signed up | Account created | Measure acquisition |
| Onboarding completed | Wizard finished | Measure activation |
| First value action | User creates their first entity | Measure time-to-value |
| Feature used | User uses a core feature | Measure engagement |
| Subscription created | User subscribes to a paid plan | Measure conversion |
| Subscription cancelled | User cancels | Measure churn |
| Plan upgraded/downgraded | Subscription changes | Measure expansion |

**Design decisions:**
- Track events on the server side (Server Actions, API routes), not the client — server-side events are reliable (no ad blockers, no browser differences) and tied to real business logic
- Every event includes: user ID, workspace ID, timestamp, and relevant properties (plan name, feature name, etc.)
- Don't track everything — track events that map to your business questions. "How many button clicks on the settings page" is not useful. "How many users complete onboarding" is.
- Respect cookie consent — if the user rejected analytics cookies, don't send events to PostHog (covered in phase-9/05-legal-pages.md)

### Funnels

Define the key conversion funnels and monitor them:

**Sign-up → Value funnel:**
1. Visited pricing page
2. Signed up
3. Completed onboarding
4. Created first entity
5. Returned next day (Day-1 retention)

**Free → Paid funnel:**
1. Active free user (used the product in the last 7 days)
2. Viewed pricing page
3. Started checkout
4. Completed subscription

**Design decisions:**
- Define funnels before launch — instrument the events during development, not after
- Review funnel metrics weekly — a drop at any stage indicates a UX or messaging problem
- PostHog's funnel analysis shows where users drop off and how long each step takes

### Session Replay

PostHog records user sessions so you can watch exactly what a user did when they report a bug or when you want to understand a funnel drop-off.

**Design decisions:**
- Enable session replay on PostHog (opt-in per plan — may require paid tier)
- Mask sensitive fields automatically (password inputs, credit card fields) — PostHog does this by default
- Use session replay for debugging user-reported issues and understanding funnel drop-offs, not for surveillance
- Session recordings are retained for 30 days

## Feature Flags

Control which users see new features without redeploying.

**Design decisions:**
- Use PostHog's feature flags — they're evaluated on the server and client
- Use feature flags for: gradual rollouts (10% → 50% → 100%), A/B testing (show variant A to half, variant B to half), kill switches (disable a feature instantly if it causes issues)
- Feature flags are temporary — once a feature is fully rolled out, remove the flag and the conditional code. Flags that stay forever become tech debt.
- Feature flag names follow the convention: `feature-name-enabled` (e.g., `ai-suggestions-enabled`, `new-dashboard-enabled`)

**When to use feature flags:**
- Launching a new feature to a subset of users before full rollout
- Testing a UX change with a percentage of users
- Deploying code that isn't ready to be visible yet (feature is in progress, code is merged but hidden)
- Kill switch for a feature that depends on an external service (disable if the service is down)

**When NOT to use feature flags:**
- Configuration that should be in environment variables (API keys, service URLs)
- User preferences (these are user settings, not feature flags)
- Plan-based feature gating (this is driven by subscription state, not flags)

## What to Test

- Core events fire correctly (sign-up, onboarding, first action, subscription)
- Events include the correct properties (user ID, workspace ID, plan)
- Events respect cookie consent (no tracking when analytics cookies rejected)
- Funnels show correct conversion rates in PostHog
- Session replay captures sessions and masks sensitive fields
- Feature flags control feature visibility correctly (enabled = visible, disabled = hidden)
- Feature flag changes take effect without redeployment

## Edge Cases

- **User has analytics cookies rejected** — no events sent to PostHog. The app still functions, you just have no analytics for that user. Accept the data gap rather than violating consent.
- **Ad blocker blocks PostHog** — if using client-side tracking, ad blockers may block PostHog. Server-side event tracking bypasses this entirely. Default to server-side for critical events.
- **Feature flag evaluation on the server vs client** — server-side evaluation is more reliable (no race conditions, no flash of wrong content). Use server-side evaluation for page-level features, client-side for interactive UI toggles.
- **Feature flag removal** — when removing a flag, search the codebase for all references. A forgotten flag check that always evaluates to the default value is dead code that confuses future developers.
- **Session replay storage** — session recordings consume storage. PostHog's retention policy (30 days) prevents unbounded growth. Don't extend retention without considering storage costs.

## Implementation Checklist

- [ ] PostHog account created and SDK installed (server-side and client-side)
- [ ] Core events instrumented (sign-up, onboarding, first action, subscription, cancellation)
- [ ] Every event includes user ID, workspace ID, and relevant properties
- [ ] Server-side tracking used for business-critical events
- [ ] Cookie consent respected (no tracking when rejected)
- [ ] Sign-up → Value funnel defined in PostHog
- [ ] Free → Paid funnel defined in PostHog
- [ ] Session replay enabled with sensitive field masking
- [ ] Feature flags configured for any features launching with gradual rollout
- [ ] Feature flag naming convention documented (feature-name-enabled)

## Common Mistakes

- Tracking every click and page view (too much noise, no actionable insight)
- Client-side only tracking (ad blockers block it, critical events go unrecorded)
- No funnels defined (you have data but no way to see where users drop off)
- Tracking events without user ID or workspace ID (can't segment, can't debug)
- Feature flags that never get removed (codebase accumulates dead conditional code)
- Using feature flags for user preferences or plan gating (wrong tool for the job)
- Ignoring cookie consent (tracking users who rejected analytics — legal violation)
- Not reviewing analytics weekly (data accumulates, nobody looks at it, problems go undetected)
