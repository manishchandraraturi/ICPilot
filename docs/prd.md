# ICPilot — Product Requirements Document (PRD)

> Status: v1.0 · Owner: Capt · Last updated: 2026-11-24
> This file is LAW for WHAT we build. `architecture.md` is law for HOW. `design.md` is law for how it LOOKS.
> If anything here conflicts with a hackathon spec line, the spec wins and this file must be fixed.

---

## 0. Reading rules (for every AI agent)

1. Every requirement has an ID (e.g. `ICP-05`). The full list is in section 9 and in `SPEC_CHECKLIST.md`.
2. Do not skip, rename, merge, or "simplify" a requirement. If you cannot finish one, write it in your report as `SPEC GAP: <ID> — <reason>` in CAPITALS. Silence is treated as a lie.
3. Do not invent data. No hardcoded leads, emails, companies, metrics, or reviews outside the labeled demo seed (`ADD-02`).
4. `SRC` = from the hackathon spec. `ADD` = our addition. `DSN` = design fidelity. `DEL` = deliverable.

---

## 1. One-line pitch

**ICPilot is one system that takes a plain-language product description and runs the whole B2B go-to-market loop: infer the ideal customer, find real companies and buyers with proof, map them in a knowledge graph, write personalised outreach, make a human approve it, and track every lead to a closed deal.**

## 2. Problem (from the spec, plain English)

Early-stage B2B teams run sales across five disconnected places:

| Step | Where it lives today | What breaks |
|---|---|---|
| Who to sell to (ICP) | Slide decks | Goes stale, nobody uses it |
| Finding prospects | Google searches, big databases | Slow, noisy, no proof |
| Contact details | Separate enrichment tools | Copy-paste, wrong data |
| Outreach | Personal inboxes, LinkedIn | Generic copy, no tracking |
| Pipeline | Spreadsheets | No attribution, no learning |

Result: context is lost at every handoff → wrong targets, generic messages, high customer acquisition cost (CAC).

## 3. Who it is for

| Persona | Needs | How ICPilot helps |
|---|---|---|
| **Founder / first sales hire** (primary) | Find the right buyers fast with almost no sales-ops budget | Describes product once; gets ICP, cited leads, ready-to-approve outreach |
| **SDR / sales rep** | Stop doing research and copy-paste | Works from one HITL queue and one Kanban board |
| **Sales manager** | See what works | Funnel, tier, and A/B variant analytics with AI-vs-human attribution |
| **Prospect** (never logs in) | Not being spammed | Messages cite real signals; unsubscribe works; suppression list honoured |

## 4. Product vision (from the spec)

An autonomous, closed-loop system that:
1. Ingests plain-language product descriptions and commercial criteria
2. Infers and refines target buyer profiles
3. Discovers validated accounts and verified decision-makers grounded in public evidence
4. Connects entities, org histories, and real-time buying signals in a Knowledge Graph
5. Writes and (after human approval) dispatches multi-channel outreach
6. Centralises lead progression and attribution analytics in one dashboard

## 5. Product shape

- A **web app** (not a phone app, not a marketing site). Next.js, runs in the browser.
- One login. One role in the MVP (member of an organization). No team/permission builder (not scored).
- Judges get a ready **demo account** (`ADD-01`). They never have to sign up.
- Desktop-first (≥1280px). See `DSN-15`.

## 6. Principles (non-negotiable)

1. **Evidence or nothing.** A lead, competitor claim, or signal without a verified source does not exist in the product.
2. **Human decides before anything leaves.** No message is queued or sent without approval, enforced in code and database.
3. **Honest about uncertainty.** Every inferred field shows a confidence score. Missing data says "not found", never a guess.
4. **Real or labeled.** Real data from real APIs, or clearly labeled `SAMPLE DATA`. Never mixed.
5. **Safe by default.** Dispatch defaults to `dry_run`. Live sending needs an explicit switch.
6. **One flow, no silos.** Every module reads and writes the same records; nothing is re-typed.

## 7. Core user journey (the demo path)

