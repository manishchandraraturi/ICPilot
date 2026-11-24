# phases.md — Phased Build Plan

> One phase at a time. Never start phase N+1 until phase N's acceptance criteria are met with evidence and the human/DeepSeek has approved the report.
> Each phase is split into **slices** (one Antigravity prompt each). A slice should touch ≤ ~10 files and take a weak model ≤ ~1 hour.
> Tag each finished phase `phase-Pn-done`. Update `progress.md` at the end of every slice.

## Global Definition of Done (every slice and phase)
1. Code written exactly to `prd.md`, `architecture.md`, `design.md`, `schema.md`, `security.md`, `AGENTS.md`.
2. `npm run lint && npm run typecheck && npm run test && npm run build` pass (raw output pasted in report).
3. New logic has tests. New UI has a screenshot in `evidence/Pn/`.
4. No fake data, no hardcoded leads, no secrets, no TODO left behind.
5. `progress.md` and `SPEC_CHECKLIST.md` updated.
6. Committed and pushed. Report in the exact format of `AGENTS.md` §10.
7. Every requirement ID of the phase is either `DONE` with evidence or reported as `SPEC GAP: <ID> — <REASON>` in CAPITALS.

---
## P0 — Repo, scaffold, tooling, CI skeleton, health route

- **Depends on:** none
- **Goal:** A clean, empty-but-correct project that lints, type-checks, tests, builds, and has CI. Nothing visual yet.

**Requirements covered (all mandatory):**
- `ADD-07` Structured logging + error boundary + /api/health

**Suggested slices (one Antigravity prompt each):**
- 0a repo + Next.js + tooling
- 0b env/logger/errors/health + CI skeleton

**Tasks:**
1. `git init`; create the repo layout from `architecture.md` §4 (empty folders get `.gitkeep`). If no GitHub remote exists: STOP and ask the human for the repo URL (do not invent one).
2. Copy `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.gitignore`, `.env.example`, `.nvmrc`, `.editorconfig`, `.prettierrc`, `docker-compose.yml`, `vercel.json`, `.github/workflows/ci.yml` from the scaffold. Do not modify their meaning.
3. Create Next.js app (TypeScript, Tailwind, App Router, `src/` dir, ESLint, no Turbopack-only flags). Enable `strict`, `noUncheckedIndexedAccess`.
4. Install: radix-ui primitives (as needed later), `zod`, `pino`, `@tanstack/react-query`, `@tanstack/react-table`, `@tanstack/react-virtual`, `@dnd-kit/core`, `lucide-react`, `react-hook-form`, `drizzle-orm`, `drizzle-kit`, `pg`, `@supabase/supabase-js`, `neo4j-driver`, `ai`, `@ai-sdk/google`, `nodemailer`, `vitest`, `@playwright/test`, `prettier`.
5. Add npm scripts exactly: `dev build start lint typecheck test test:e2e format db:generate db:migrate seed:demo eval:citations graph:rebuild check:env`.
6. `src/lib/env.ts` (zod; the only reader of `process.env`; server vs client split), `result.ts`, `errors.ts` (AppError + codes from `architecture.md` §18), `logger.ts` (pino, redacts secrets/emails).
7. `/api/health` returns `{ok, db, graph, llm, providers:{...}, dispatchMode}` — booleans/strings only, no secrets. Items not yet built report `not_built`.
8. One real unit test (e.g. `Result` helpers, env parsing).
9. Commit and push. Tag `phase-P0-done`.

**Acceptance criteria:**
- [ ] `npm run lint`, `typecheck`, `test`, `build` all pass.
- [ ] `GET /api/health` returns valid JSON with no secret values.
- [ ] Folder tree matches `architecture.md` §4.
- [ ] `.env.local` is ignored by git; `.env.example` lists every variable in `architecture.md` §8.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P1 — Design system + app shell (exact glass UI from reference images)

- **Depends on:** P0
- **Goal:** The exact glass UI from the 6 reference images exists as reusable components and a full app shell, with placeholder REAL-shaped (not fake-data) states. This is the visual foundation; do it perfectly before anything else.

**Requirements covered (all mandatory):**
- `UI-01` Hybrid layout: clean editorial look + dense spreadsheet-style operating surface
- `DSN-01` Page background gradient: slate-blue top fading to pale mint-white
- `DSN-02` Glass surface system (translucent fill, blur, white hairline border, soft shadow, large radius)
- `DSN-03` Typography: geometric light sans, thin large numerals, exact scale in design.md
- `DSN-04` Sidebar: search + filter button, Home/Dashboard tiles, tree with active pill and count badges, bottom icon row, navy feedback pill, avatar + bell
- `DSN-05` Top nav (logo, app name, tabs) + page header (back circle, title, filter input, Refresh, download, share, navy context pill)
- `DSN-06` Pinned-view folder cards with peeking preview sheets, '+' tile, dashed drop zone
- `DSN-07` Hero metric card: thin large number + 2x2 sub-stat grid
- `DSN-08` Filter chip row with dropdown chevrons
- `DSN-09` Hatched band chart: dark and grey dotted series, diagonal hatch fill, dashed gridlines, glass callout
- `DSN-10` Gauge ring card with navy delta pill
- `DSN-11` Thin vertical bar chart card with glass callout
- `DSN-12` Floating view-switcher dock (grid / board / table / chart)
- `DSN-13` Icon circle buttons, navy pill buttons, count badges
- `DSN-14` Interaction states: hover, active, focus-visible, loading skeleton, empty, error
- `DSN-15` Responsive: designed for >=1280px; collapses gracefully to 768px

**Suggested slices (one Antigravity prompt each):**
- 1a tokens + fonts + glass.css + primitives
- 1b shell (TopNav, Sidebar, PageHeader, Dock, AgentDrawer shell)
- 1c cards + charts (FolderCard, HeroMetric, FilterChips, BandChart, GaugeCard, BarsCard, Callout, DropZone)
- 1d DataGrid + states + /design-preview + screenshots

