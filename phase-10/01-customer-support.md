# Customer Support Platform

---

## Provider

Use Tawk.to. It handles live chat, help desk, knowledge base, and feedback in one platform.

## What Tawk.to Provides

| Feature | Purpose |
|---|---|
| Live chat widget | Real-time messaging on marketing site and in the app |
| Help desk / inbox | Centralized place to manage all conversations |
| Knowledge base | Self-service articles users can search before contacting support |
| Feedback widget | In-app feedback collection from users |

## Live Chat Widget

**Design decisions:**
- Show the chat widget on every page — marketing site and app
- Widget position: bottom-right corner
- Widget shows availability status — if nobody is online, show "We'll reply within 24 hours" instead of "We're online"
- Pre-chat form: ask for email before starting the chat — this lets you follow up if they disconnect before getting a response
- Chat history persists for the user — if they come back later, they can see previous conversations
- For authenticated app users: pass the user's name, email, and workspace to Tawk.to so the support agent has context without asking
- For marketing site visitors: only email is available (from the pre-chat form)

## Help Desk

**Design decisions:**
- All conversations (live chat, email replies to support) land in Tawk.to's inbox
- Single inbox — no separate queues for billing, technical, general. One person handles everything for an MVP.
- When a conversation requires action (bug report, feature request, billing issue), tag it in Tawk.to and track it manually. Don't build a custom ticketing system.
- Response time target: 24 hours for all inquiries. Display this on the chat widget and contact page.
- Conversations from the chat widget include the user's metadata (name, email, workspace, plan) so you don't have to ask for it.

## Knowledge Base

**Design decisions:**
- Build the knowledge base in Tawk.to — it provides a hosted, searchable knowledge base out of the box
- Structure articles by category: Getting Started, Account & Billing, Features, Troubleshooting
- Write articles for the top 10 questions you expect before launch (how to sign up, how to reset password, how to upgrade, how to cancel, how to invite team members, etc.)
- Link to the knowledge base from: the chat widget ("Search our help center first"), the app's help menu, and the marketing site footer
- Articles are public (no auth required) — users should find answers without signing in

**Content rules for KB articles:**
- Title is the question the user would ask: "How do I cancel my subscription?" not "Subscription Cancellation Process"
- Answer in the first sentence, then details below
- Include screenshots for UI-specific instructions
- Keep articles under 500 words — if it's longer, split into multiple articles
- Update articles when the product changes — stale KB articles are worse than no articles

## Feedback Widget

**Design decisions:**
- In-app feedback widget allows users to submit feedback, bug reports, and feature requests from within the app
- Feedback lands in the same Tawk.to inbox as chat conversations — no separate tool
- Include a screenshot/screen recording option — visual context reduces back-and-forth
- Tag feedback by type: bug, feature request, general feedback
- Review feedback weekly and use it to inform the product roadmap

## Integration with the App

**Design decisions:**
- Pass user context to Tawk.to when the chat widget loads in the authenticated app: user name, email, workspace name, current plan, account creation date
- This context appears in the Tawk.to sidebar when the support agent opens the conversation — no need to ask "what's your email?" or "what plan are you on?"
- For the marketing site: no user context is available, the pre-chat form collects the email

## Integration with Alerting

Support volume is a signal. Route it to the unified alerting hub (phase-11/01-unified-alerting.md):

**Design decisions:**
- Tawk.to can send webhooks on new conversations — post a notification to the #alerts-info Discord channel when a new support conversation starts
- Don't alert on every message within a conversation — only on new conversations
- If support volume spikes (3x normal), investigate — it may indicate a bug, outage, or confusing UX change

## What to Test

- Chat widget appears on marketing site (bottom-right)
- Chat widget appears in the authenticated app with user context (name, email, plan)
- Pre-chat form collects email on the marketing site
- Messages reach the Tawk.to inbox
- Knowledge base is accessible and searchable
- Knowledge base link works from the chat widget, app help menu, and marketing footer
- Feedback submission from within the app reaches the Tawk.to inbox
- New conversation webhook posts to Discord #alerts-info

## Edge Cases

- **User starts chat then navigates away** — Tawk.to maintains the conversation. If they return, the chat history is there. If they don't, the support agent can follow up via email (collected from the pre-chat form or user context).
- **Support agent is offline** — the widget shows "We'll reply within 24 hours." The message is queued in Tawk.to and the agent responds when they're back. Don't hide the chat widget when offline — users should still be able to leave a message.
- **User submits feedback about a critical bug** — the support agent should escalate by creating an issue in the project tracker and tagging the conversation. Tawk.to is for communication, not issue tracking.
- **Knowledge base article is outdated** — a user follows an article that no longer matches the UI. Include a "Was this helpful?" prompt at the bottom of every article. If an article gets consistently negative feedback, update it.
- **Spam in the chat widget** — Tawk.to has built-in spam filtering. The pre-chat form (requiring email) reduces spam from bots. If spam is persistent, enable Tawk.to's anti-spam features.
- **User context not loading in the widget** — if the Tawk.to SDK fails to load user context (network issue, SDK error), the chat still works — just without the sidebar metadata. The agent asks for context manually.

## Implementation Checklist

- [ ] Tawk.to account created
- [ ] Chat widget installed on marketing site
- [ ] Chat widget installed in the authenticated app
- [ ] User context passed to Tawk.to in the app (name, email, workspace, plan)
- [ ] Pre-chat form enabled for unauthenticated visitors (requires email)
- [ ] Availability message configured ("We'll reply within 24 hours" when offline)
- [ ] Knowledge base created with initial articles (top 10 expected questions)
- [ ] Knowledge base linked from: chat widget, app help menu, marketing footer
- [ ] KB articles follow content rules (question as title, answer first, under 500 words)
- [ ] Feedback widget enabled in the app
- [ ] Tawk.to webhook configured to post new conversations to Discord #alerts-info
- [ ] Response time target displayed (24 hours)

## Common Mistakes

- Hiding the chat widget when nobody is online (users can't leave a message — they leave the site instead)
- No pre-chat form (user disconnects, you have no way to follow up)
- No user context in the app widget (agent asks "what's your email?" to an authenticated user — wastes time, feels impersonal)
- Empty knowledge base at launch (every question requires a live conversation)
- KB articles written from the product's perspective, not the user's ("Subscription Cancellation Process" instead of "How do I cancel?")
- Building a custom ticketing system instead of using Tawk.to's inbox
- No feedback collection mechanism (users report bugs via chat, which is harder to track than tagged feedback)
- Separate tools for chat, tickets, and KB