1. **Intake**: Founder describes product in chat (or form). System extracts features, constraints, reference customers, pricing.
2. **ICP**: System shows structured ICP with confidence per field and expansion segments. Founder accepts / adjusts / dismisses.
3. **Discovery**: System finds real companies and role-matched buyers, each with a verified citation. Negative signals filtered. Competitors mapped.
4. **Graph**: Entities, people, signals, competitors go into the knowledge graph. Warm-intro paths and job-req/churn insights appear.
5. **Score**: Each lead gets Fit & Intent score (0–100) and Tier A/B/C.
6. **Enrich**: Emails (+MX check), LinkedIn URLs, domain, HQ resolved from vendors.
7. **Outreach**: Multi-touch Email + LinkedIn sequences drafted using the lead's real signals.
8. **HITL**: Rep approves / edits / rejects in a queue. Only approved messages can go out.
9. **Track**: Leads move across the Kanban; every move shows AI vs rep; audit log records all.
10. **Learn**: Analytics show funnel by industry/tier and A/B results by template, channel, value prop.

## 8. MVP scope vs full scope

**There is no "cut" of spec requirements.** Everything in section 9 ships. What we control is **build order** so a demoable end-to-end slice exists early.

- **Demo-ready cut line = end of Phase 14.** After P14 the full pipeline (intake → Kanban) runs end-to-end. P15–P22 deepen and harden.
- Bonuses (Flexprice, Band, Snyk) are scored extra. Build them after the core works (P17–P19).

## 9. Requirements (generated from `tools/requirements.py` — do not hand-edit)

<!--REQ_TABLE-->

