# Credit / Usage Metering System

---

## What It Is

The system that tracks how much of your product a workspace or user has consumed, enforces plan limits, and alerts users when they're approaching or exceeding those limits. This is what makes your pricing tiers real — without it, every plan is functionally identical.

## Usage Types

Every SaaS meters one or more of these. Decide which apply to your product:

- **Action-based** — number of operations performed (messages sent, reports generated, API calls made)
- **Seat-based** — number of team members in a workspace
- **Storage-based** — amount of file storage consumed (GB)
- **Resource-based** — number of entities created (projects, workflows, forms)
- **Time-based credits** — consumable credits that represent compute time or processing units

Most products combine 2-3 of these. Seat-based + one other is the most common pattern (e.g., "Pro plan: 10 seats, 100 projects, 5GB storage").

## Architecture

```
User Triggers Action
  │
  ▼
Service Function
  │ 1. Check: is the user within their limit?
  │    ├── Yes → proceed to step 2
  │    └── No → reject with "limit reached" error
  │ 2. Execute the action
  │ 3. Increment the usage counter
  ▼
Done
```

**The order is non-negotiable: check BEFORE, execute, then increment.** Never execute first and check after — the user has already consumed the resource.

## Where Limits Live vs Where Counts Live

Separate the limit definition from the current count. They change for different reasons and at different times.

**Limits** — defined per plan, stored wherever your plan definitions live:
- Each plan specifies: max seats, max projects, max storage, max actions per period, etc.
- When a workspace changes plans, the limits change immediately
- Limits are the same for every workspace on the same plan

**Current counts** — tracked per workspace, stored in your database:
- Each workspace has its own usage record tracking current consumption
- Counts increment as the workspace uses the product
- Counts reset on billing period boundaries (for period-based usage) or never (for cumulative usage like storage)

**Rules:**
- Never hardcode limits in application code — always read them from the plan definition
- Never store limits on the workspace record — the workspace references its plan, and the plan defines the limits
- Each usage record tracks which workspace it belongs to, what type of usage, the current count, and the billing period window (start and end timestamps)

## Enforcement Strategy

Use hard limits. When a workspace hits its limit, block the action and show a clear upgrade prompt.

**How enforcement works:**
- Before every metered action, query the workspace's current usage and compare against the plan's limit
- If at or over the limit, return an error with a clear message: "You've reached your limit of X. Upgrade to [next plan] for more."
- The error should include: what limit was hit, the current count, the plan limit, and a link to the upgrade page

**Where enforcement runs:**
- In the service layer, inside the service function for the metered action — not in middleware, not in the frontend
- The frontend should also check limits for UX (disable the "create" button when at limit, show usage bar) but this is informational only — the server enforces

## Usage Alerts

Alert users before they hit the limit so they're not surprised.

**Thresholds:**
- At 80% usage: trigger a notification through the notification system (channel decision is handled in 07-notification-system.md)
- At 100% usage: block the action, show upgrade prompt

**Design decisions:**
- Usage warnings are dismissible but reappear on next page load if still above threshold
- Show a usage bar/indicator in the relevant section of the UI (e.g., project count in the projects sidebar, storage usage in settings)
- Usage data is fetched from the server, not cached client-side — it must be current when making enforcement decisions
- The billing/settings page shows a full usage breakdown: every metered resource with current count, plan limit, and a progress bar. This is where users go to understand their consumption across all resources, not just the one they hit a limit on.

## Period-Based Usage Reset

For action-based and time-based usage, counters reset at the start of each billing period.

**Design decisions:**
- Billing period is defined by Stripe — when Stripe fires the `invoice.paid` webhook for a new period, reset the workspace's usage counters
- The usage record tracks period_start and period_end timestamps — any action outside this window should trigger a reset check before incrementing
- Resetting means setting the counter to 0, not deleting the record — preserve the record for the new period
- If the webhook is delayed (Stripe retries, network issues), the check-before-increment logic handles it: if the current period has ended, reset the counter before checking the limit

## Seat-Based Limits

Seats are the number of workspace members. Unlike action-based usage, seats don't reset — they're a current count.

**Design decisions:**
- Check seat limit when inviting a new member or accepting an invitation — not when the member signs up (they might not be joining this workspace)
- If at the seat limit, block the invitation with "Your plan supports X members. Upgrade to add more."
- When a member is removed, the seat count decreases immediately — no need to wait for a billing cycle
- When downgrading to a plan with fewer seats than the current member count, block the downgrade — require the workspace to remove members first

## Storage-Based Limits

Storage usage is cumulative — it doesn't reset per period.

**Design decisions:**
- Track total storage consumed per workspace in bytes
- Increment when files are uploaded, decrement when files are deleted
- Check limit before accepting an upload — reject with "Storage limit reached" if the upload would exceed the limit
- Show storage usage in workspace settings (e.g., "2.3 GB of 5 GB used" with a progress bar)
- When downgrading to a plan with less storage than currently used, don't delete files — grandfather the existing storage but block new uploads until usage drops below the new limit

## Plan Changes Mid-Period

When a workspace upgrades or downgrades, usage needs to be handled.

**Upgrade:**
- New (higher) limits take effect immediately
- Current usage counts carry over — don't reset on upgrade
- If the workspace was blocked at their old limit, they're immediately unblocked under the new limit

**Downgrade:**
- New (lower) limits take effect at the start of the next billing period, not immediately (the workspace already paid for the current period at the higher tier)
- If current usage exceeds the new plan's limit, don't delete anything — block new actions but preserve existing data
- For seats: if current member count exceeds the new plan's limit, block the downgrade until members are removed

