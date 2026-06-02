# User Journey Map

---

## What It Is

The canonical path your user takes from "doesn't know your product exists" to "couldn't live without it." It defines the stages, the single most important moment in the journey (activation), and the measurable signal at each stage. Every downstream system — onboarding flow, lifecycle emails, product analytics, billing prompts, retention mechanics — implements against this map. Without it, those systems get designed in isolation and contradict each other.

This is **not** a UI flow. UI flows live in Phase 5 (which screens, which buttons, which states). This is the strategic layer above: what counts as success at each stage, and what we measure to prove it.

## How It Differs From Adjacent Documents

| Document | Question It Answers |
|---|---|
| ICP (`02-ideal-customer-profile.md`) | *Who* is the user |
| Value proposition (`01-value-proposition.md`) | *What* outcome they get |
| **User journey (this file)** | **What path they take and what proves they reached value** |
| Onboarding (`phase-5-ui-ux/06-onboarding.md`) | *How* the first-run UI walks them through it |
| Product analytics (`phase-10-observability/02-product-analytics.md`) | *How* we instrument the journey |

One journey per ICP. If you have two ICPs, you have two journeys — different triggers, different aha moments, different success signals.

## The Six Stages

```
Trigger ──► Evaluation ──► Sign-Up ──► Activation ──► Habit ──► Expansion
(pre-product)              (in product)
```

Stages 0–2 happen before the user touches the product. Stages 3–5 happen inside it. The handoff between Sign-Up and Activation is where most SaaS products lose users — average activation rate sits around 37%, meaning roughly 6 in 10 signups never reach value.

### Stage 1: Trigger

The moment a person stops tolerating their current situation and starts looking for a solution. Defined by the **four forces of progress** from Jobs-to-be-Done: a switch only happens when *Push* (frustration with current state) and *Pull* (appeal of new solution) are stronger than *Anxiety* (fear of the new thing) and *Habit* (inertia of the old way).

