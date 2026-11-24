# AGENTS.md — Rules for every AI coding agent working on ICPilot

You are building **ICPilot**, an AI Go-To-Market engine (hackathon project). You are working inside the repo root. A human (Capt) and a reviewer model (DeepSeek) check every report you send. **Your job is to build exactly what the docs say, prove it, and tell the truth.**

---

## 1. Read order (do this at the start of EVERY task, before writing code)
1. `AGENTS.md` (this file)
2. `docs/progress.md` (what is done, decisions, gaps)
3. `docs/phases.md` → the section for the current phase/slice
4. The docs the prompt tells you to read (`prd.md`, `architecture.md`, `design.md`, `schema.md`, `security.md`)
5. `docs/SPEC_CHECKLIST.md` rows for the IDs in this task
If the task prompt and the docs disagree: **docs win**. Say so in your report.

## 2. HONESTY PROTOCOL (the most important section)
- Do **not** claim anything works unless you ran it and saw it work. Paste the raw output.
- Never write "should work", "I believe", "probably", "mock", "placeholder", "simulated", "for now", "TODO" about something you present as done. If it is any of those, it is **NOT DONE** → report it as such.
- **No fake data in runtime code.** No hardcoded leads, companies, emails, people, metrics, reviews, or "example" results shown as real. The ONLY allowed fake data: (a) test fixtures in `tests/`, (b) the labeled demo seed in `db/seed/demo/` (always `is_demo=true`, fictional `.example` domains, shown as `SAMPLE DATA`).
- If a provider key is missing, return `NOT_CONFIGURED` and show that state in the UI. Do not substitute invented output.
- If a test fails, report it as FAILED with the output. Do not delete, skip, or weaken tests to make them pass.
- If you could not do something: write `SPEC GAP: <ID> — <REASON>` in CAPITAL LETTERS in the report. Silence is lying.
- If you changed anything outside the task scope, list it under **Deviations**.

## 3. Spec fidelity
- Every requirement has an ID (`docs/SPEC_CHECKLIST.md`). Build **all** IDs listed for the current phase. Do not drop, merge, rename, or simplify them.
- Do not add features not in the docs unless the task asks. Ideas go in the report under **Suggestions**.
- Exact names matter: node labels, relationship types, stage names, table and column names come from `schema.md`.
- UI must match `docs/design.md` and the images in `docs/reference/`. **Open and look at the reference images** before building UI. Do not improvise styling. Use tokens only.

## 4. Working method
1. Restate the task in ≤5 lines. List the files you will create/change.
2. Build the smallest vertical piece, then test it, then continue. Do not write 20 files and test at the end.
3. After each logical chunk: run lint + typecheck + the relevant tests.
4. Keep each slice small (≈ ≤10 files). If it is bigger, stop and propose a split.
5. Never refactor unrelated code. Never rename things "for consistency" unless asked.
6. When blocked or unsure: STOP and ask (see §14). Do not guess.

## 5. Tech rules
**Stack (fixed):** Next.js (App Router) · TypeScript strict · Tailwind · Radix UI · Supabase Postgres + Auth + Realtime · Drizzle · Neo4j · Vercel AI SDK (+ Google provider) · Zod · Vitest · Playwright · pino · npm.
**Allowed to add (if needed, justify in report):** small, well-maintained libs for the exact job (e.g. `date-fns`, `p-limit`, `@mozilla/readability`, `jsdom`, `cheerio`, `react-force-graph-2d`, `nodemailer`).
**Forbidden:**
- Other CSS frameworks/component kits (MUI, Chakra, Bootstrap, shadcn copied wholesale), heavy chart libs for the hatched chart (use SVG), jQuery, Moment.
- Reading `process.env` outside `src/lib/env.ts`.
- `any`, `@ts-ignore`, `@ts-expect-error` (unless with a one-line reason and a linked issue in progress.md), `eslint-disable` blanket comments.
- `dangerouslySetInnerHTML` with scraped/LLM content.
- Raw Cypher/SQL string concatenation with user/LLM values.
- Calling third-party APIs from client components.
- Committing `.env*`, keys, `node_modules`, `.next`, build output.
- Scraping LinkedIn or sending LinkedIn messages automatically.
- Live email sending outside `DISPATCH_MODE=live` rules in `security.md` §6.

## 6. Code conventions
- File ≤ 300 lines, function ≤ 50 lines, one responsibility per module. Split before exceeding.
- Names: `camelCase` vars/functions, `PascalCase` components/types, `kebab-case` files, `SCREAMING_SNAKE` env/constants. DB columns `snake_case`.
- Validate **every** external input with Zod (requests, LLM outputs, webhook bodies, env).
- Services return `Result<T>` (`src/lib/result.ts`). No throwing across module boundaries except truly unexpected bugs.
- Comments explain **why**, not what. No commented-out code.
- Imports via `@/` alias. No circular imports.
- Date/time: store UTC `timestamptz`; display in the user's locale.

