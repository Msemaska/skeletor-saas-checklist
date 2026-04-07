# Search

---

## What It Is

How users find things in your product. This covers both basic filtering (narrowing a list by criteria) and full-text search (typing a query and getting ranked results). Both use Postgres — no external search engine needed for an MVP SaaS.

## Two Levels of Search

### Level 1: Filtering

Filtering narrows a list by known criteria. The user selects from predefined options — they're not typing a free-text query, they're refining what's shown.

- Dropdown filters (status, category, role, date range)
- Toggle filters (show archived, show completed)
- Sort controls (newest, oldest, alphabetical, most recent activity)
- Combined filters (status = active AND category = marketing AND created this month)

Filtering is just database queries with `WHERE` clauses. No special Postgres features needed — standard queries with proper indexes.

**Design decisions:**
- Filters are applied server-side via query parameters, not client-side JavaScript filtering. This ensures filtering works at scale (10,000+ records) and respects RLS.
- Filter state lives in the URL (query params) so filtered views are shareable and bookmarkable
- Every filterable column needs an index (covered in 01-database-schema.md)
- Filters always include the workspace/user scope — never let a filter bypass the tenant boundary

### Level 2: Full-Text Search

Full-text search handles free-text queries — the user types words and gets ranked results. This is for search bars, command palettes, and "find anything" features.

Use Postgres `pg_trgm` extension for fuzzy matching. It handles typos, partial matches, and similarity scoring without any external service.

**Why pg_trgm over tsvector:**
- `pg_trgm` handles typos and partial matches out of the box ("projct" matches "project")
- `tsvector` is better for natural language document search (articles, blog posts) but worse for short-form data (names, titles, descriptions)
- For SaaS product data (project names, workspace names, user names, short descriptions), `pg_trgm` is the right tool
- If you later need to search long-form content (knowledge base articles, documentation), add `tsvector` for that specific use case

**How it works:**
- `pg_trgm` breaks strings into groups of three characters (trigrams) and compares them using a similarity score
- A GIN index on the searchable columns makes these lookups fast
- The similarity threshold determines how closely the query must match (0.3 is a reasonable default — lower catches more typos, higher is stricter)

## What's Searchable

Decide upfront which entities and which columns are searchable. Every searchable column needs a GIN trigram index.