**Tasks:**
1. Copy the 6 reference images into `docs/reference/` as `ref-1..ref-6` (names in `design.md`). The agent must OPEN and LOOK at each image before coding.
2. `src/styles/tokens.css`, `glass.css`, `globals.css` exactly per `design.md` §1–3. Load Urbanist via `next/font/google` (300/400/500/600).
3. Primitives in `src/components/glass/`: `Surface`, `Chip`, `CircleButton`, `NavyPill`, `Badge`, `Callout`.
4. Shell in `src/components/shell/`: `TopNav`, `Sidebar` (tree content from `design.md` §5.2), `PageHeader` (back circle, title, filter input, Refresh, download, share, navy context pill), `Dock`, `AgentDrawer` (static shell + collapse behaviour; wired in P3).
5. Cards: `FolderCard` (peeking sheets + glow), `PlusTile`, `DropZone`, `HeroMetric` (thin number + 2x2), `FilterChips` (Radix Popover), `BandChart` (SVG hatch, dotted series, dashed grid, callout), `GaugeCard`, `BarsCard`.
6. `DataGrid` (react-table + virtual) with `EnrichmentCell`, `CitationChip`, `TierPill`, `ScoreBar` per `design.md` §5.12.
7. States: skeleton, empty, error components (`design.md` §7).
8. `/design-preview` page showing every component with sample props (props only; clearly titled 'Component preview', no fake business data).
9. Playwright script saves screenshots to `evidence/P1/` at 1440x900, 1280x800, 1024x768, 768x1024. Compare side-by-side to `ref-6`.
10. Tick every item of `design.md` §11 in the report with the screenshot path.

**Acceptance criteria:**
- [ ] `/design-preview` at 1440px is visually the same layout language as `ref-6`: gradient, glass, sidebar, tiles, folder cards, hero, chips, hatched chart, gauge, bars, dock.
- [ ] No hex colours inside component files (grep proves it). No italic text. No mac window / monitor / skew.
- [ ] All interactive controls work or are not rendered (no dead buttons). Sun icon is rendered **disabled with a tooltip 'Coming in P20'** until `ADD-09`.
- [ ] Keyboard focus ring visible on every control.
- [ ] lint/typecheck/test/build pass; component tests for BandChart geometry and DataGrid.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P2 — Database, auth, organizations, RLS, demo account, labeled seed data

- **Depends on:** P0, P1
- **Goal:** Real database, auth, organizations, security rules, a one-click demo login, and clearly labeled sample data.

**Requirements covered (all mandatory):**
- `ADD-01` Login (Supabase Auth) + ready demo account so judges do not sign up
- `ADD-02` Demo dataset mode, always labeled 'SAMPLE DATA' in UI; never mixed with real leads

**Suggested slices (one Antigravity prompt each):**
- 2a migrations + Drizzle schema + RLS
- 2b DB integrity triggers + tests
- 2c Supabase Auth + org bootstrap + login page + demo account
- 2d demo seed + SAMPLE DATA labeling

**Tasks:**
1. Create migrations from `schema.md` §2 (split across files, in order). Add indexes (§3). Create Drizzle schema in `src/lib/db/schema.ts`.
2. RLS policies on every table (§5). `audit_log` select-only.
3. Triggers/constraints in §4: lead-needs-verified-citation (deferred constraint trigger), `hitl_gate` check, `audit_log` immutability, suppression-list block.
4. `docker-compose.yml` local Postgres with pgvector for integration tests (`DATABASE_URL_TEST`).
5. Integration tests that PROVE each integrity rule (insert violating row => error).
6. Supabase Auth (email+password). On first login: create `organizations`, `org_members`, `credit_wallet` (demo credits), `founder_profile`.
7. `/login` page in glass style with **'Use demo account'** button that logs in via env demo credentials server-side (never expose password to the client).
8. `scripts/seed-demo.ts` + `db/seed/demo/*.json`: fictional companies (`.example` domains), people, signals, sources, citations pointing to **fixture source text stored in the seed** — all rows `is_demo=true`.
9. UI: `SAMPLE DATA` navy context pill in `PageHeader` and a `SAMPLE` badge on every demo row when demo rows are shown; toggle 'Include sample data'.

**Acceptance criteria:**
- [ ] All migrations apply cleanly on an empty DB and are idempotent to re-run via the migration tool.
- [ ] Integration tests prove: lead without verified citation fails; unapproved message cannot be `queued`; audit_log update/delete fails; suppressed email cannot be queued.
- [ ] RLS test: user A cannot read org B rows.
- [ ] Demo login works in one click; seed loads; sample data is labeled everywhere it appears.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P3 — Core infra: LLM adapter, jobs, live event feed (Supabase Realtime), metering hooks, quota guard, logging

- **Depends on:** P2
- **Goal:** The shared plumbing every module uses: LLM client, jobs, live event feed, metering hook, quota guard, cache, rate limiter, logging.

**Requirements covered (all mandatory):**
- `UI-03` Collapsible live 'Agent Thinking' panel: reasoning steps, graph traversals, validation logs
- `ADD-05` Job runner with progress + resumable steps (serverless-safe)
- `ADD-06` Quota guard for free-tier APIs (Tavily, Hunter, Apollo, LLM) with caching

**Suggested slices (one Antigravity prompt each):**
- 3a LLM client + embeddings + cost tracking
- 3b jobs + step runner + cache + quota + rate limiter
- 3c agent_events + Realtime + AgentDrawer wiring + meter()

