# setup-checklist.md — Accounts and keys YOU (Capt) must create

> Never paste real keys into chat or into a prompt. Put them in `.env.local` (and later in Vercel/GitHub secrets).
> Free-tier limits change often. Check each provider's current pricing page before relying on a number. I could not verify all limits.

| # | Service | Needed by | Used for | Notes |
|---|---|---|---|---|
| 1 | **GitHub** repo (public, empty) | **P0** | Code, CI, deliverable | Give DeepSeek/Antigravity the URL. Antigravity cannot create it for you. |
| 2 | **Node 20+**, Git, Docker Desktop | P0 / P2 | Local dev + test Postgres | Docker is only for the local test DB |
| 3 | **Google AI (Gemini) API key** | P3 | LLM + embeddings | Free tier is rate-limited; keep prompts small |
| 4 | **Supabase** project | **P2** | Postgres, Auth, Realtime, pgvector | Free tier. Copy URL, anon key, service-role key, DB URL |
| 5 | **Tavily** API key | P6 | Web search | Free monthly credits; cache everything |
| 6 | **Serper** key (optional fallback) | P6 | Search fallback | |
| 7 | **Neo4j AuraDB Free** | **P9** | Knowledge graph | Free instances can pause when idle; wake before demo |
| 8 | **Hunter** key | P11 | Email finding/verification | Small free quota |
| 9 | **Apollo** key | P11 | Contact/company enrichment | API access may need a paid plan; if unavailable the adapter reports `not_configured` |
| 10 | **Prospeo** key | P11 | Email/LinkedIn enrichment | Optional |
| 11 | **Clearbit** | P11 | Company enrichment | Clearbit's free tools were shut down in 2025; now HubSpot-based/paid. Probably `not_configured`. That's an honest, acceptable state |
| 12 | **SMTP (Gmail app password)** or Gmail API OAuth | P13 | Email sending (sandbox/own inbox) | Use your own addresses for tests. Don't blast real prospects |
| 13 | **SendGrid** (optional) | P13 | Alt email provider + webhooks | Free plan was retired in 2025; skip unless you have a paid/trial account |
| 14 | **Flexprice** account | P17 | Metering + credits | Needed for the +20 bonus |
| 15 | **Band** account + API access | P18 | Multi-agent bonus | Docs were thin when this was written; the agent must read current docs |
| 16 | **Snyk** account + `SNYK_TOKEN` | P19 | SAST/SCA in CI | Add token as GitHub Actions secret |
| 17 | **Vercel** account | P21 | Hosting | Connect GitHub repo |
| 18 | A **sender postal address** + sender name | P13 | Email footer (compliance) | Required in the footer |

**Before P0:** items 1 and 2.
**Before P2:** 4 (and 3 by P3).
**Reference images:** copy the 6 UI images into `docs/reference/` with the names in `design.md` (they are already in this pack).