| ID | Group | Requirement | Phase | Verified by |
|---|---|---|---|---|
| `INT-01` | SRC M1 | Interactive plain-language chat interface for product intake | P4 | Chat UI works; messages stored; agent asks follow-up questions |
| `INT-02` | SRC M1 | Structured form interface for product intake (same data as chat) | P4 | Form and chat edit the same intake record |
| `INT-03` | SRC M1 | Extract core product features, capabilities, technical constraints | P4 | Structured intake JSON has features[], capabilities[], constraints[] |
| `INT-04` | SRC M1 | Extract existing customer profiles and reference wins | P4 | Intake JSON has reference_customers[] with outcome text |
| `INT-05` | SRC M1 | Extract pricing frameworks, contract values, monetization models | P4 | Intake JSON has pricing{model, tiers[], acv_range} |
| `ICP-01` | SRC M1 | ICP: target industries and vertical classifications aligned to NAICS/SIC codes | P5 | ICP has industries[] each with naics_code and sic_code |
| `ICP-02` | SRC M1 | ICP: company size thresholds (headcount tiers and revenue bands) | P5 | ICP has headcount_tiers[] and revenue_bands[] |
| `ICP-03` | SRC M1 | ICP: buyer personas, functional titles, seniority levels | P5 | ICP has personas[] with titles[] and seniority |
| `ICP-04` | SRC M1 | ICP: high-value trigger events (tech stack change, exec hire, funding) | P5 | ICP has trigger_events[] with type and weight |
| `ICP-05` | SRC M1 | Field-level confidence scores 0.00 to 1.00 on every inferred ICP field | P5 | Every ICP field has confidence (2 decimals) and rationale; UI shows it |
| `ICP-06` | SRC M1 | ICP expansion: adjacent segments, emerging verticals, non-obvious uses, each with evidence-based reasoning | P5 | expansion_segments[] each has evidence_source_ids[] and reasoning |
| `ICP-07` | SRC M1 | Founder can explicitly accept, adjust, or dismiss each expansion suggestion | P5 | Three working actions; status persisted; accepted ones feed discovery |
| `DIS-01` | SRC M2 | Identify prospective corporate accounts based on the approved ICP | P7 | Discovery run uses the approved ICP version id |
| `DIS-02` | SRC M2 | Ingest public company websites, regulatory filings, press statements, job postings | P6 | Source types: website, filing, press, job_posting all supported and tagged |
| `DIS-03` | SRC M2 | Every prospect has mandatory inline source citations (quote + URL), verified against fetched page text | P7 | Lead without a verified citation cannot be saved; eval harness proves it |
| `DIS-04` | SRC M2 | Identify role-matched buyers (people) at each account | P7 | People rows with title matching ICP personas, each with a source |
| `DIS-05` | SRC M2 | Filter prospects against explicit negative signals (hiring freeze, non-target region) | P7 | Excluded leads kept with exclusion_reason; visible in UI toggle |
| `COM-01` | SRC M2 | Map competitors into direct, indirect, and alternative categories | P8 | competitors.category in {direct, indirect, alternative} |
| `COM-02` | SRC M2 | Extract competitor market positioning and core value propositions | P8 | Each competitor has positioning + value_props[] with sources |
| `COM-03` | SRC M2 | Evaluate publicly accessible competitor pricing tiers and packaging | P8 | competitor_pricing rows with source; 'not public' stated when absent |
| `COM-04` | SRC M2 | Summarize structural advantages and technical differentiators vs our product | P8 | Per-competitor comparison block with cited claims |
| `GRA-01` | SRC M3 | Native graph store with node labels Company, Person, Product, Competitor, Signal | P9 | Neo4j constraints exist for all five labels |
| `GRA-02` | SRC M3 | Directional edges works_at, competes_with, matched_to, contacted_via, replied_to, previously_employed_at | P9 | All six relationship types created by real app events |
| `GRA-03` | SRC M3 | Multi-hop query: warm introduction paths via shared employment or education | P9 | Graph view shows path from founder to target with hops |
| `GRA-04` | SRC M3 | Link newly discovered job requisitions to likely active buyer evaluation cycles | P10 | Job signal -> company flagged 'likely evaluating' with reasoning |
| `GRA-05` | SRC M3 | Map competitor displacement opportunities from renewal and churn signals | P10 | Displacement list shows company, competitor, signal, source |
| `GRA-06` | SRC M3 | Continuous signal ingestion that periodically updates node properties and relationship states | P10 | Cron endpoint + manual 'Refresh' update graph; last_refreshed shown |
| `ENR-01` | SRC M4 | Resolve primary business email with deliverability and MX validation status | P11 | enrichment_results has email, deliverability, mx_valid |
| `ENR-02` | SRC M4 | Resolve validated personal LinkedIn URL | P11 | linkedin_url stored with provider and validation status |
| `ENR-03` | SRC M4 | Resolve verified corporate domain record and headquarters location | P11 | domain + hq_location stored with provider |
| `ENR-04` | SRC M4 | Programmatic adapters for Hunter.io, Apollo, Clearbit, Prospeo (all four exist; unconfigured ones report not_configured, never fake data) | P11 | 4 adapters behind one interface; /api/health lists each status |
| `OUT-01` | SRC M4 | Multi-touch outreach sequences across Email and LinkedIn | P12 | Sequence has >=3 steps across both channels |
| `OUT-02` | SRC M4 | Weave specific company trigger signals and value props into copy | P12 | Message stores signal_ids used; UI highlights them |
| `OUT-03` | SRC M4 | Obey persona-specific tone, length, and CTA constraints | P12 | Validator rejects drafts violating persona rules |
| `OUT-04` | SRC M4 | HITL gateway: reps can approve, edit, or reject copy from a queue | P12 | Queue UI with 3 actions; edits saved with diff |
| `OUT-05` | SRC M4 | Nothing is queued or sent without approval (enforced in server code AND database) | P12 | Test: dispatch of unapproved message fails at API and DB level |
| `OUT-06` | SRC M4 | Messaging layer: delivery orchestration, webhook feedback loop, rate limiting (SendGrid / Gmail API / LinkedIn) | P13 | Dispatch works in dry_run and sandbox; webhook updates status |
| `PIP-01` | SRC M5 | Interactive Kanban: Matched -> Contacted -> Replied -> Meeting Booked -> Closed/Lost | P14 | Drag-and-drop between all stages; persisted |
| `PIP-02` | SRC M5 | Distinguish AI-driven progression from human rep actions | P14 | Every stage change has actor_type ai|rep|system shown on card |
| `PIP-03` | SRC M5 | Execution timestamps and audit records for every outbound interaction | P14 | Append-only audit_log; UI timeline per lead |
| `ANA-01` | SRC M5 | Conversion rate progression across stages, segmented by industry and company tier | P15 | Funnel with segment filters; counts shown with rates |
| `ANA-02` | SRC M5 | Split-test analytics across message templates, channels, and value propositions | P15 | Variant table: sent, replied, rate, sample size warning |
| `UI-01` | SRC UI | Hybrid layout: clean editorial look + dense spreadsheet-style operating surface | P1 | Data grids are dense (40px rows) inside the glass shell |
| `UI-02` | SRC UI | Inline enrichment indicators and confidence scores inside data grids | P11 | Grid cells show verified / unverified / not_configured + score |
| `UI-03` | SRC UI | Collapsible live 'Agent Thinking' panel: reasoning steps, graph traversals, validation logs | P3 | Panel streams real events from agent_events via Supabase Realtime |
| `UI-04` | SRC UI | Composite Fit & Intent score 0-100 with Tier A/B/C | P10 | Score + tier + breakdown tooltip on every lead |
| `BND-01` | SRC BONUS | Band: Planner/Architect agent role | P18 | Role exists, visible in agent stream |
| `BND-02` | SRC BONUS | Band: Builder agent role (API calls, graph mutations, outreach drafting) | P18 | Role exists and performs real tasks |
| `BND-03` | SRC BONUS | Band: Reviewer agent role (audits copy, fact checks, validates source links) | P18 | Role exists; rejects a bad draft in the demo |
| `BND-04` | SRC BONUS | Band: Security agent role (scans endpoints, data exposure, outbound links) | P18 | Role exists; flags a bad link in the demo |
| `BND-05` | SRC BONUS | Handoffs between agents using @mentions | P18 | Messages show @Builder, @Reviewer etc. and trigger the next agent |
| `BND-06` | SRC BONUS | Strict HITL control on schema modifications and outreach dispatch | P18 | Human approval card blocks both actions |
| `BND-07` | SRC BONUS | Operational telemetry: shared message stream, active session tokens, cumulative cost | P18 | Panel shows live messages, token count, USD cost |
| `FLX-01` | SRC BONUS | Flexprice metered events: Leads Enriched, Outreach Sent, Graph Queries Evaluated | P17 | 3 event types arrive in Flexprice (screenshot proof) |
| `FLX-02` | SRC BONUS | Credit enforcement: gate enrichment when org balance is exhausted | P17 | At 0 credits enrichment returns 402 and UI explains |
| `FLX-03` | SRC BONUS | Cost allocation: per-lead outreach cost mapped to the billing ledger | P17 | Lead detail shows cost; ledger row per lead |
| `SNK-01` | SRC BONUS | Snyk vulnerability scanning on outbound data-dispatching services | P19 | Scan results for messaging + enrichment modules |
| `SNK-02` | SRC BONUS | SAST and SCA embedded in CI/CD pipeline | P19 | CI job runs snyk code test + snyk test and fails on high |
| `ADD-01` | ADD | Login (Supabase Auth) + ready demo account so judges do not sign up | P2 | Judge can log in with demo creds in one click |
| `ADD-02` | ADD | Demo dataset mode, always labeled 'SAMPLE DATA' in UI; never mixed with real leads | P2 | Banner + badge; is_demo flag on rows |
| `ADD-03` | ADD | Dispatch modes dry_run (default) / sandbox / live, controlled by env + org setting | P13 | dry_run never calls a provider; live needs explicit flag |
| `ADD-04` | ADD | Compliance: unsubscribe link, suppression list, physical-address footer, no demo sends to real prospects | P13 | Suppressed address cannot be queued |
| `ADD-05` | ADD | Job runner with progress + resumable steps (serverless-safe) | P3 | Long discovery survives a 60s limit via chunked steps |
| `ADD-06` | ADD | Quota guard for free-tier APIs (Tavily, Hunter, Apollo, LLM) with caching | P3 | Repeat query served from cache; budget counters in settings |
| `ADD-07` | ADD | Structured logging + error boundary + /api/health | P0 | Health route lists env + integration status without leaking secrets |
| `ADD-08` | ADD | Citation evaluation harness proving no hallucinated leads | P20 | Script outputs citation precision; target 100% verified |
| `ADD-09` | ADD | Light/dark theme toggle (sun icon in sidebar must work) | P20 | Toggle works; no dead buttons anywhere |
| `ADD-10` | ADD | CSV export of leads and analytics | P15 | Download works and matches grid |
| `ADD-11` | ADD | End-to-end Playwright test of full pipeline (intake -> dashboard) | P20 | Test passes in CI |
| `DSN-01` | DSN | Page background gradient: slate-blue top fading to pale mint-white | P1 | Matches design.md tokens |
| `DSN-02` | DSN | Glass surface system (translucent fill, blur, white hairline border, soft shadow, large radius) | P1 | All cards/sidebar/chips use .glass classes |
| `DSN-03` | DSN | Typography: geometric light sans, thin large numerals, exact scale in design.md | P1 | Font loaded; scale tokens used |
| `DSN-04` | DSN | Sidebar: search + filter button, Home/Dashboard tiles, tree with active pill and count badges, bottom icon row, navy feedback pill, avatar + bell | P1 | Matches reference image 4/6 |
| `DSN-05` | DSN | Top nav (logo, app name, tabs) + page header (back circle, title, filter input, Refresh, download, share, navy context pill) | P1 | Matches reference image 6 |
| `DSN-06` | DSN | Pinned-view folder cards with peeking preview sheets, '+' tile, dashed drop zone | P1 | Matches reference image 3 |
| `DSN-07` | DSN | Hero metric card: thin large number + 2x2 sub-stat grid | P1 | Matches reference image 1 |
| `DSN-08` | DSN | Filter chip row with dropdown chevrons | P1 | Matches reference image 2/3 |
| `DSN-09` | DSN | Hatched band chart: dark and grey dotted series, diagonal hatch fill, dashed gridlines, glass callout | P1 | Matches reference image 2 |
| `DSN-10` | DSN | Gauge ring card with navy delta pill | P1 | Matches reference image 6 (bottom) |
| `DSN-11` | DSN | Thin vertical bar chart card with glass callout | P1 | Matches reference image 5/6 |
| `DSN-12` | DSN | Floating view-switcher dock (grid / board / table / chart) | P1 | Matches reference image 5/6 |
| `DSN-13` | DSN | Icon circle buttons, navy pill buttons, count badges | P1 | Matches references |
| `DSN-14` | DSN | Interaction states: hover, active, focus-visible, loading skeleton, empty, error | P1 | Defined for every component |
| `DSN-15` | DSN | Responsive: designed for >=1280px; collapses gracefully to 768px | P1 | Checked at 1440, 1280, 1024, 768 |
| `DEL-01` | DEL | Deployed end-to-end prototype (intake -> dashboard), public URL | P21 | Judge opens URL and runs the pipeline |
| `DEL-02` | DEL | Public GitHub repo with setup docs and modular architecture | P22 | Fresh clone runs from README alone |
| `DEL-03` | DEL | 2-3 minute demo video: intake, graph matching, message composition, HITL review, dashboard sync | P22 | Video exists, 2:00-3:00 long |
| `DEL-04` | DEL | System design document: query logic, graph ontology, reasoning loops, bonus services | P22 | docs/system-design.md complete |
| `DEL-05` | DEL | Startup viability brief: problem, buyer, differentiation vs existing tools, pricing, unit economics (15% of score) | P22 | docs/startup-viability.md reviewed |