**Tasks:**
1. `src/lib/llm/`: `LlmClient` per `architecture.md` §5 using AI SDK + Google provider; `generateObject` with Zod; retries (2x, backoff); timeouts; logs tokens + estimated cost per call to `agent_events` and `usage_events(llm_tokens)`.
2. Provider fallback via `LLM_FALLBACK_*` (OpenAI-compatible) — implemented and unit-tested with a fake server.
3. `src/lib/jobs/`: create job, `step` endpoint, cursor persistence, idempotent step contract, `/api/cron/jobs-tick`.
4. `quota.ts`, `cache.ts`, `rate-limit.ts` (token bucket) per §19; tests.
5. `src/lib/events/emit.ts`: `emit(orgId, runId, type, message, data)` inserts `agent_events`. Client hook `useAgentEvents(runId)` using Supabase Realtime with polling fallback.
6. Wire `AgentDrawer` to real events; footer shows tokens/cost of current run. Event row UI per `design.md` §5.15.
7. `src/lib/billing/meter.ts` + `assertCredits` (local ledger version; Flexprice sync arrives P17).
8. A tiny 'Test run' button in Settings → Developer that fires a real LLM call and shows its events in the drawer (proves the loop).

**Acceptance criteria:**
- [ ] A real LLM structured call returns Zod-valid JSON and shows tokens/cost in the drawer.
- [ ] Job step endpoint resumes correctly after a simulated timeout (test).
- [ ] Repeated search/LLM-embedding call is served from cache (test).
- [ ] Quota guard refuses at 90% (test). No secrets in logs (test).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P4 — Module 1a: Product intake (chat + structured form)

- **Depends on:** P3
- **Goal:** Founder describes the product in chat or form; system extracts structured intake and shows what is still missing.

**Requirements covered (all mandatory):**
- `INT-01` Interactive plain-language chat interface for product intake
- `INT-02` Structured form interface for product intake (same data as chat)
- `INT-03` Extract core product features, capabilities, technical constraints
- `INT-04` Extract existing customer profiles and reference wins
- `INT-05` Extract pricing frameworks, contract values, monetization models

**Suggested slices (one Antigravity prompt each):**
- 4a intake schema + form + persistence
- 4b chat agent + extraction + missing-fields checklist
- 4c UI polish in glass style + tests

**Tasks:**
1. Zod `IntakeData` schema exactly per `prd.md` §10.1. Store in `intake.data`.
2. `/intake` page: chat (left) and structured form (right) bound to the same record; edits in one update the other.
3. `/api/intake/chat`: LLM extracts into `IntakeData`, returns assistant follow-up question targeting the most important missing field, plus `missing_fields` and `completeness`.
4. Unknown stays `null` and is shown as 'missing'. No guessing.
5. Persist `intake_messages`. Emit `reason` events.
6. Show 'Generate ICP' button only when completeness ≥ configured threshold (default 0.7); disabled state explains why.

**Acceptance criteria:**
- [ ] Typing a plain-language product description fills features, capabilities, constraints, reference customers, pricing.
- [ ] Editing the form changes what the chat knows and vice versa.
- [ ] Unit tests with fixture conversations assert extraction shape and that unknown fields remain null.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P5 — Module 1b: ICP engine, confidence scores, expansion, accept/adjust/dismiss

- **Depends on:** P4
- **Goal:** Structured ICP with per-field confidence from a defined rubric, versioning, and expansion suggestions the founder can accept/adjust/dismiss.

**Requirements covered (all mandatory):**
- `ICP-01` ICP: target industries and vertical classifications aligned to NAICS/SIC codes
- `ICP-02` ICP: company size thresholds (headcount tiers and revenue bands)
- `ICP-03` ICP: buyer personas, functional titles, seniority levels
- `ICP-04` ICP: high-value trigger events (tech stack change, exec hire, funding)
- `ICP-05` Field-level confidence scores 0.00 to 1.00 on every inferred ICP field
- `ICP-06` ICP expansion: adjacent segments, emerging verticals, non-obvious uses, each with evidence-based reasoning
- `ICP-07` Founder can explicitly accept, adjust, or dismiss each expansion suggestion

**Suggested slices (one Antigravity prompt each):**
- 5a ICP generation + NAICS/SIC table + confidence calculator
- 5b ICP page UI (field cards, confidence rings, versions)
- 5c expansion segments + actions + approval of ICP

**Tasks:**
1. Bundle NAICS and SIC reference tables (`src/lib/icp/naics.ts`, `sic.ts`) sourced from the official lists; LLM must pick codes from the table (validate; reject unknown codes).
2. `generateIcp()` → LLM returns fields with `value, rationale, evidence[]`. Code computes confidence by the rubric in `architecture.md` §9.2 (unit-tested). Intake-evidence quotes are verified against intake text.
3. Fields: industries (NAICS+SIC), headcount tiers, revenue bands, personas (titles + seniority), trigger events (type + weight), geography.
4. Versioning: every edit creates a new `icp_versions` row; one `approved` version per product.
5. Expansion: ≥3 segments when evidence exists; each with reasoning + evidence source ids (can require a small web-research step using the search/scrape libs once P6 exists; until then evidence may come from intake + reference customers and must say so). Accept / Adjust (edit dialog) / Dismiss; persisted with `decided_by`.
6. UI: field cards with confidence ring + rationale popover; expansion cards with three actions; 'Approve ICP' navy pill.

**Acceptance criteria:**
- [ ] Every ICP field shows confidence with 2 decimals and rationale. No field lacks a score.
- [ ] Codes are real (test rejects an invented NAICS code).
- [ ] Accepted/adjusted expansions are saved and included in the data passed to discovery; dismissed ones are not.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P6 — Search, scraping, RAG store (sources, chunks, embeddings, SSRF guard)

- **Depends on:** P3
- **Goal:** Reliable, safe fetching and storing of public sources with embeddings for retrieval.

**Requirements covered (all mandatory):**
- `DIS-02` Ingest public company websites, regulatory filings, press statements, job postings

**Suggested slices (one Antigravity prompt each):**
- 6a search providers (Tavily, Serper) + cache
- 6b scraper (fetch+Readability) + SSRF guard + source typing
- 6c chunking + embeddings + retrieval
- 6d optional Playwright worker