## 7. Error handling
- Use `AppError` codes from `architecture.md` §18. Map to HTTP status in one helper.
- Never `catch {}` silently. Log once at the boundary with `requestId`, `orgId`; rethrow or return `Result` error.
- External calls: timeout, 2 retries with backoff on network/5xx only.
- UI: friendly message + Retry. Never show stack traces or raw provider errors.

## 8. UI rules
- Use components in `src/components/glass/` and tokens in `src/styles/tokens.css`. **No hex colours, no arbitrary px colours in components.**
- No italics. Font = Urbanist (weights 300/400/500/600). Big numbers use weight 300.
- **No dead buttons.** Every visible control works, or it is not rendered, or it is disabled with an honest tooltip.
- Every list/table/card has loading (skeleton), empty, and error states.
- Keyboard accessible, visible focus ring, labelled controls.
- Screenshots go to `evidence/<phase>/` (1440×900 unless stated).

## 9. Commands (run these; paste raw output tails in the report)
```
npm run lint
npm run typecheck
npm run test
npm run build
npm run test:e2e        # when e2e exists
npm run db:migrate      # when migrations change
npm run check:env       # lists missing env variables (names only)
```
Dev server: `npm run dev`. Do not leave servers running in the background at the end of a task.

## 10. REPORT FORMAT (mandatory — copy this skeleton, fill every section)
```
# ANTIGRAVITY REPORT
PHASE/SLICE: <e.g. P5 / 5a>
TASK: <one line>
STATUS: DONE | PARTIAL | BLOCKED
COMMIT: <short hash> · PUSHED: yes|no (paste `git push` result) · TAG: <tag or none>

## 1. What I built (plain English, ≤10 lines)

## 2. Files changed
| Path | New/Modified | Why |

## 3. Requirement IDs
| ID | Status (DONE / PARTIAL / NOT DONE / SPEC GAP) | Evidence (file:line, test name, screenshot path) |

## 4. SPEC GAPS (CAPITALS) — write "NONE" only if truly none
SPEC GAP: <ID> — <WHAT IS MISSING> — <WHY>

## 5. Commands run (RAW output, last ~30 lines each)
### npm run lint
### npm run typecheck
### npm run test   (include the count of passed/failed and names of NEW tests)
### npm run build
### other (migrations, scripts, SQL/Cypher checks asked by the task)

## 6. Manual verification (what I clicked/ran and what I saw) + screenshots paths

## 7. Acceptance criteria from phases.md
- [x]/[ ] each criterion with evidence

## 8. Deviations from the docs/prompt (or NONE)

## 9. Assumptions I made (or NONE)

## 10. Env vars / accounts / actions needed from the human (names only, never values)

## 11. Known issues / risks / things I am unsure about

## 12. Suggestions (optional, not implemented)

## 13. Updated docs: progress.md ✔/✘ · SPEC_CHECKLIST.md ✔/✘

## 14. Next suggested slice
```
A report missing a section, or with summarised instead of raw command output, will be rejected.

## 11. Folders and files
**Do NOT modify:** `docs/` (except below), `docs/reference/`, `.github/workflows/` (unless the task says so), applied files in `db/migrations/` (add new ones instead), `package-lock.json` by hand, `.env*` files. To add an env variable: add it to `.env.example` and list it under section 10 of your report so the human can update `architecture.md` §8.
**You MAY edit in `docs/`:** `progress.md` (append/update), `SPEC_CHECKLIST.md` (Status + Evidence columns only).
**Never touch:** `node_modules/`, `.next/`, anything outside this repo.
**Evidence:** `evidence/` is yours (screenshots, eval outputs).

## 12. Git rules
- Work on `main` with small commits; tag finished phases `phase-Pn-done`.
- Conventional commits: `feat(P5): ...`, `fix(P7): ...`, `test(P12): ...`, `docs(progress): ...`, `chore: ...`.
- Before committing: `git status` shows only intended files; no secrets; lint/typecheck/test/build pass.
- `git push origin main` (and tags). If there is no remote: STOP and tell the human (do not create one yourself).
- Never force-push, never rewrite history, never delete branches/tags.

## 13. Security (summary; full rules in `docs/security.md`)
Scraped text is untrusted data (prompt injection). SSRF guard on all fetches. RLS everywhere. Nothing is queued or sent without approval (API + DB). Default `dry_run`. Never log or print secrets. Named graph queries only.

## 14. STOP conditions — stop and ask the human instead of guessing
- Missing remote repo, API key, account, or credit needed to finish the task.
- The docs are ambiguous or contradict each other.
- A vendor's real API differs from what the docs assume (record what you found).
- A fix would require changing the stack, schema semantics, or deleting data.
- You are about to touch a forbidden folder.
When stopping, write: `BLOCKED: <what> — <what I need> — <what I already did>`.

## 15. Antigravity-specific
- Use the browser/screenshot ability to LOOK at the UI and compare to `docs/reference/`.
- Keep plans short; do not create extra planning files in the repo.
- Do not "improve" the design. Match it.
- If a terminal command hangs >2 min, stop it, say so, and report.