## 10. Detailed behaviour

### 10.1 Product Intake (INT-*)
- Two entry points edit **one** `products` + `intake` record: chat and structured form.
- Chat agent keeps asking until it has: features, capabilities, constraints, reference customers (with outcomes), pricing model, contract values. It shows a progress checklist of what is still missing.
- Output (Zod-validated JSON) fields: `product_name, one_liner, features[], capabilities[], constraints[], reference_customers[{name, industry, size, outcome}], pricing{model, tiers[], acv_min, acv_max, currency}, geography_notes`.
- If the founder doesn't know something, field stays `null` and is flagged "missing". Never auto-filled with a guess.

### 10.2 ICP Engine (ICP-*)
- Input: completed intake. Output: `icp_versions` row (versioned; every edit = new version).
- Every field = `{ value, confidence (0.00–1.00), rationale }`. Confidence must come from a defined rubric (see `architecture.md` §9.2), not from the LLM "feeling".
- Industries carry real **NAICS** and **SIC** codes. A code lookup table is bundled in `src/lib/icp/naics.ts` (agent must source it from the official NAICS list; no invented codes).
- **Expansion**: ≥3 suggestions (if evidence exists), each: segment, why, evidence sources (real URLs), confidence. Actions: Accept / Adjust (edit fields) / Dismiss. Only Accepted/Adjusted feed discovery.