**Tasks:**
1. `SearchProvider` for Tavily (primary) and Serper (fallback) with cache + quota; missing key => `NOT_CONFIGURED`.
2. `Scraper.fetchPage`: `fetch` with timeout, size cap (2MB), content-type check, robots.txt respected, Readability extraction, `content_hash`.
3. **SSRF guard**: block private/loopback/link-local IPs and non-http(s); resolve DNS then check IP; re-check after redirects (tests).
4. Source typing: `website | filing | press | job_posting | other` by URL/domain/content rules (e.g. sec.gov, mca.gov.in, press-release domains, greenhouse/lever/workable job boards).
5. Chunk (800 tokens, 100 overlap) + embed + insert `source_chunks`; `retrieve(orgId, query, k)`.
6. Optional: `worker/` Dockerfile with Playwright for JS-heavy pages, called over HTTP with a shared secret. If not built, report `SPEC GAP: PLAYWRIGHT WORKER NOT BUILT (OPTIONAL PER ARCHITECTURE D6)`.

**Acceptance criteria:**
- [ ] Search + fetch + store + retrieve works on a real public page and emits events.
- [ ] SSRF tests pass for 127.0.0.1, 169.254.169.254, 10.x, localhost, DNS-rebinding style redirects.
- [ ] Prompt-injection fixture page does not change behaviour (see `security.md` §4 test).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P7 — Module 2a: Buyer discovery with verified citations and negative-signal filter

- **Depends on:** P5, P6
- **Goal:** Find real accounts and role-matched buyers with verified citations; filter negative signals; show them in the lead grid. THE core promise of the product.

**Requirements covered (all mandatory):**
- `DIS-01` Identify prospective corporate accounts based on the approved ICP
- `DIS-03` Every prospect has mandatory inline source citations (quote + URL), verified against fetched page text
- `DIS-04` Identify role-matched buyers (people) at each account
- `DIS-05` Filter prospects against explicit negative signals (hiring freeze, non-target region)

**Suggested slices (one Antigravity prompt each):**
- 7a query planner + extraction prompt + candidate schema
- 7b citation verifier (pure code) + tests
- 7c negative-signal filter + lead/people persistence
- 7d leads grid + lead detail + discovery run dialog (budget)

**Tasks:**
1. Run dialog: pick ICP version, budget (queries/pages/candidates), start job.
2. Query planner: LLM → ≤N queries from ICP + accepted expansions. Emit `search` events.
3. Extractor: LLM reads retrieved chunks (as DATA) and returns candidates `{company, domain?, person?, title?, signal?, evidence_quote, source_url}`.
4. **CitationVerifier** per `architecture.md` §9.1. Only verified candidates become leads. Rejected ones are logged (`cite_reject`) with reason.
5. Negative signals per §9.7; excluded leads kept with `exclusion_reason` + source.
6. Create `companies`, `people`, `leads` (stage `matched`, actor `ai`), `lead_citations`, `pipeline_events`, `audit_log`.
7. Leads grid: dense DataGrid with company, person, title, citation chip, tier placeholder, status; filters; 'Excluded' toggle; lead detail page with citations (quote + URL + verified tick).
8. Role-matched people: title similarity to ICP personas (embedding + keyword rule); stored with source.

**Acceptance criteria:**
- [ ] Run on a real sample product yields leads where **every** lead has ≥1 `verified=true` citation (SQL query pasted in report).
- [ ] A planted fake candidate (test fixture) is rejected.
- [ ] Excluded leads appear under the toggle with reason.
- [ ] No lead appears without a source (UI test).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P8 — Module 2b: Competitor discovery and landscape mapping

- **Depends on:** P6
- **Goal:** Competitor landscape with positioning, value props, public pricing, and cited differentiators.

**Requirements covered (all mandatory):**
- `COM-01` Map competitors into direct, indirect, and alternative categories
- `COM-02` Extract competitor market positioning and core value propositions
- `COM-03` Evaluate publicly accessible competitor pricing tiers and packaging
- `COM-04` Summarize structural advantages and technical differentiators vs our product

**Suggested slices (one Antigravity prompt each):**
- 8a competitor discovery + categories
- 8b positioning/pricing extraction with verification
- 8c comparison UI

**Tasks:**
1. Discover competitors from product description + search; classify `direct | indirect | alternative` with reasoning.
2. Extract positioning and value props from each competitor's own pages; store `competitor_claims` (claim + quote + source + verified via §9.1).
3. Pricing: extract public tiers/packaging; if none public, store `is_public=false` and show 'Pricing not public'.
4. Advantages/differentiators relative to our product — each claim cited; unverified claims dropped.
5. `/competitors` UI: three lanes + comparison panel (glass).

**Acceptance criteria:**
- [ ] Each competitor shows category, positioning, value props, pricing-or-'not public', differentiators — all with source links.
- [ ] An uncited claim cannot be saved (test).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P9 — Module 3a: Knowledge graph (Neo4j schema, sync, graph view, warm-intro paths)

- **Depends on:** P7, P8
- **Goal:** Neo4j graph with the five spec labels and six spec relationships, projected from Postgres, plus warm-intro path finding and a graph view.

**Requirements covered (all mandatory):**
- `GRA-01` Native graph store with node labels Company, Person, Product, Competitor, Signal
- `GRA-02` Directional edges works_at, competes_with, matched_to, contacted_via, replied_to, previously_employed_at
- `GRA-03` Multi-hop query: warm introduction paths via shared employment or education

**Suggested slices (one Antigravity prompt each):**
- 9a Neo4j client + constraints + sync/rebuild
- 9b real writes for all six spec relationships
- 9c warm-intro query + founder profile UI
- 9d graph view page

