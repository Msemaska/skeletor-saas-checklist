---
name: init
description: Generate a PROGRESS.md build roadmap for the current project from the Skeletor master checklist, scoped to the project type. Use to bootstrap a new SaaS build before writing any code.
argument-hint: [project-type — e.g. "full SaaS", "internal tool", "MVP", "API-only"]
disable-model-invocation: true
allowed-tools: Read, Glob, Write
---

# Skeletor — Generate PROGRESS.md

Produce a `PROGRESS.md` in the current project root: the single, ordered task list for the entire build, where every item links to the Skeletor deep-dive file that contains its full design decisions.

## Steps

1. **Read the rules and checklist.**
   - Read `${CLAUDE_PLUGIN_ROOT}/CLAUDE.md` — focus on the "How to Use This Repo" project-type table.
   - Read `${CLAUDE_PLUGIN_ROOT}/saas-launch-checklist.md` in full.

2. **Determine project type.**
   - If `$ARGUMENTS` names a project type, use it.
   - Otherwise ask the user: full production SaaS, internal tool / admin dashboard, MVP / prototype, or API-only service.
   - Use the project-type table to decide which phases apply and which to skip. Exclude skipped phases entirely.

3. **Map each included checklist item to its deep-dive file.**
   - Deep-dive files live at `${CLAUDE_PLUGIN_ROOT}/phase-N-<slug>/NN-<name>.md`, where `NN` matches the checklist item number.
   - Glob `${CLAUDE_PLUGIN_ROOT}/phase-*/*.md` to confirm the exact filenames.
   - Items with no deep-dive file (e.g. "Domain purchased") link to the phase folder instead.

4. **Write `./PROGRESS.md`** with three sections — Pass 1 (`[1]`), Pass 2 (`[2]`), Pass 3 (`[3]`). Within each pass, order items by phase number to respect dependencies. One line per item, no grouping or summarising:

   ```
   - [ ] `[1]` Auth — `phase-3-core-product/03-auth.md`
   ```

   The checkbox tracks completion; the priority marker places the item in its pass; the path is the deep-dive file to read before starting that task.

5. **Confirm** the project type, which phases were included vs skipped, and the total item count back to the user.

## Rules

- Every included item gets exactly one line. No summarising.
- Never reorder items within a pass — phase order is intentional.
- The deep-dive file is authoritative; `PROGRESS.md` only points to it.
- Do not begin implementing anything — this command only produces the roadmap.