### 10.3 Discovery (DIS-*, COM-*)
- Pipeline: ICP → search queries → fetch pages → chunk + embed → LLM extracts candidates **with quotes** → server verifies each quote exists in the fetched text → save.
- **Citation = {url, quote, fetched_at, verified}**. `verified=true` only if the quote is found in stored page text (normalised whitespace, fuzzy ≥ 0.9). Unverified candidates are dropped and logged ("rejected: citation not found").
- Source types: company website, regulatory filing (e.g. SEC EDGAR / MCA filings where available), press release, job posting. Each saved source records its type.
- Negative signals (configurable list, defaults): hiring freeze / layoffs announcement, outside target regions, company too small/large vs ICP, already our customer, competitor-owned. Excluded leads stay in DB with `exclusion_reason` and show under an "Excluded" toggle.
- Competitors: `direct / indirect / alternative`; positioning, value props, public pricing tiers (or "pricing not public"), advantages/differentiators relative to our product — each claim cited.

### 10.4 Knowledge Graph (GRA-*)
- Store: **Neo4j (Aura Free)**. Reason in `architecture.md` §3 (Kùzu is no longer maintained).
- Postgres is the system of record; Neo4j is a projection built from it (idempotent `MERGE`). Graph rebuild from Postgres must always be possible.
- Graph view page: node/edge explorer (force layout), path finder (founder → target), saved queries panel.
- Warm intro: founder's own background (employers, schools) is entered in Settings → profile. Paths run through shared employers/schools with people at the target company. If no public data links them, UI says "no warm path found" — never invents one.