**Tasks:**
1. `src/lib/graph/neo4j.ts` (driver, session helper, timeouts) and `sync.ts` (idempotent MERGE).
2. Apply `graph/constraints.cypher`. `scripts/rebuild-graph.ts`.
3. Write nodes: Company, Person, Product, Competitor, Signal (+School). Write edges: works_at, previously_employed_at, competes_with, matched_to; `contacted_via` and `replied_to` are written when outreach events happen (P12–P14) — add the write functions now and unit test them.
4. Founder profile in Settings (employers/schools) → founder Person node + history edges.
5. Named query registry (`warm_paths`, `neighbourhood` now; others later) — client sends names only.
6. `/graph` page: force graph (react-force-graph-2d), path finder (pick lead), results panel, Cypher shown in the Agent Thinking drawer.
7. Each query emits `graph` event + `meter('graph_queries_evaluated')`.

**Acceptance criteria:**
- [ ] Neo4j contains Company/Person/Product/Competitor/Signal nodes for the run (counts in report).
- [ ] A warm path is found for a seeded founder+target pair and displayed; with no overlap the UI says none found.
- [ ] Rebuild from Postgres reproduces the same node/edge counts.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P10 — Module 3b: Signal ingestion, graph reasoning queries, Fit & Intent scoring

- **Depends on:** P9
- **Goal:** Signals flow in periodically; graph reasoning queries run; every lead gets a Fit & Intent score and Tier.

**Requirements covered (all mandatory):**
- `GRA-04` Link newly discovered job requisitions to likely active buyer evaluation cycles
- `GRA-05` Map competitor displacement opportunities from renewal and churn signals
- `GRA-06` Continuous signal ingestion that periodically updates node properties and relationship states
- `UI-04` Composite Fit & Intent score 0-100 with Tier A/B/C

**Suggested slices (one Antigravity prompt each):**
- 10a signal extraction + storage + graph sync
- 10b cron + manual refresh + last-refreshed UI
- 10c job-req and displacement queries + UI
- 10d scoring engine + UI

**Tasks:**
1. Signal types per schema. Extract from fetched sources (jobs, press, filings) with quotes verified (§9.1).
2. `/api/cron/signals` (secret header) + `/api/signals/refresh`: re-fetch watched companies, upsert signals, update graph node properties and relationships; show `last_refreshed`.
3. `USES` edges for competitor customers (needs source + quote).
4. Queries Q2 `job_req_evaluations` and Q3 `displacement` per `schema.md` §6.5; UI cards with evidence links. `likely_evaluating` flag written to `leads`.
5. Scoring engine per `architecture.md` §9.3 in `src/lib/scoring/` with config file + unit tests for each component and tier boundaries (74/75, 49/50).
6. UI: score bar + tier pill in grid; breakdown popover on hover; re-score on new signals.

**Acceptance criteria:**
- [ ] After a refresh, a new signal changes a lead's intent score and the graph (before/after in report).
- [ ] Job-req and displacement lists render with evidence and 'likely' wording (no certainty claims).
- [ ] Tier boundaries tested.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P11 — Module 4a: Enrichment pipeline (4 vendors, MX check, inline indicators)

- **Depends on:** P7
- **Goal:** Contact enrichment through four vendor adapters with an honest waterfall, own MX validation, and inline status indicators.

**Requirements covered (all mandatory):**
- `ENR-01` Resolve primary business email with deliverability and MX validation status
- `ENR-02` Resolve validated personal LinkedIn URL
- `ENR-03` Resolve verified corporate domain record and headquarters location
- `ENR-04` Programmatic adapters for Hunter.io, Apollo, Clearbit, Prospeo (all four exist; unconfigured ones report not_configured, never fake data)
- `UI-02` Inline enrichment indicators and confidence scores inside data grids

**Suggested slices (one Antigravity prompt each):**
- 11a adapter interface + Hunter + Apollo
- 11b Clearbit + Prospeo adapters + not_configured behaviour
- 11c waterfall + MX check + persistence
- 11d UI indicators + enrichment page

**Tasks:**
1. `EnrichmentProvider` implementations: Hunter, Apollo, Clearbit, Prospeo. Read each vendor's CURRENT API docs first; record endpoints used in `progress.md`.
2. No key => `not_configured` status. Vendor failure => `error` with message. **Never fabricate or pattern-guess emails and label them verified.** (Pattern guesses, if offered, must be `unverified`.)
3. Own MX check using `node:dns` `resolveMx` for every email domain; store `mx_valid`.
4. Waterfall order configurable; stop when field verified; cache results (30 days); quota guard.
5. Resolve: email + deliverability, LinkedIn URL, domain, HQ location. Store in `enrichment_results` + update `companies/people`.
6. Each lead enriched => `meter('leads_enriched')`; `assertCredits('enrich')` before the call.
7. UI: `EnrichmentCell` inline in grid (dot + label + confidence); `/enrichment` page with provider status cards; bulk 'Enrich selected'.

**Acceptance criteria:**
- [ ] With only Hunter configured, results show Hunter data and the other three as `not_configured` (screenshot).
- [ ] A made-up domain yields `not_found`/MX invalid, not a fake email.
- [ ] Grid shows status + confidence for each enriched field.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P12 — Module 4b: Outreach synthesis + HITL approval gateway

- **Depends on:** P10, P11
- **Goal:** Signal-personalised multi-touch sequences, validators, a Reviewer check, and a strict approval queue enforced in code AND database.

**Requirements covered (all mandatory):**
- `OUT-01` Multi-touch outreach sequences across Email and LinkedIn
- `OUT-02` Weave specific company trigger signals and value props into copy
- `OUT-03` Obey persona-specific tone, length, and CTA constraints
- `OUT-04` HITL gateway: reps can approve, edit, or reject copy from a queue
- `OUT-05` Nothing is queued or sent without approval (enforced in server code AND database)

**Suggested slices (one Antigravity prompt each):**
- 12a templates/personas/value props + validator
- 12b sequence planner + drafter + reviewer
- 12c HITL queue UI + approve/edit/reject API
- 12d DB-level gate tests + A/B variant ids + graph edge writes