## Race Conditions

Two requests can check the limit simultaneously, both see "under limit," and both increment — pushing the count over the limit.

**Design decisions:**
- For action-based usage: use an atomic database operation (increment and check in a single query) instead of separate read-then-write. A single UPDATE with a WHERE clause that includes the limit check prevents overshooting.
- For seat-based usage: the unique constraint on (workspace_id, user_id) in workspace_members prevents duplicate members. The seat count check + insert should be wrapped in a transaction.
- For storage-based usage: minor overshoot (by one upload) is acceptable — the next upload will be blocked. Perfect enforcement on storage is not worth the complexity.

## Security Considerations

- Usage counters are updated in system context only — users cannot modify their own usage counts
- Limit checks run server-side in the service layer — client-side limit checks are informational only
- Atomic increment-and-check prevents race condition exploitation (two requests passing the check simultaneously)
- Plan and credit values are never accepted from client requests — always derived from the workspace's plan

## What to Test

- Usage check happens BEFORE the action executes (not after)
- Action is blocked when workspace is at the plan limit
- Error message includes: what limit, current count, plan limit, upgrade link
- Usage counter increments atomically (no race condition overshoot)
- Usage counter resets on billing period change (Stripe webhook)
- Seat limit is checked on invitation, not on member sign-up
- Storage limit is checked before upload, not after
- Upgrade: new limits take effect immediately, counts carry over
- Downgrade: blocked if current usage exceeds new plan limits (seats, storage)
- Free plan with NULL limit is treated as unlimited
- System/admin operations do not increment usage counters

## Edge Cases

- **Free plan with no limits on some metrics** — represent "unlimited" as NULL in the plan definition. The enforcement check treats NULL as "no limit" and always passes.
- **Workspace on a custom/enterprise plan** — custom plans are just another plan record with custom limits. No special code paths.
- **Usage counter drifts from reality** — the counter says 95 projects but there are actually 97 (bug, failed decrement, etc.). Run a periodic reconciliation job that counts actual records and corrects the counter. Schedule this weekly.
- **Workspace has usage but no active subscription** — trial ended, subscription cancelled. Treat as the free plan's limits (or zero limits if no free plan). Don't delete their data — just block new actions.
- **User creates and immediately deletes an entity** — both actions should update the counter (increment on create, decrement on delete). The count reflects current state, not historical total.
- **Bulk operations** — user imports 50 records at once. Check if the remaining capacity is sufficient for the entire batch before starting — don't process 30 of 50 then fail.
- **Stripe webhook delay on period reset** — the billing period has ended but the webhook hasn't arrived yet. The check-before-increment logic should compare the current timestamp against the usage record's period_end. If the period has ended, reset the counter regardless of whether the webhook has arrived.
- **Downgrade blocked by seat count** — the workspace owner tries to downgrade but has too many members. Show a clear message listing how many members need to be removed and which plan they're downgrading to.
- **Admin/system operations that bypass limits** — system context operations (migrations, seeding, bulk updates) should not increment usage counters. Usage only tracks user-initiated actions.

## Implementation Checklist

### Data Model
- [ ] Plan definitions include limits for each metered resource (seats, actions, storage, etc.)
- [ ] Usage record per workspace tracks current counts with period start/end
- [ ] Limits and counts are separated (plan defines limits, workspace record tracks counts)
- [ ] NULL represents "unlimited" in plan definitions
- [ ] No limits hardcoded in application code

### Enforcement
- [ ] Check-before-execute pattern on every metered action
- [ ] Enforcement runs in the service layer (not middleware, not frontend)
- [ ] Atomic increment-and-check to prevent race conditions on action-based usage
- [ ] Clear error message when limit is hit (what limit, current count, plan limit, upgrade link)
- [ ] Bulk operations check total capacity before starting

### Alerts
- [ ] 80% usage warning banner shown in-app (dismissible, reappears on page load)
- [ ] Usage indicator in relevant UI sections (project count, storage bar)
- [ ] 100% limit block with upgrade prompt

### Period Management
- [ ] Usage counters reset on Stripe `invoice.paid` webhook for new billing period
- [ ] Period boundary check before increment (reset if period has ended regardless of webhook)
- [ ] Weekly reconciliation job corrects counter drift

### Plan Changes
- [ ] Upgrade: new limits take effect immediately, counts carry over
- [ ] Downgrade: new limits take effect at next billing period
- [ ] Downgrade blocked if current usage exceeds new plan limits (seats, storage)
- [ ] Seat limit checked on invitation, not on member sign-up
- [ ] Storage limit checked before upload, not after

## Common Mistakes

- Checking usage AFTER the action executes (user already consumed the resource before being told they can't)
- Separate read-then-write instead of atomic check-and-increment (race condition allows overshoot)
- Hardcoding limits in application code instead of reading from plan definitions (every plan change requires a code deploy)
- Storing limits on the workspace record instead of the plan (limits drift when plan changes)
- Resetting usage counters on the client or via a cron job instead of on Stripe webhook (out of sync with actual billing period)
- Not handling the gap between period end and webhook arrival (users are blocked or unblocked incorrectly)
- Overage billing instead of hard limits (surprises users, complex to implement, destroys trust)
- Not blocking downgrades when current usage exceeds the new plan's limits (data loss or broken state)
- Incrementing usage counters for system/admin operations (inflates usage beyond what the user actually consumed)
- No reconciliation job (counters drift over time from bugs, failed decrements, edge cases)