### 10.5 Enrichment (ENR-*)
- Waterfall: try providers in configured order, stop when the field is resolved with `status=verified`.
- Own MX check (Node `dns.resolveMx`) on every email domain regardless of vendor.
- Statuses: `verified`, `unverified`, `risky`, `not_found`, `not_configured`, `error`. Shown inline in grid (`UI-02`).
- Each paid call is metered (`FLX-01`) and credit-gated (`FLX-02`).

### 10.6 Outreach + HITL (OUT-*)
- Sequence default: Email #1 (day 0) → LinkedIn connect note (day 2) → Email #2 follow-up (day 5) → LinkedIn message (day 8) → Email #3 breakup (day 12). Configurable.
- Each message: uses ≥1 real signal from that lead; stores `signal_ids`; passes persona rules (max words, tone, one CTA); has a `variant_id` for A/B.
- Reviewer step checks: every factual claim appears in the lead's sources; no banned phrases; length; CTA count.
- **HITL queue**: shows draft, highlighted signals, source links, risk flags. Actions: Approve / Edit & Approve / Reject (reason required). Editing after approval **invalidates** approval (hash check).
- **LinkedIn**: there is no open API for sending LinkedIn messages. LinkedIn steps are **drafted + queued as manual tasks**: rep clicks "Open profile + copy message", then marks "Sent". Audit records `actor_type=rep`. UI states this plainly. Not a gap, a platform limit. Must be written in README.

### 10.7 Pipeline + Analytics (PIP-*, ANA-*)
- Stages: `matched → contacted → replied → meeting_booked → closed_won | closed_lost`. (`Closed/Lost` in spec = one column with won/lost outcome toggle; we show both as separate sub-lanes in the final column.)
- Every stage change writes `pipeline_events` + `audit_log` with `actor_type ∈ {ai, rep, system}`.
- Analytics: funnel counts + rates by stage; filters: industry, company tier (A/B/C), date range; A/B table across template, channel, value prop; shows sample size and a "too few samples" warning under 30 sends per variant (no fake significance).