**Tasks:**
1. Seed persona rules (tone, max words, CTA) in `templates`/Settings; `value_props` from ICP/intake.
2. Sequence default per `prd.md` §10.6; each step has `variant_id = template:channel:valueprop`.
3. Drafter prompt injects: lead facts, real signals (with ids), persona rules. Output Zod-validated.
4. Validator (code): word limit, one CTA, banned phrases, ≥1 signal referenced, no unverifiable numeric claims. Violations → regenerate (max 2) then flag.
5. Reviewer step (LLM, no tools): every factual claim must map to a source in the lead's sources; produces `risk_flags`.
6. HITL queue (`/outreach`): list + detail per `design.md` §5.14. Actions Approve / Edit & Approve / Reject(reason). Store `approvals` (before/after).
7. Edit after approval resets status to `needs_review` and clears approval (same UPDATE).
8. API gate: `dispatch` refuses anything not approved with matching hash. DB `hitl_gate` is the second wall.
9. Write `contacted_via` / `replied_to` graph edges when those events occur (functions from P9).

**Acceptance criteria:**
- [ ] Tests: (1) API refuses to dispatch unapproved message, (2) direct SQL update to `queued` without approval FAILS, (3) editing after approval invalidates it.
- [ ] Each message highlights the signals it used and links to their sources.
- [ ] Sequence has ≥3 steps across email and LinkedIn.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P13 — Module 4c: Messaging layer (dispatch modes, webhooks, rate limits, compliance)

- **Depends on:** P12
- **Goal:** Safe message delivery: dry_run by default, sandbox to owned addresses, live behind explicit switches; webhooks; rate limits; compliance.

**Requirements covered (all mandatory):**
- `OUT-06` Messaging layer: delivery orchestration, webhook feedback loop, rate limiting (SendGrid / Gmail API / LinkedIn)
- `ADD-03` Dispatch modes dry_run (default) / sandbox / live, controlled by env + org setting
- `ADD-04` Compliance: unsubscribe link, suppression list, physical-address footer, no demo sends to real prospects

**Suggested slices (one Antigravity prompt each):**
- 13a provider interface + dry_run + SMTP/Gmail
- 13b SendGrid adapter + webhook verification
- 13c suppression + unsubscribe + footer + rate limit
- 13d LinkedIn manual-task flow + status sync

**Tasks:**
1. `MessagingProvider`: `dry_run` (writes status `sent` with provider `dry_run`, no network), `smtp` (nodemailer), `gmail` (OAuth), `sendgrid` (optional). Mode from env + org setting; `live` requires both and a typed confirmation in UI.
2. `sandbox` mode only sends to `SANDBOX_ALLOWED_RECIPIENTS`.
3. Rate limiting per provider and per domain (token bucket) + daily cap setting.
4. Compliance (`ADD-04`): footer with sender physical address, one-click unsubscribe link (signed token), suppression list honoured, List-Unsubscribe header.
5. `/api/webhooks/email`: verify signature; map events delivered/open/click/bounce/spam/unsubscribe; update message + `email_events`; bounce/unsubscribe → suppression.
6. Reply detection: webhook where supported; else 'Mark replied' rep action; (Gmail API polling optional). Each reply → `replied` stage + graph `REPLIED_TO`.
7. LinkedIn steps: status `manual_pending`; UI button 'Open profile + copy message'; rep clicks 'Mark sent' → `manual_sent`, actor `rep`.
8. On send: `meter('outreach_sent')` + `cost_ledger` row.
9. README note (final in P22): LinkedIn automation is intentionally not done.

**Acceptance criteria:**
- [ ] dry_run: approved message flows to `sent` without any network call (test asserts no outbound request).
- [ ] Suppressed address is blocked (test). Unsubscribe link works and suppresses.
- [ ] Webhook with bad signature is rejected (test); good one updates status.
- [ ] `live` cannot be enabled without the explicit steps.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P14 — Module 5a: Kanban pipeline, attribution, audit log

- **Depends on:** P13
- **Goal:** Kanban pipeline with visible AI-vs-rep attribution and a complete audit trail. DEMO CUT LINE: after this phase the whole pipeline runs end-to-end.

**Requirements covered (all mandatory):**
- `PIP-01` Interactive Kanban: Matched -> Contacted -> Replied -> Meeting Booked -> Closed/Lost
- `PIP-02` Distinguish AI-driven progression from human rep actions
- `PIP-03` Execution timestamps and audit records for every outbound interaction

**Suggested slices (one Antigravity prompt each):**
- 14a pipeline move API + events + audit
- 14b Kanban UI (dnd-kit) + actor chips
- 14c lead timeline + dashboard hero cards wired to real data

**Tasks:**
1. `/api/pipeline/move`: validates allowed transitions, writes `pipeline_events` + `audit_log` with `actor_type`.
2. AI actions: auto `matched`, `contacted` on send, `replied` on webhook (actor=system or ai). Rep actions by drag/drop (actor=rep, with user id). Closed has Won/Lost sub-lanes.
3. Kanban per `design.md` §5.13; keyboard-accessible; optimistic update with rollback on error.
4. Lead timeline: every outbound interaction with timestamp + actor + provider id (`PIP-03`).
5. Wire dashboard: `HeroMetric`, `BandChart`, `GaugeCard`, `BarsCard`, folder cards ← real queries (empty states when no data; sample data labeled).
6. **End-to-end smoke run by the agent** (manual script): intake → ICP → discovery → enrich → generate → approve → dispatch(dry_run) → Kanban shows contacted. Paste evidence.

**Acceptance criteria:**
- [ ] Every stage change shows actor chip AI/Rep and appears in the timeline with a timestamp.
- [ ] `audit_log` has a row per outbound interaction (SQL count in report).
- [ ] The end-to-end smoke run succeeds.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

> ### ⭐ DEMO CUT LINE — after P14 the end-to-end pipeline exists. If time runs short, everything after is deepening, bonus, and delivery. Do NOT skip P20–P22 (proof + deployment + deliverables).

---
## P15 — Module 5b: Analytics, split-testing, CSV export