**For each ICP, define:**
- The push event (concrete, datable: "I just spent 3 hours doing X manually," "My current tool raised prices," "My team grew past spreadsheets")
- The pull (the specific outcome your product offers that they now want)
- The anxieties (migration cost, learning curve, lock-in, data security)
- The habit (what they're doing instead, including doing nothing)

This is not theoretical. The trigger dictates the channel, the headline, and the first thing the marketing site must address.

### Stage 2: Evaluation

The user has landed on a touchpoint (marketing site, content, ad, peer recommendation, app store, integration directory) and is deciding whether to try. Time-on-site is short — they will make this decision in under a minute.

**Decisions to make:**
- The single question their landing page must answer (usually: "is this for me, and what specifically does it do?")
- The proof they need to convert to sign-up (demo video, free tier, screenshot, social proof, security badge)
- The friction barrier — credit card or no, email or SSO, free trial length, freemium gate

The output of this stage is the **sign-up rate from a primary landing page** — not a vanity number, but the conversion of qualified visitors who match the ICP.

### Stage 3: Sign-Up

The account is created. This is **not activation**. Treating sign-up as success is the single most expensive mistake in SaaS — it leads to inflated funnels and lifecycle emails that congratulate users for doing nothing yet.

**Decisions to make:**
- What identity is required (email + password, Google OAuth, magic link, SSO — Phase 3 auth covers the implementation)
- What is collected at sign-up vs. deferred (the rule: collect only what's needed to unlock the activation path; everything else can come later)
- Whether the user is dropped into the product, an onboarding wizard, or an empty workspace
- Whether they get a "welcome" email (yes — and it should point at the activation path, not "thanks for signing up")

### Stage 4: Activation (The Aha Moment)

The single most important decision in this entire document. Activation is the moment the user experiences the core value of the product for the first time, defined as a **measurable action with a threshold and a time window**.

**Format:** `[specific user action] × [count] × [within time window]`

| Product | Activation Event |
|---|---|
| Slack | Team sends 2,000 messages |
| Dropbox | File saved to one folder, accessed on a second device |
| Facebook (historical) | 7 friends added in 10 days |
| Calendly | First meeting booked through a shared link |
| Figma | Created a file with 2+ collaborators |

**Rules for choosing the activation event:**
- It must be a *user action*, not a system state ("uploaded a file," not "has data in the database")
- It must correlate with retention — pick the action that historically separates users who stay from users who leave (early in life, this is a hypothesis; later, validate with cohort analysis)
- It must be reachable in **under 10 minutes** for first-time users on the golden path
- It must be a single event, not a list of "any of these counts"
- One activation event per ICP. If you have two ICPs, you have two activation events.

**Time to Value (TTV)** is the time from sign-up to activation. The industry-wide average is roughly 1.5 days. Aim for under 10 minutes. Every minute beyond that loses users in proportion.

The activation event drives:
- Onboarding flow design (every screen exists to push the user toward this one event)
- The empty state (the first thing the user sees if their workspace is blank — a launchpad with a single CTA, not a tour)
- Lifecycle email triggers (signed up but not activated → nudge after 24h, 3d, 7d, then stop)
- The credit/usage metering thresholds in Phase 3
- The product analytics north star in Phase 11

### Stage 5: Habit

The user has activated and now returns repeatedly. This is where retention is won or lost — a user who returns 3+ times in their first week is overwhelmingly likely to stay.

**Decisions to make:**
- The **habit signal**: the action that, when repeated, predicts long-term retention (e.g., "logs in 3 of 7 days in week 1," "completes the core action 5 times in 14 days")
- The **free-to-paid conversion trigger**: at what point in the habit phase does the paywall show, and what specifically prompts it (usage limit hit, feature gate, time-based trial expiry)
- The signals that flag a user is **at risk** (no return for 7 days post-activation, drop in core-action frequency, support ticket sentiment)

In PLG SaaS, paid conversion happens here, not as its own stage. A user pays because they've already become reliant on the product — not because a paywall convinced them.

### Stage 6: Expansion & Advocacy

The user is paying, retained, and now grows in value: more seats, higher tier, referrals, public advocacy.

**Decisions to make:**
- The **expansion event** (added a teammate, upgraded plan, exceeded usage tier, integrated with a second tool)
- The **advocacy ask** — when and how to ask for testimonials, reviews, referrals (Phase 17 retention mechanics implements this; the journey map decides *when in the lifecycle* the ask happens)
- The **power user signal** — usage pattern that identifies a candidate for case studies, beta features, community ambassadorship

## What to Instrument at Each Stage

Every stage gets one primary metric. More than one means you don't know what matters yet.

| Stage | Primary Metric | Where It's Tracked |
|---|---|---|
| Trigger | (Inferred — channel attribution by message tested) | PostHog + UTMs |
| Evaluation | Sign-up rate from primary landing page | PostHog funnel |
| Sign-Up | Sign-ups per day (qualified — matches ICP) | PostHog + Stripe |
| **Activation** | **% of sign-ups who reach the activation event within TTV target** | **PostHog (the north star)** |
| Habit | Week-2 retention (% of activated users who return in week 2) | PostHog cohorts |
| Expansion | Net revenue retention (NRR) | Stripe + PostHog |

The activation metric is the closest thing to a north star metric a SaaS product has — it captures real value delivered and predicts revenue.

## How This Map Is Used Downstream

| Phase | How It Consumes The Journey Map |
|---|---|
| Phase 3 — Credit/usage system | Activation event defines what's metered; habit signals define usage tiers |
| Phase 3 — Notifications | Lifecycle email triggers fire on stage transitions (signed up, activated, at-risk, etc.) |
| Phase 5 — Onboarding | First-run wizard exists to push user to the activation event in under 10 minutes |
| Phase 5 — Component states | Empty state design = activation launchpad |
| Phase 8 — Transactional emails | Welcome, activation nudge, at-risk, win-back emails are stage-driven |
| Phase 9 — Stripe | Free-to-paid trigger and dunning recovery hook into habit-stage signals |
| Phase 11 — Product analytics | Funnels, cohorts, and the north star metric are defined here |
| Phase 14 — Content | Content topics are organized by stage (top-of-funnel = trigger, bottom = activation/expansion) |
| Phase 17 — Retention mechanics | Win-back, NPS, referral asks are timed against the habit and expansion stages |

## Journey Map Template

```
**ICP:** [Name from ICP doc]

**Stage 1 — Trigger**
- Push: [What frustrates them now]
- Pull: [What we offer that they want]
- Anxiety: [What scares them about switching]
- Habit: [What they do instead today]
- Channel hypothesis: [Where they are when the trigger fires]

**Stage 2 — Evaluation**
- Primary landing page: [URL or description]
- Question it must answer: [The one thing they need to know]
- Proof element: [Demo, free tier, screenshot, social proof]
- Sign-up friction: [What we ask for]

**Stage 3 — Sign-Up**
- Auth method: [Email, Google, etc.]
- Data collected: [Fields]
- Post-signup destination: [Workspace, wizard, empty state]

**Stage 4 — Activation (Aha Moment)**
- Activation event: [Action × count × time window]
- TTV target: [Minutes from sign-up]
- Empty-state CTA: [The single thing the empty workspace prompts]

**Stage 5 — Habit**
- Habit signal: [Returning action × frequency × window]
- Free-to-paid trigger: [Limit, gate, or time]
- At-risk signal: [What flags churn risk]

**Stage 6 — Expansion**
- Expansion event: [Seat add, tier upgrade, integration]
- Advocacy ask timing: [When in lifecycle]
- Power-user signal: [Usage pattern]
```

## Deliverables

- [ ] One journey map per ICP, fully filled in using template above
- [ ] Activation event defined as `action × count × time window`
- [ ] Time-to-value target set (under 10 minutes)
- [ ] Habit signal defined
- [ ] At-risk signal defined
- [ ] Free-to-paid conversion trigger decided
- [ ] Expansion event defined
- [ ] Primary metric per stage assigned to a tool

## Edge Cases

- **User signs up but never activates.** The most common failure mode. Send a stage-aware nudge at 24h, 3d, 7d post-signup pointing directly at the activation event — then stop. If the activation event is unreachable in under 10 minutes, the event is wrong, not the user.
- **User activates but never returns.** The activation event correlates poorly with retention. Re-examine: pick a different event further into the product, or a higher count threshold.
- **User is highly engaged but never pays.** The free tier is too generous OR the paywall trigger is misaligned with the habit signal. Move the paywall to the moment the user hits a real usage ceiling, not an arbitrary day count.
- **User completes the onboarding wizard but never uses the real product.** The wizard is a tutorial, not the path to value. Wizard completion ≠ activation. Replace the wizard with a path that puts the user inside the product completing the activation event.
- **Multi-seat product where one user activates but invited teammates don't.** Inviter activation and invitee activation are separate events. Track both. Invitee activation often requires a different empty state because they're entering an existing workspace, not an empty one.
- **User cancels paid plan but stays on free.** Treat as a stage regression, not churn. Trigger downgrade survey + win-back sequence. They are a higher-quality lead than a cold sign-up.
- **Returning user resurrected after 30+ days dormant.** They are effectively a new user. Re-run them through a condensed activation path; do not assume they remember the product.
- **Power user hits the upper tier ceiling.** This is an expansion signal, not a friction point. Trigger a sales/upgrade conversation, not a hard block.
- **Two distinct ICPs sharing one product.** Each gets its own journey, its own activation event, its own onboarding. Do not collapse them — the average of two journeys serves neither.
- **Activation event chosen, but the data shows no correlation with retention.** The hypothesis was wrong. Run cohort analysis on retained vs. churned users and find the action that actually separates them. Update the journey map.

## Common Mistakes

- Treating sign-up as success. Sign-up is the start of the funnel, not the win.
- Defining activation as "completed the onboarding tour." Tour completion is not value delivery.
- Vague activation events ("user is engaged"). If you can't write it as `action × count × window`, it's not measurable.
- One journey for all ICPs. The journey should fork by ICP.
- Tracking everything. One primary metric per stage. The rest is noise until you have signal.
- Skipping the trigger and evaluation stages because they happen "before the product." They define your channels and headline — without them, marketing is a guess.
- Building lifecycle emails before the journey map exists. The triggers will be wrong because the stages are undefined.
- Picking an activation event that's too easy (e.g., "logged in once"). It must correlate with retention, not just signup.
- Picking an activation event the user can't reach in under 10 minutes. Time-to-value beyond that loses users in proportion.
- Never revisiting the journey map. It's a hypothesis until cohort data confirms it. Update it quarterly in the first year.