### 10.8 Agent Thinking panel (UI-03)
- Right-side collapsible glass drawer. Shows live events: reasoning step, search query, page fetched, citation verified/rejected, graph traversal (with Cypher shown), enrichment call result, validation log. Events come from the `agent_events` table via Supabase Realtime (polling fallback). No fake streaming.

### 10.9 Fit & Intent score (UI-04)
Defined in `architecture.md` §9.3. Composite 0–100. Tier A ≥ 75, B 50–74, C < 50. Breakdown shown on hover.

## 11. Bonus features

### 11.1 Band multi-agent (BND-*)
- Four roles: Planner/Architect, Builder, Reviewer, Security. Handoffs via `@mentions`. HITL cards for schema changes and outreach dispatch. Telemetry: message stream, session tokens, cumulative cost.
- Band is a real product (band.ai, from Thenvoi AI) with Python/TypeScript SDKs. Its public documentation was thin when this pack was written. **Agent must read the current Band docs before coding and write down the real connection method in `progress.md`.** If Band cannot be connected, the in-app orchestrator still ships and the report must say `SPEC GAP: BND — REAL BAND CONNECTION NOT ACHIEVED` in capitals. See `architecture.md` §12.

### 11.2 Flexprice (FLX-*)
- Events: `leads_enriched`, `outreach_sent`, `graph_queries_evaluated`. Credit wallet per org. Enrichment gated at zero balance. Per-lead cost in ledger. See `architecture.md` §13.

### 11.3 Snyk (SNK-*)
- CI runs SAST (`snyk code test`) and SCA (`snyk test`), fails build on high severity. Outbound services (messaging, enrichment) scanned and reported in README.

## 12. Additions (ADD-*) — why each exists

| ID | Why it matters |
|---|---|
| ADD-01 | Without login anyone can burn our API quota; judges need a one-click demo account |
| ADD-02 | If an API fails during judging, the app still shows something — clearly labeled, never faked as real |
| ADD-03/04 | Live cold email from a hackathon project is a legal and reputation risk |
| ADD-05/06 | Free tiers are tiny; serverless has timeouts |
| ADD-08 | Proves "no hallucinated leads" with a number — judges will test this |
| ADD-09 | The reference design has a sun icon; a dead button looks broken |
| ADD-11 | One automated test of the whole pipeline protects the demo |

## 13. Out of scope (explicit)

- Team roles / permissions builder, SSO, billing UI beyond credits display
- Native mobile app
- Automated LinkedIn sending or scraping (platform terms; no API)
- Real payment processing (Flexprice used for metering + credit balance only)
- Multi-language UI

## 14. Success criteria

- A stranger opens the public URL, logs in with the demo account, and in under 5 minutes completes intake → approved message → Kanban update.
- Eval harness reports **100% of saved leads have a verified citation**.
- Demo video 2:00–3:00 shows intake, graph matching, message composition, HITL review, dashboard sync.
- Every row in `SPEC_CHECKLIST.md` is `DONE` with evidence, or explicitly marked `SPEC GAP` with reason.

## 15. Risks (honest)

| Risk | Impact | Mitigation |
|---|---|---|
| Scope is very large for the time available | Missed requirements | Phased plan with demo cut line at P14; no polishing before P14 |
| Free-tier API limits (Tavily, Hunter, Apollo, LLM) | Demo fails | Cache, quota guard (ADD-06), small runs, demo dataset fallback |
| LLM invents facts | Fails the core promise | Quote verification (DIS-03), Reviewer step, eval harness |
| Prompt injection from scraped pages | Agent misbehaves, leaks data | Treat page text as data only; see `security.md` |
| Weak coding model skips requirements | Silent gaps | Checklist + mandatory report format + `SPEC GAP` rule + DeepSeek review |
| Startup viability (15%) is crowded space (Apollo, Clay, Outreach, etc.) | Low score | `startup-viability.md`: narrow wedge, evidence-first + graph + HITL as differentiators |
| Cold email legality (CAN-SPAM, GDPR, India DPDP) | Real-world harm | dry_run default, unsubscribe, suppression, demo sends only to owned addresses. This is not legal advice |