- **Depends on:** P14
- **Goal:** Analytics that answer: where do leads drop off, by which segments, and which message variants win.

**Requirements covered (all mandatory):**
- `ANA-01` Conversion rate progression across stages, segmented by industry and company tier
- `ANA-02` Split-test analytics across message templates, channels, and value propositions
- `ADD-10` CSV export of leads and analytics

**Suggested slices (one Antigravity prompt each):**
- 15a funnel + segment queries
- 15b variant analytics
- 15c UI + CSV export

**Tasks:**
1. Funnel counts + stage-to-stage conversion; segment filters industry and tier (+ date range, + include-sample toggle).
2. Variant analytics by template, channel, value prop: sent, replied, rate, n; Wilson interval; 'too few samples' under 30 sends.
3. UI in glass style: funnel, `BandChart` trend, variant table, filters via `FilterChips`.
4. CSV export (`ADD-10`) for leads and variant table.

**Acceptance criteria:**
- [ ] Numbers match SQL (agent pastes the verifying query + result).
- [ ] Segment filters change the funnel. CSV matches the grid.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P16 — Integration pass: end-to-end flow, Agent Thinking wired everywhere

- **Depends on:** P15
- **Goal:** Make the whole product feel like one system: wiring gaps closed, Agent Thinking everywhere, no dead ends.

**Requirements covered (all mandatory):**
- (infrastructure phase, no spec IDs)

**Suggested slices (one Antigravity prompt each):**
- 16a cross-module links + empty/error states
- 16b Agent Thinking coverage audit
- 16c Home/dashboard polish + navigation badges

**Tasks:**
1. Every module emits events of the right type (audit table: module → event types → proof).
2. Sidebar badges real (pending suggestions, new leads, pending approvals).
3. Cross-links: lead ↔ citations ↔ graph ↔ messages ↔ timeline.
4. Fix every dead button/placeholder text found by a crawl (`grep` for TODO/placeholder/coming soon; list in report).
5. Saved views (folder cards) persist to `saved_views` and re-apply filters; dropping a filter chip on the drop zone saves a segment.

**Acceptance criteria:**
- [ ] Fresh run of the demo path touches every module without manual DB edits.
- [ ] Grep for `TODO|FIXME|placeholder|coming soon|lorem` returns only allow-listed lines (P20 theme tooltip).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P17 — BONUS: Flexprice metering and credit gating

- **Depends on:** P16
- **Goal:** BONUS: Flexprice usage metering and credit-gated enrichment.

**Requirements covered (all mandatory):**
- `FLX-01` Flexprice metered events: Leads Enriched, Outreach Sent, Graph Queries Evaluated
- `FLX-02` Credit enforcement: gate enrichment when org balance is exhausted
- `FLX-03` Cost allocation: per-lead outreach cost mapped to the billing ledger

**Suggested slices (one Antigravity prompt each):**
- 17a read Flexprice docs + client + customer/meter setup
- 17b event sync + wallet check + 402 gate
- 17c /usage page + cost allocation

**Tasks:**
1. READ the current Flexprice API docs/Swagger first; write real endpoints, auth header, and wallet/entitlement calls into `progress.md`. If unclear, STOP and ask the human.
2. Create customer per org (`external_customer_id=org_id`), features/meters: `leads_enriched`, `outreach_sent`, `graph_queries_evaluated`, and a credit wallet.
3. Background sync of `usage_events` with `event_id = usage_events.id`, RFC3339 timestamps, retry, idempotency.
4. `assertCredits` reads wallet balance (30s cache). Balance ≤ 0 → enrichment returns HTTP 402 `CREDITS_EXHAUSTED`; UI dialog explains and offers demo top-up (admin).
5. Cost allocation (`cost_ledger`) per lead; lead detail shows cost; `/usage` page: gauge, events, ledger.
6. Proof: Flexprice dashboard screenshots showing all three events and the gate triggering.

**Acceptance criteria:**
- [ ] Three event types visible in Flexprice. Gate blocks at zero and unblocks after top-up.
- [ ] If Flexprice is down, user actions still work (except credit check shows 'unverified balance') — tested.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P18 — BONUS: Band multi-agent orchestration

- **Depends on:** P16
- **Goal:** BONUS: four-role multi-agent orchestration with @mentions, HITL gates, telemetry; connected to real Band if possible.

**Requirements covered (all mandatory):**
- `BND-01` Band: Planner/Architect agent role
- `BND-02` Band: Builder agent role (API calls, graph mutations, outreach drafting)
- `BND-03` Band: Reviewer agent role (audits copy, fact checks, validates source links)
- `BND-04` Band: Security agent role (scans endpoints, data exposure, outbound links)
- `BND-05` Handoffs between agents using @mentions
- `BND-06` Strict HITL control on schema modifications and outreach dispatch
- `BND-07` Operational telemetry: shared message stream, active session tokens, cumulative cost

**Suggested slices (one Antigravity prompt each):**
- 18a in-app orchestrator + roles + message log
- 18b HITL approval cards for schema change + dispatch
- 18c telemetry UI (/agents)
- 18d Band adapter (after reading docs)

**Tasks:**
1. Orchestrator per `architecture.md` §12: roles Planner/Architect, Builder, Reviewer, Security; `@mention` parser routes turns; `agent_runs`, `agent_messages`, `agent_approvals`.
2. A founder prompt ('Find and reach fintech CFOs in India') → Planner plan → Builder tasks via real services → Reviewer audit → Security link/endpoint check.
3. Reviewer can reject and send back (`@Builder`). Security can block a link not on allowlist.
4. HITL cards: schema-change proposals and outreach dispatch cannot proceed until a human approves (tested).
5. `/agents` UI: shared message stream, active session tokens, cumulative USD cost, approval cards.
6. Band: read current Band docs/SDK; record connection method in `progress.md`; register the four agents; mirror the stream. Real credentials come from the human.
7. If Band connection fails: report `SPEC GAP: BND — REAL BAND CONNECTION NOT ACHIEVED` (capitals) and keep layer 1.

