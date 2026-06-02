---
name: skeletor
description: Skeletor is an opinionated, deterministic reference architecture for building a SaaS product from idea to launch. Use when starting or planning a SaaS build, deciding architecture or tech stack, scoping which systems a SaaS needs, generating a build roadmap, or filling gaps in a product plan. Routes to a phased master checklist and deep-dive design decisions covering strategy, branding, testing, core product, public API, UI/UX, marketing site, infrastructure, billing, support, observability, email, security, content, go-to-market, admin, growth, accessibility, partnerships, and app-store phases.
when_to_use: Invoke at the start of any SaaS project, or whenever a decision needs a pre-made, consistent answer instead of an improvised one.
allowed-tools: Read, Glob, Grep, Write
---

# Skeletor — SaaS Launch Checklist & Reference Architecture

Skeletor is a knowledge base, not a codebase. It pre-makes every architecture and product decision a SaaS needs so that implementation is **deterministic** — the same inputs produce nearly the same build every time. When this skill is active, follow Skeletor's decisions rather than improvising your own.

All files referenced below ship with this plugin under `${CLAUDE_PLUGIN_ROOT}`.

## How to use Skeletor on a project

1. **Read the rules first.** Read `${CLAUDE_PLUGIN_ROOT}/CLAUDE.md` — it defines how the knowledge base is structured and the "How to Use This Repo" project-type table.

2. **Read the master checklist.** Read `${CLAUDE_PLUGIN_ROOT}/saas-launch-checklist.md`. It lists every system across 20 phases (0–19), each marked:
   - `[1]` need while developing · `[2]` need before launch · `[3]` need after launch

3. **Scope by project type.** Not every project needs every phase. Use the project-type table in `CLAUDE.md` to decide which phases apply (e.g. an internal tool skips strategy, brand, and marketing phases). Only load what's relevant — this keeps context focused.

4. **Generate the roadmap.** Run the bundled `/skeletor:init` command (or follow its logic) to produce a `PROGRESS.md` in the project root: one line per applicable checklist item, split into the three passes, each linking to its deep-dive file.

5. **Build in three passes, phases in order within each pass.** Pass 1 = all `[1]` items, Pass 2 = all `[2]` items, Pass 3 = `[3]` items as needed. Phases are ordered by dependency (database before auth, auth before billing), so respect phase order within a pass.

## The prime rule when implementing

**Before implementing any checklist item, read its deep-dive file in full.** Each phase has a folder at the plugin root named `phase-N-<slug>/` containing numbered deep-dive files (e.g. `${CLAUDE_PLUGIN_ROOT}/phase-3-core-product/03-auth.md`). The file number matches the checklist item number.

Each deep-dive file gives you the design decisions, security considerations, edge cases, what-to-test, and common mistakes for that system. Do not improvise a system that has a deep-dive file — implement what the file specifies. The checklist line is only a pointer; the deep-dive file is the authoritative source.

## Phase map

`phase-0-strategy-positioning` · `phase-1-brand-foundation` · `phase-2-testing` · `phase-3-core-product` · `phase-4-public-api` · `phase-5-ui-ux` · `phase-6-marketing-site` · `phase-7-infrastructure` · `phase-8-business-wrapper` · `phase-9-customer-support` · `phase-10-observability` · `phase-11-email-domain` · `phase-12-security-audit` · `phase-13-content` · `phase-14-go-to-market` · `phase-15-admin-operations` · `phase-16-growth-retention` · `phase-17-accessibility-i18n` · `phase-18-partnerships-sales` · `phase-19-app-store`

Glob `${CLAUDE_PLUGIN_ROOT}/phase-*/*.md` to list every available deep-dive file.

## Non-negotiable principles (from `CLAUDE.md`)

- **User-first.** Favour the user's trust, time, and confidence over developer convenience. No surprise charges, no dark patterns, no deleting user data for convenience.
- **Opinionated, not educational.** State the decision; don't present options unless there's a genuine architectural fork.
- **Handle the edge cases.** Every deep-dive file lists real-world edge cases users hit in month one — implement them, don't skip them.
- **Test what you build.** Every deep-dive file's "What to Test" section travels with the feature.