**Typically searchable in a SaaS product:**
- Workspace name and slug
- User names and emails (within the workspace)
- Primary entity names and descriptions (whatever your product's core objects are — projects, documents, tasks, etc.)

**Not searchable (query directly instead):**
- IDs and references (use exact-match lookup, not fuzzy search)
- Timestamps and dates (use date filters, not text search)
- Boolean/enum fields (use filters, not search)
- System fields (stripe_customer_id, internal metadata)

**Rules:**
- Search is always scoped to the current workspace — never return results from other workspaces
- Search results respect RLS — a user can only find things they have access to
- Limit results to 20 per query for dropdown results, 50 for full results pages (if applicable)

## Search UI Patterns

### Search Bar (most common)

- Debounce input by 300ms — don't query on every keystroke
- Show results in a dropdown below the search bar as the user types
- Each result shows: entity type icon, primary name, secondary info (e.g., workspace, status)
- Clicking a result navigates to that entity
- Pressing Enter with text but no selection selects the top result
- Show "No results found" with the query echoed back, not an empty state
- Clear button (X) to reset the search

### Command Palette (power user feature)

- Triggered by keyboard shortcut (Cmd+K / Ctrl+K)
- Displayed in a centered modal with a search input
- The search component uses the same search infrastructure as the search bar
- Additionally includes static actions (navigate to page, create new entity, toggle setting) — these are hardcoded navigation items, not database search results
- Group results by type (search results vs actions)
- Keyboard navigable (arrow keys to select, Enter to activate)

### Full Results Page (if needed)

- For products where search is a primary workflow, a dedicated results page with filters
- Results grouped by entity type with count per group
- Pagination or infinite scroll for large result sets
- Highlight matching text in results

**For an MVP, start with the search bar. Add the command palette as a quality-of-life improvement. The full results page is only needed if search is central to your product's workflow.**

## Performance

- **GIN trigram index** on every searchable column — without it, `pg_trgm` does a sequential scan
- **Composite search:** if you need to search across multiple columns simultaneously (name + description), create a generated column that concatenates them, and index that column
- **Limit results** — always use `LIMIT` in search queries. Never return unbounded results.
- **Monitor slow queries** — search queries on large tables can become slow. Check query performance regularly and add indexes as needed.
- **Don't search on page load** — search is triggered by user input, not by default. Loading a page should show a filtered/sorted list, not search results.

## What to Test

- Search results are scoped to the current workspace (no cross-tenant results)
- Fuzzy search handles typos (searching "projct" finds "project")
- Search with fewer than 3 characters shows minimum character message
- Empty search query shows the default list, not "no results"
- Search results are limited to 20 items in dropdown
- Filter state persists in URL query params (shareable, bookmarkable)
- UUID input triggers exact-match lookup in addition to fuzzy search
- Sort state persists in URL and toggles between ascending/descending

## Edge Cases

- **Empty search query** — show the default list view (filtered/sorted), not "no results." An empty search bar means "show me everything" not "I searched for nothing."
- **Single character search** — `pg_trgm` needs at least 3 characters for meaningful trigram matching. Show "Type at least 3 characters" for queries shorter than 3 characters instead of returning irrelevant results.
- **Search query with special characters** — sanitize the search input. Strip or escape characters that could break the query (SQL injection is handled by parameterized queries, but special Postgres pattern characters like `%` and `_` should be escaped for `LIKE` queries).
- **Search returns results the user can't access** — this shouldn't happen if search queries go through the service layer with RLS enforced. But if you're building a search index or cached view, ensure it respects permissions at query time.
- **User searches for an exact ID** — if the input looks like a UUID, do an exact-match lookup in addition to fuzzy search. Users sometimes paste IDs into search bars.
- **Search across multiple entity types** — run parallel queries per entity type and merge results client-side, rather than trying to search one giant table. Each entity type has its own searchable columns and display format.
- **Stale search results after mutation** — user creates a new project, immediately searches for it, and it doesn't appear. If using server-side search via Server Actions, revalidation should handle this. If using a cached search index, ensure it's updated on mutations.
- **Very long search query** — cap search input at 100 characters. Nobody types a meaningful search query longer than that, and extremely long strings can cause performance issues with trigram matching.

## Implementation Checklist

### Database
- [ ] `pg_trgm` extension enabled in Postgres
- [ ] GIN trigram indexes created on all searchable columns
- [ ] Searchable entities and columns documented
- [ ] Generated column for multi-column search (if searching across name + description)
- [ ] Similarity threshold configured (0.3 default)

### Search Queries
- [ ] All search queries scoped to current workspace (tenant boundary)
- [ ] All search queries go through service layer with RLS enforced
- [ ] Results limited to 20 (dropdown) or 50 (full page)
- [ ] Parameterized queries used (no string concatenation of search input)
- [ ] UUID detection for exact-match ID lookups

### UI
- [ ] Search bar with 300ms debounce
- [ ] Results dropdown with entity type, name, and secondary info
- [ ] "No results found" message with query echoed back
- [ ] Minimum 3 character requirement shown to user
- [ ] Clear/reset button on search input
- [ ] Search input max length capped at 100 characters
- [ ] Filter state stored in URL query params (shareable, bookmarkable)

## Common Mistakes

- Using an external search engine (Meilisearch, Algolia) before outgrowing Postgres (unnecessary complexity and cost for an MVP)
- Client-side filtering on large datasets (breaks at scale, bypasses RLS, loads all data into the browser)
- No debounce on search input (fires a query on every keystroke, hammers the database)
- Missing GIN trigram indexes (search works but does a sequential scan — fine on 100 rows, unusable on 10,000)
- Searching across workspaces (tenant data leakage)
- Returning unbounded results (no LIMIT — returns thousands of rows for a common search term)
- Not handling short queries (1-2 character searches return meaningless results from trigram matching)
- Filter state in component state instead of URL params (filtered views can't be shared or bookmarked)
- Building the full results page before validating that users need it (search bar is sufficient for most MVPs)
