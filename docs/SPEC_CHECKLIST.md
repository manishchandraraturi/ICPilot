# SPEC_CHECKLIST — traceability of every requirement

> Agents may edit ONLY the `Status` and `Evidence` columns. Status values: `TODO`, `IN PROGRESS`, `DONE`, `SPEC GAP`.
> `DONE` needs evidence: a file path + test name + commit hash. `SPEC GAP` needs a reason written in CAPITALS in the report.
> A row with no evidence is not DONE.

Total requirements: **89**

| ID | Requirement | Phase | Verify | Status | Evidence |
|---|---|---|---|---|---|
| `INT-01` | Interactive plain-language chat interface for product intake | P4 | Chat UI works; messages stored; agent asks follow-up questions | TODO | |
| `INT-02` | Structured form interface for product intake (same data as chat) | P4 | Form and chat edit the same intake record | TODO | |
| `INT-03` | Extract core product features, capabilities, technical constraints | P4 | Structured intake JSON has features[], capabilities[], constraints[] | TODO | |
| `INT-04` | Extract existing customer profiles and reference wins | P4 | Intake JSON has reference_customers[] with outcome text | TODO | |
| `INT-05` | Extract pricing frameworks, contract values, monetization models | P4 | Intake JSON has pricing{model, tiers[], acv_range} | TODO | |
| `ICP-01` | ICP: target industries and vertical classifications aligned to NAICS/SIC codes | P5 | ICP has industries[] each with naics_code and sic_code | TODO | |
| `ICP-02` | ICP: company size thresholds (headcount tiers and revenue bands) | P5 | ICP has headcount_tiers[] and revenue_bands[] | TODO | |
| `ICP-03` | ICP: buyer personas, functional titles, seniority levels | P5 | ICP has personas[] with titles[] and seniority | TODO | |
| `ICP-04` | ICP: high-value trigger events (tech stack change, exec hire, funding) | P5 | ICP has trigger_events[] with type and weight | TODO | |
| `ICP-05` | Field-level confidence scores 0.00 to 1.00 on every inferred ICP field | P5 | Every ICP field has confidence (2 decimals) and rationale; UI shows it | TODO | |
| `ICP-06` | ICP expansion: adjacent segments, emerging verticals, non-obvious uses, each with evidence-based reasoning | P5 | expansion_segments[] each has evidence_source_ids[] and reasoning | TODO | |
| `ICP-07` | Founder can explicitly accept, adjust, or dismiss each expansion suggestion | P5 | Three working actions; status persisted; accepted ones feed discovery | TODO | |
| `DIS-01` | Identify prospective corporate accounts based on the approved ICP | P7 | Discovery run uses the approved ICP version id | TODO | |
| `DIS-02` | Ingest public company websites, regulatory filings, press statements, job postings | P6 | Source types: website, filing, press, job_posting all supported and tagged | TODO | |
| `DIS-03` | Every prospect has mandatory inline source citations (quote + URL), verified against fetched page text | P7 | Lead without a verified citation cannot be saved; eval harness proves it | TODO | |
| `DIS-04` | Identify role-matched buyers (people) at each account | P7 | People rows with title matching ICP personas, each with a source | TODO | |
| `DIS-05` | Filter prospects against explicit negative signals (hiring freeze, non-target region) | P7 | Excluded leads kept with exclusion_reason; visible in UI toggle | TODO | |
| `COM-01` | Map competitors into direct, indirect, and alternative categories | P8 | competitors.category in {direct, indirect, alternative} | TODO | |
| `COM-02` | Extract competitor market positioning and core value propositions | P8 | Each competitor has positioning + value_props[] with sources | TODO | |
| `COM-03` | Evaluate publicly accessible competitor pricing tiers and packaging | P8 | competitor_pricing rows with source; 'not public' stated when absent | TODO | |
| `COM-04` | Summarize structural advantages and technical differentiators vs our product | P8 | Per-competitor comparison block with cited claims | TODO | |
| `GRA-01` | Native graph store with node labels Company, Person, Product, Competitor, Signal | P9 | Neo4j constraints exist for all five labels | TODO | |
| `GRA-02` | Directional edges works_at, competes_with, matched_to, contacted_via, replied_to, previously_employed_at | P9 | All six relationship types created by real app events | TODO | |
| `GRA-03` | Multi-hop query: warm introduction paths via shared employment or education | P9 | Graph view shows path from founder to target with hops | TODO | |
| `GRA-04` | Link newly discovered job requisitions to likely active buyer evaluation cycles | P10 | Job signal -> company flagged 'likely evaluating' with reasoning | TODO | |
| `GRA-05` | Map competitor displacement opportunities from renewal and churn signals | P10 | Displacement list shows company, competitor, signal, source | TODO | |
| `GRA-06` | Continuous signal ingestion that periodically updates node properties and relationship states | P10 | Cron endpoint + manual 'Refresh' update graph; last_refreshed shown | TODO | |
| `ENR-01` | Resolve primary business email with deliverability and MX validation status | P11 | enrichment_results has email, deliverability, mx_valid | TODO | |
| `ENR-02` | Resolve validated personal LinkedIn URL | P11 | linkedin_url stored with provider and validation status | TODO | |
| `ENR-03` | Resolve verified corporate domain record and headquarters location | P11 | domain + hq_location stored with provider | TODO | |
| `ENR-04` | Programmatic adapters for Hunter.io, Apollo, Clearbit, Prospeo (all four exist; unconfigured ones report not_configured, never fake data) | P11 | 4 adapters behind one interface; /api/health lists each status | TODO | |
| `OUT-01` | Multi-touch outreach sequences across Email and LinkedIn | P12 | Sequence has >=3 steps across both channels | TODO | |
| `OUT-02` | Weave specific company trigger signals and value props into copy | P12 | Message stores signal_ids used; UI highlights them | TODO | |
| `OUT-03` | Obey persona-specific tone, length, and CTA constraints | P12 | Validator rejects drafts violating persona rules | TODO | |
| `OUT-04` | HITL gateway: reps can approve, edit, or reject copy from a queue | P12 | Queue UI with 3 actions; edits saved with diff | TODO | |
| `OUT-05` | Nothing is queued or sent without approval (enforced in server code AND database) | P12 | Test: dispatch of unapproved message fails at API and DB level | TODO | |
| `OUT-06` | Messaging layer: delivery orchestration, webhook feedback loop, rate limiting (SendGrid / Gmail API / LinkedIn) | P13 | Dispatch works in dry_run and sandbox; webhook updates status | TODO | |
| `PIP-01` | Interactive Kanban: Matched -> Contacted -> Replied -> Meeting Booked -> Closed/Lost | P14 | Drag-and-drop between all stages; persisted | TODO | |
| `PIP-02` | Distinguish AI-driven progression from human rep actions | P14 | Every stage change has actor_type ai|rep|system shown on card | TODO | |
| `PIP-03` | Execution timestamps and audit records for every outbound interaction | P14 | Append-only audit_log; UI timeline per lead | TODO | |
| `ANA-01` | Conversion rate progression across stages, segmented by industry and company tier | P15 | Funnel with segment filters; counts shown with rates | TODO | |
| `ANA-02` | Split-test analytics across message templates, channels, and value propositions | P15 | Variant table: sent, replied, rate, sample size warning | TODO | |
| `UI-01` | Hybrid layout: clean editorial look + dense spreadsheet-style operating surface | P1 | Data grids are dense (40px rows) inside the glass shell | TODO | |
| `UI-02` | Inline enrichment indicators and confidence scores inside data grids | P11 | Grid cells show verified / unverified / not_configured + score | TODO | |
| `UI-03` | Collapsible live 'Agent Thinking' panel: reasoning steps, graph traversals, validation logs | P3 | Panel streams real events from agent_events via Supabase Realtime | TODO | |
| `UI-04` | Composite Fit & Intent score 0-100 with Tier A/B/C | P10 | Score + tier + breakdown tooltip on every lead | TODO | |
| `BND-01` | Band: Planner/Architect agent role | P18 | Role exists, visible in agent stream | TODO | |
| `BND-02` | Band: Builder agent role (API calls, graph mutations, outreach drafting) | P18 | Role exists and performs real tasks | TODO | |
| `BND-03` | Band: Reviewer agent role (audits copy, fact checks, validates source links) | P18 | Role exists; rejects a bad draft in the demo | TODO | |
| `BND-04` | Band: Security agent role (scans endpoints, data exposure, outbound links) | P18 | Role exists; flags a bad link in the demo | TODO | |
| `BND-05` | Handoffs between agents using @mentions | P18 | Messages show @Builder, @Reviewer etc. and trigger the next agent | TODO | |
| `BND-06` | Strict HITL control on schema modifications and outreach dispatch | P18 | Human approval card blocks both actions | TODO | |
| `BND-07` | Operational telemetry: shared message stream, active session tokens, cumulative cost | P18 | Panel shows live messages, token count, USD cost | TODO | |
| `FLX-01` | Flexprice metered events: Leads Enriched, Outreach Sent, Graph Queries Evaluated | P17 | 3 event types arrive in Flexprice (screenshot proof) | TODO | |
| `FLX-02` | Credit enforcement: gate enrichment when org balance is exhausted | P17 | At 0 credits enrichment returns 402 and UI explains | TODO | |
| `FLX-03` | Cost allocation: per-lead outreach cost mapped to the billing ledger | P17 | Lead detail shows cost; ledger row per lead | TODO | |
| `SNK-01` | Snyk vulnerability scanning on outbound data-dispatching services | P19 | Scan results for messaging + enrichment modules | TODO | |
| `SNK-02` | SAST and SCA embedded in CI/CD pipeline | P19 | CI job runs snyk code test + snyk test and fails on high | TODO | |
| `ADD-01` | Login (Supabase Auth) + ready demo account so judges do not sign up | P2 | Judge can log in with demo creds in one click | TODO | |
| `ADD-02` | Demo dataset mode, always labeled 'SAMPLE DATA' in UI; never mixed with real leads | P2 | Banner + badge; is_demo flag on rows | TODO | |
| `ADD-03` | Dispatch modes dry_run (default) / sandbox / live, controlled by env + org setting | P13 | dry_run never calls a provider; live needs explicit flag | TODO | |
| `ADD-04` | Compliance: unsubscribe link, suppression list, physical-address footer, no demo sends to real prospects | P13 | Suppressed address cannot be queued | TODO | |
| `ADD-05` | Job runner with progress + resumable steps (serverless-safe) | P3 | Long discovery survives a 60s limit via chunked steps | TODO | |
| `ADD-06` | Quota guard for free-tier APIs (Tavily, Hunter, Apollo, LLM) with caching | P3 | Repeat query served from cache; budget counters in settings | TODO | |
| `ADD-07` | Structured logging + error boundary + /api/health | P0 | Health route lists env + integration status without leaking secrets | TODO | |
| `ADD-08` | Citation evaluation harness proving no hallucinated leads | P20 | Script outputs citation precision; target 100% verified | TODO | |
| `ADD-09` | Light/dark theme toggle (sun icon in sidebar must work) | P20 | Toggle works; no dead buttons anywhere | TODO | |
| `ADD-10` | CSV export of leads and analytics | P15 | Download works and matches grid | TODO | |
| `ADD-11` | End-to-end Playwright test of full pipeline (intake -> dashboard) | P20 | Test passes in CI | TODO | |
| `DSN-01` | Page background gradient: slate-blue top fading to pale mint-white | P1 | Matches design.md tokens | TODO | |
| `DSN-02` | Glass surface system (translucent fill, blur, white hairline border, soft shadow, large radius) | P1 | All cards/sidebar/chips use .glass classes | TODO | |
| `DSN-03` | Typography: geometric light sans, thin large numerals, exact scale in design.md | P1 | Font loaded; scale tokens used | TODO | |
| `DSN-04` | Sidebar: search + filter button, Home/Dashboard tiles, tree with active pill and count badges, bottom icon row, navy feedback pill, avatar + bell | P1 | Matches reference image 4/6 | TODO | |
| `DSN-05` | Top nav (logo, app name, tabs) + page header (back circle, title, filter input, Refresh, download, share, navy context pill) | P1 | Matches reference image 6 | TODO | |
| `DSN-06` | Pinned-view folder cards with peeking preview sheets, '+' tile, dashed drop zone | P1 | Matches reference image 3 | TODO | |
| `DSN-07` | Hero metric card: thin large number + 2x2 sub-stat grid | P1 | Matches reference image 1 | TODO | |
| `DSN-08` | Filter chip row with dropdown chevrons | P1 | Matches reference image 2/3 | TODO | |
| `DSN-09` | Hatched band chart: dark and grey dotted series, diagonal hatch fill, dashed gridlines, glass callout | P1 | Matches reference image 2 | TODO | |
| `DSN-10` | Gauge ring card with navy delta pill | P1 | Matches reference image 6 (bottom) | TODO | |
| `DSN-11` | Thin vertical bar chart card with glass callout | P1 | Matches reference image 5/6 | TODO | |
| `DSN-12` | Floating view-switcher dock (grid / board / table / chart) | P1 | Matches reference image 5/6 | TODO | |
| `DSN-13` | Icon circle buttons, navy pill buttons, count badges | P1 | Matches references | TODO | |
| `DSN-14` | Interaction states: hover, active, focus-visible, loading skeleton, empty, error | P1 | Defined for every component | TODO | |
| `DSN-15` | Responsive: designed for >=1280px; collapses gracefully to 768px | P1 | Checked at 1440, 1280, 1024, 768 | TODO | |
| `DEL-01` | Deployed end-to-end prototype (intake -> dashboard), public URL | P21 | Judge opens URL and runs the pipeline | TODO | |
| `DEL-02` | Public GitHub repo with setup docs and modular architecture | P22 | Fresh clone runs from README alone | TODO | |
| `DEL-03` | 2-3 minute demo video: intake, graph matching, message composition, HITL review, dashboard sync | P22 | Video exists, 2:00-3:00 long | TODO | |
| `DEL-04` | System design document: query logic, graph ontology, reasoning loops, bonus services | P22 | docs/system-design.md complete | TODO | |
| `DEL-05` | Startup viability brief: problem, buyer, differentiation vs existing tools, pricing, unit economics (15% of score) | P22 | docs/startup-viability.md reviewed | TODO | |
