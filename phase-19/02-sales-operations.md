# Sales Operations

---

## What It Is

The sales infrastructure for closing B2B deals — CRM for pipeline management, enterprise pricing configuration, contract templates, proposal generation, and a demo environment. Everything a founder or small sales team needs to run an enterprise sales process without hiring a sales ops team.

## Design Decisions

**CRM:** Attio. It handles lead tracking, deal pipeline, and contact management. Attio is modern, API-first, and doesn't require a Salesforce admin to configure. Create a single pipeline with these stages: Lead, Qualified, Demo, Proposal, Negotiation, Closed Won, Closed Lost. Every inbound enterprise inquiry starts as a Lead and moves through the pipeline.

**Pipeline stages:**
- **Lead** — inbound inquiry received (form submission, email, referral). No qualification yet.
- **Qualified** — confirmed the prospect has budget, authority, need, and timeline. Meets minimum criteria: 10+ seats, annual billing willingness.
- **Demo** — live product walkthrough scheduled or completed.
- **Proposal** — pricing proposal and contract sent to the prospect.
- **Negotiation** — prospect is reviewing terms, requesting changes, or getting internal approval.
- **Closed Won** — deal signed, payment received or invoicing set up.
- **Closed Lost** — prospect declined. Record the reason (price, feature gap, chose competitor, timing).

**Enterprise pricing:** Custom plans created in Stripe. Enterprise plans differ from self-serve plans: higher usage limits, custom feature flags, annual billing with invoicing (net-30 terms), dedicated support channel. Create each enterprise plan as a custom Stripe Price object tied to a custom Product. Don't reuse self-serve plan Products for enterprise deals.

**Contract templates:** Store templates in `docs/contracts/` in the repository. Three templates:
- **NDA** — mutual non-disclosure agreement for pre-sales conversations
- **MSA** — master service agreement covering terms of service, liability, SLA guarantees, data processing
- **SoW** — statement of work for the specific deal (scope, pricing, timeline, deliverables)

Use Claude to generate initial drafts from a structured prompt with the deal specifics (company name, deal terms, custom requirements). Have a lawyer review the templates once, then reuse with per-deal customization.

**Proposal generation:** Create a proposal template in Notion. The template includes: executive summary, product overview, proposed plan and pricing, implementation timeline, support terms, and next steps. For each deal, duplicate the template and fill in the deal-specific details. Generate the first draft with Claude using the deal details from Attio, then review and customize manually.

**Sales demo environment:** A separate workspace in your product with realistic seed data that showcases all features. This workspace is never used for development or testing — it is exclusively for sales demos. Seed data includes: 3 sample projects with tasks, 5 team members with different roles, realistic usage history, and example integrations. Create a guided walkthrough script (a checklist of features to show in order) stored in `docs/sales/demo-script.md`.

**Demo data maintenance:** Reset the demo workspace to its seeded state after every demo. Build a seed script that truncates and re-inserts the demo data. Run it via a Server Action in an internal admin panel.

## What to Test

- Attio pipeline has all 7 stages configured correctly
- Enterprise Stripe plans are created with correct pricing and billing intervals
- Contract templates render correctly with placeholder variables filled in
- Proposal template is complete and covers all sections
- Demo workspace has realistic seed data across all features
- Demo seed script resets the workspace to a clean state
- Closed Lost deals capture the loss reason

## Edge Cases

- **Prospect wants monthly billing** — enterprise plans are annual only. If the prospect insists on monthly, they use the self-serve plan with a volume discount applied as a Stripe coupon. Don't create custom monthly enterprise plans — it defeats the purpose of annual commitments.
- **Prospect needs custom SLA** — the MSA template includes a standard SLA (99.9% uptime). If the prospect needs a higher SLA, negotiate it as a line item in the SoW with corresponding pricing. Don't modify the MSA template for one-off requests.
- **Demo workspace data gets corrupted** — someone accidentally modifies the demo workspace during a live demo. The seed script resets it. Run the script before every demo, not just when something looks wrong.
- **Multiple deals in negotiation simultaneously** — Attio handles this natively. Each deal is a separate record in the pipeline. Use Attio's deal value field to prioritize which deals get attention first.
- **Prospect requests a feature that doesn't exist** — log it in the Closed Lost reason if the deal falls through. If the feature is on the roadmap, share the timeline. Never promise a feature delivery date in a contract.
- **Legal review bottleneck** — the prospect's legal team takes 4 weeks to review the MSA. This is normal for enterprise deals. Track legal review as a sub-stage within Negotiation using an Attio note, not a separate pipeline stage.

## Implementation Checklist

- [ ] Attio account created and pipeline configured with 7 stages
- [ ] Pipeline fields defined: company name, deal value, expected close date, loss reason
- [ ] Enterprise Stripe Products and Prices created (annual billing, net-30 invoicing)
- [ ] NDA template created and stored in `docs/contracts/nda.md`
- [ ] MSA template created and stored in `docs/contracts/msa.md`
- [ ] SoW template created and stored in `docs/contracts/sow.md`
- [ ] Proposal template created with all sections
- [ ] Claude prompt for contract draft generation written and tested
- [ ] Demo workspace created with a dedicated account
- [ ] Demo seed script built (truncate + re-insert sample data)
- [ ] Seed data covers: projects, tasks, team members, roles, usage history
- [ ] Demo walkthrough script written and stored in `docs/sales/demo-script.md`
- [ ] Enterprise inquiry form on the pricing page routes to Attio as a new Lead
- [ ] Closed Lost stage requires a loss reason before saving

## Common Mistakes

- Using a spreadsheet instead of a CRM (works for 3 deals, breaks at 10 — no pipeline visibility, no history)
- No demo environment (demoing on production or staging with broken data and half-finished features)
- Demo data is unrealistic (5 items named "test" and "asdf" — undermines credibility)
- Not resetting demo data between demos (previous demo's mess is visible to the next prospect)
- Handwriting contracts from scratch for every deal instead of using templates (slow, error-prone, inconsistent terms)
- Promising features in contracts that don't exist yet (legal liability when the feature ships late or differently)
- No loss reason tracking (no idea why deals fail — can't improve the product or the process)
- Enterprise plans on monthly billing (high churn risk, unpredictable revenue, no commitment signal)
- Overcomplicating the pipeline with 12 stages (stages should represent meaningful state changes, not micro-steps)
