# demo-script.md — 2:30 demo video (must be 2:00–3:00)

Spec requires: **product intake, automated graph matching, message composition, HITL review, dashboard synchronization.**
Record at 1440×900, `DISPATCH_MODE=dry_run`, demo account, real (small) discovery run primed beforehand so waits are short.

| Time | Screen | Say | Must be visible |
|---|---|---|---|
| 0:00–0:12 | Dashboard | "Sales tools are scattered. ICPilot runs the whole loop with proof." | Glass dashboard |
| 0:12–0:35 | `/intake` | Type a plain-language product description | Chat + form fill, "missing" checklist |
| 0:35–0:55 | `/icp` | ICP with confidence scores; accept one expansion | Confidence 0.00–1.00, Accept/Adjust/Dismiss |
| 0:55–1:20 | `/leads` + Agent Thinking | Discovery: leads with citations; show one rejected citation | `cite_ok` / `cite_reject` events, citation chip with quote |
| 1:20–1:40 | `/graph` | Graph matching: warm path and job-req insight | Nodes/edges, path, Cypher in drawer |
| 1:40–1:55 | `/leads` | Tier A/B/C, score breakdown, enrichment status | Inline verified/not_configured indicators |
| 1:55–2:15 | `/outreach` | Composed sequence using real signals; **Reviewer flags** an issue; Edit and Approve | HITL queue, highlighted signals |
| 2:15–2:30 | `/pipeline` + `/analytics` | Card moves to Contacted (AI chip); funnel + A/B table update | Actor chip, audit timeline, variant table |
| bonus (if time) | `/agents`, `/usage` | Band stream with @mentions; Flexprice credits gate | Tokens, cost, 402 gate |

Checklist before recording: Neo4j awake · demo data labeled · no secrets on screen · notifications off · audio test.
