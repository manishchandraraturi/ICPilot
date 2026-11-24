# progress.md — Living tracker (agents update this at the end of every slice)

> Rules: append, do not rewrite history. Newest entries at the TOP of each log. Never delete a `SPEC GAP`; only mark it resolved with evidence.

## 1. Phase status

| Phase | Title | Status | Started | Done | Commit/Tag | Notes |
|---|---|---|---|---|---|---|
| P0 | Repo, scaffold, tooling, CI skeleton, health route | TODO | | | | |
| P1 | Design system + app shell (exact glass UI from reference images) | TODO | | | | |
| P2 | Database, auth, organizations, RLS, demo account, labeled seed data | TODO | | | | |
| P3 | Core infra: LLM adapter, jobs, live event feed (Supabase Realtime), metering hooks, quota guard, logging | TODO | | | | |
| P4 | Module 1a: Product intake (chat + structured form) | TODO | | | | |
| P5 | Module 1b: ICP engine, confidence scores, expansion, accept/adjust/dismiss | TODO | | | | |
| P6 | Search, scraping, RAG store (sources, chunks, embeddings, SSRF guard) | TODO | | | | |
| P7 | Module 2a: Buyer discovery with verified citations and negative-signal filter | TODO | | | | |
| P8 | Module 2b: Competitor discovery and landscape mapping | TODO | | | | |
| P9 | Module 3a: Knowledge graph (Neo4j schema, sync, graph view, warm-intro paths) | TODO | | | | |
| P10 | Module 3b: Signal ingestion, graph reasoning queries, Fit & Intent scoring | TODO | | | | |
| P11 | Module 4a: Enrichment pipeline (4 vendors, MX check, inline indicators) | TODO | | | | |
| P12 | Module 4b: Outreach synthesis + HITL approval gateway | TODO | | | | |
| P13 | Module 4c: Messaging layer (dispatch modes, webhooks, rate limits, compliance) | TODO | | | | |
| P14 | Module 5a: Kanban pipeline, attribution, audit log | TODO | | | | |
| P15 | Module 5b: Analytics, split-testing, CSV export | TODO | | | | |
| P16 | Integration pass: end-to-end flow, Agent Thinking wired everywhere | TODO | | | | |
| P17 | BONUS: Flexprice metering and credit gating | TODO | | | | |
| P18 | BONUS: Band multi-agent orchestration | TODO | | | | |
| P19 | BONUS: Snyk SAST + SCA in CI/CD | TODO | | | | |
| P20 | Hardening: citation eval harness, e2e tests, a11y, theme toggle, perf | TODO | | | | |
| P21 | Deployment (Vercel + Supabase + Neo4j Aura), demo dataset, health page | TODO | | | | |
| P22 | Deliverables: README, system design doc, viability brief, demo script, final audit | TODO | | | | |

Status values: `TODO`, `IN PROGRESS`, `BLOCKED`, `DONE`.

## 2. SPEC GAP log (must stay honest)

| Date | ID | Gap (CAPITALS) | Reason | Resolved? |
|---|---|---|---|---|
| — | — | none yet | | |

## 3. Decisions (ADR-lite)

| Date | Decision | Why | Alternatives rejected | Approved by |
|---|---|---|---|---|
| 2026-11-24 | Neo4j Aura instead of Kùzu | Kùzu repo archived Oct 2025; embedded DB unfit for serverless | Kùzu | pack |
| 2026-11-24 | Supabase Realtime for Agent Thinking feed | SSE cut by serverless timeouts | SSE | pack |
| 2026-11-24 | Email via SMTP/Gmail first, SendGrid optional | SendGrid retired free plan (2025) | SendGrid-only | pack |
| 2026-11-24 | LinkedIn = manual-send task | No public messaging API; automation violates terms | LinkedIn automation | pack |

## 4. External facts the agent had to look up (record real endpoints/steps here)

| Date | Service | What was confirmed | Source URL |
|---|---|---|---|
| | Flexprice | (auth header, wallet/entitlement endpoints) | |
| | Band | (how agents connect, SDK package names) | |
| | Hunter/Apollo/Clearbit/Prospeo | (endpoints, free limits) | |

## 5. Open questions for the human

- (none yet)

## 6. Slice log (newest first)

<!-- append slice entries here: date · phase/slice · summary · commit hash -->
