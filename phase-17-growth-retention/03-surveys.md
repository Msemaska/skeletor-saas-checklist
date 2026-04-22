# NPS Surveys

---

## What It Is

In-app Net Promoter Score surveys that measure user satisfaction and collect qualitative feedback. Delivered through PostHog Surveys, not email, for higher response rates.

## Survey Design

### Design Decisions

**Design decisions:**
- Use PostHog Surveys to create and deliver the NPS survey — it's already in the stack for product analytics (phase-11-observability/02-product-analytics.md)
- The survey has 2 questions:
  1. "How likely are you to recommend [product name] to a friend or colleague?" (0-10 scale, standard NPS question)
  2. "What could we improve?" (open-ended text field, optional)
- The survey appears as a small modal/popover anchored to the bottom-right of the screen — not a full-page overlay, not a blocking modal
- Dismiss button visible — users can close the survey without answering
- If dismissed, don't show the survey again for 90 days
- If completed, don't show the survey again for 180 days

## Survey Trigger

### Design Decisions

**Design decisions:**
- Trigger: 30 days after the user's first login, and only if they have been active on at least 10 of those 30 days
- "Active" means they performed at least 1 meaningful action in the app (not just logging in — creating, editing, or interacting with a core entity)
- The 30-day / 10-active-day criteria is checked via PostHog user properties — set a property like `first_active_date` and `active_days_count` and use PostHog's targeting rules
- Only show the survey to users on paid plans — free/trial users don't have enough investment to give meaningful NPS feedback
- Show the survey during a natural pause (page load on the dashboard, not mid-workflow)

## NPS Tracking

### Design Decisions

**Design decisions:**
- NPS score calculated monthly: (% promoters - % detractors) x 100
- Promoters: 9-10, Passives: 7-8, Detractors: 0-6 (standard NPS segmentation)
- Segment NPS by plan tier — track whether higher-tier users are more satisfied
- Review open-ended responses monthly for recurring themes
- Store all survey responses in PostHog — don't duplicate in your database
- Track response rate (surveys shown vs. surveys completed) to ensure the survey isn't too intrusive
- Target response rate: 15% or higher

## What to Test

- Survey appears after 30 days with 10+ active days
- Survey does not appear for free/trial users
- Survey does not appear for users with fewer than 10 active days in 30 days
- Dismissing the survey hides it for 90 days
- Completing the survey hides it for 180 days
- NPS score (0-10) is recorded correctly in PostHog
- Open-ended response is recorded correctly in PostHog
- Survey renders correctly on mobile and desktop
- Survey does not block the user from interacting with the app
- Survey does not appear during active workflows (only on dashboard page load)

## Edge Cases

- **User is active for 10 days then stops using the product** — the survey trigger checks active days at the 30-day mark. If they had 10 active days but haven't logged in for the last 2 weeks, the survey will show on their next login after day 30. This is fine — their NPS score reflects their actual experience.
- **User downgrades from paid to free** — don't show the survey. Free users are excluded from the targeting rules.
- **User submits NPS 0 with no comment** — accept it. Don't force the open-ended question. A score of 0 with no comment still counts as a detractor.
- **User sees the survey on mobile but the popover is too small** — the survey popover must be responsive. On mobile, render it as a bottom sheet instead of a corner popover.
- **NPS score drops significantly month-over-month** — this is a signal, not a bug. Review the open-ended responses from that month for the root cause. If the drop is 10+ points, investigate immediately.

## Implementation Checklist

- [ ] PostHog Surveys enabled in the PostHog project
- [ ] NPS survey created with 2 questions (0-10 scale + open-ended)
- [ ] Survey targeting: paid plan users only, 30+ days since first login, 10+ active days
- [ ] Survey trigger: dashboard page load only
- [ ] Survey dismissal cooldown: 90 days
- [ ] Survey completion cooldown: 180 days
- [ ] Survey styled to match the app (bottom-right popover on desktop, bottom sheet on mobile)
- [ ] PostHog user properties set: first_active_date, active_days_count
- [ ] Monthly NPS calculation process defined (% promoters - % detractors)
- [ ] NPS segmented by plan tier in PostHog
- [ ] Monthly review cadence for open-ended responses scheduled
- [ ] Response rate tracked (target: 15%+)

## Common Mistakes

- Sending NPS via email instead of in-app (email NPS surveys get 5% response rates — in-app gets 20%)
- Triggering the survey too early (before the user has enough experience to give meaningful feedback)
- Showing the survey to free users (they haven't committed enough to provide actionable NPS data)
- Blocking the UI with a full-screen survey modal (users close it instantly out of frustration, biasing results toward detractors)
- Not segmenting NPS by plan (hides the fact that one plan tier is significantly less satisfied)
- Forcing the open-ended question (reduces completion rate — make it optional)
- Storing NPS data in your database when PostHog already stores it (unnecessary duplication)
- Ignoring open-ended responses (the score tells you the sentiment, the comments tell you why)