**Acceptance criteria:**
- [ ] Demo run shows: @mention handoffs, a Reviewer rejection, a Security flag, a human approval.
- [ ] Telemetry numbers equal the sum of `agent_messages` rows (test).
- [ ] Band status in `/api/health` is truthful.

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P19 — BONUS: Snyk SAST + SCA in CI/CD

- **Depends on:** P13
- **Goal:** BONUS: SAST + SCA in CI/CD, results visible, high severity blocks.

**Requirements covered (all mandatory):**
- `SNK-01` Snyk vulnerability scanning on outbound data-dispatching services
- `SNK-02` SAST and SCA embedded in CI/CD pipeline

**Suggested slices (one Antigravity prompt each):**
- 19a Snyk CI jobs
- 19b fix findings + document

**Tasks:**
1. Enable the Snyk steps already present in `ci.yml` (needs `SNYK_TOKEN` secret from the human). Add `snyk code test` (SAST) and `snyk test` (SCA) with `--severity-threshold=high`.
2. Scan outbound services explicitly (`src/lib/messaging`, `enrichment`, `search`) and list them in README.
3. Fix or document every high finding. Add gitleaks (secret scan) job.
4. README section 'Security scanning' with a screenshot of a passing run.

**Acceptance criteria:**
- [ ] CI shows both Snyk jobs green (or findings fixed) on `main`.
- [ ] A deliberately vulnerable dependency on a throwaway branch fails the build (proof screenshot).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P20 — Hardening: citation eval harness, e2e tests, a11y, theme toggle, perf

- **Depends on:** P18, P17, P16
- **Goal:** Hardening: proof of no hallucinated leads, full e2e test, accessibility, performance, theme toggle.

**Requirements covered (all mandatory):**
- `ADD-08` Citation evaluation harness proving no hallucinated leads
- `ADD-09` Light/dark theme toggle (sun icon in sidebar must work)
- `ADD-11` End-to-end Playwright test of full pipeline (intake -> dashboard)

**Suggested slices (one Antigravity prompt each):**
- 20a eval harness
- 20b Playwright e2e
- 20c dark theme + toggle
- 20d a11y + perf + cleanup

**Tasks:**
1. `scripts/eval-citations.ts`: run discovery on 3 sample products, output verified/total (must be 100%), list rejected candidates and reasons, save `evidence/P20/eval.json`.
2. Playwright e2e (`ADD-11`) of the whole path with fixture providers; runs in CI.
3. Dark theme: remap tokens only (no component edits); persist choice; sun/moon toggle works.
4. a11y: axe checks on main pages, focus order, contrast per `design.md` §10. Perf: Lighthouse on `/leads` and `/dashboard`; fix obvious issues.
5. Remove dead code, unused deps, console logs; update `progress.md`.

**Acceptance criteria:**
- [ ] Eval prints `verified_citations=N/N`.
- [ ] e2e green in CI.
- [ ] Theme toggle works; no dead buttons (crawl).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P21 — Deployment (Vercel + Supabase + Neo4j Aura), demo dataset, health page

- **Depends on:** P20
- **Goal:** Public deployment judges can open and use in minutes.

**Requirements covered (all mandatory):**
- `DEL-01` Deployed end-to-end prototype (intake -> dashboard), public URL

**Suggested slices (one Antigravity prompt each):**
- 21a Vercel + Supabase + Aura setup
- 21b prod seed + demo account + health

**Tasks:**
1. Deploy to Vercel; set env vars; apply migrations to prod Supabase; apply graph constraints; Realtime enabled for `agent_events`.
2. `APP_MODE=demo` seeds labeled sample data and demo account; real mode available too.
3. Wake/verify Neo4j Aura; `/api/health` all green (or honestly amber).
4. Smoke test on the public URL (agent pastes URL + results). Confirm `DISPATCH_MODE=dry_run` in production.
5. Backup demo data + a 'Reset demo' admin button.

**Acceptance criteria:**
- [ ] Public URL runs the full demo path from the demo login.
- [ ] Health page truthful.
- [ ] No secrets in client bundle (grep build output).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.

---
## P22 — Deliverables: README, system design doc, viability brief, demo script, final audit

- **Depends on:** P21
- **Goal:** Everything the judges ask for besides running code.

**Requirements covered (all mandatory):**
- `DEL-02` Public GitHub repo with setup docs and modular architecture
- `DEL-03` 2-3 minute demo video: intake, graph matching, message composition, HITL review, dashboard sync
- `DEL-04` System design document: query logic, graph ontology, reasoning loops, bonus services
- `DEL-05` Startup viability brief: problem, buyer, differentiation vs existing tools, pricing, unit economics (15% of score)

**Suggested slices (one Antigravity prompt each):**
- 22a README + architecture diagram
- 22b system-design.md
- 22c demo script + video checklist
- 22d final spec audit

**Tasks:**
1. README: what it is, screenshots, quickstart (fresh clone → running), env table, architecture diagram, security scanning, LinkedIn note, limitations (honest).
2. `docs/system-design.md`: query logic, graph ontology, reasoning loops (discovery, outreach, scoring), bonus services; derived from the real code.
3. Review `docs/startup-viability.md` with the human; make it specific and honest.
4. `docs/demo-script.md` rehearsal with timestamps; video recorded by the human (agent prepares seed state + checklist).
5. **Final spec audit**: every row in `SPEC_CHECKLIST.md` is DONE with evidence or `SPEC GAP` with reason. Agent lists any gap in CAPITALS.

**Acceptance criteria:**
- [ ] Fresh clone runs from README alone (agent proves in a clean directory).
- [ ] Checklist has zero blank statuses.
- [ ] Video length 2:00–3:00 (human confirms).

**Report must include:** raw command output (lint/typecheck/test/build), test names for each new behaviour, screenshots for UI, SQL/Cypher results where stated, and a per-ID status table for this phase.
