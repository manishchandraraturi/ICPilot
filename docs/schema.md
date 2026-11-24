# ICPilot — Data Model (`schema.md`)

> Postgres (Supabase) = system of record. Neo4j = projection. Migrations are append-only: never edit an applied file, add a new one.
> This file shows the intended DDL. The agent turns it into `db/migrations/000N_*.sql` + Drizzle schema (`src/lib/db/schema.ts`).

## 1. Conventions
- `id uuid primary key default gen_random_uuid()`; `created_at/updated_at timestamptz default now()`.
- Every business table has `org_id uuid not null references organizations(id)` and `is_demo boolean not null default false`.
- RLS ON for every table. Policy template in §5. Server jobs use the service role.
- Enums as `text` + `check` (easier to migrate).
- Money: credits in `numeric(12,2)`. Scores: `smallint` 0–100. Confidence: `numeric(3,2)` 0.00–1.00.

## 2. Postgres DDL

```sql
create extension if not exists vector;
create extension if not exists pgcrypto;

-- ===== Tenancy =====
create table organizations (id uuid primary key default gen_random_uuid(), name text not null,
  dispatch_mode text not null default 'dry_run' check (dispatch_mode in ('dry_run','sandbox','live')),
  created_at timestamptz default now());
create table org_members (org_id uuid references organizations(id) on delete cascade,
  user_id uuid not null, role text not null default 'member', primary key (org_id, user_id));
create table founder_profile (            -- for warm-intro paths
  id uuid primary key default gen_random_uuid(), org_id uuid not null references organizations(id),
  user_id uuid not null, full_name text, linkedin_url text,
  employers jsonb not null default '[]',   -- [{company, title, start_year, end_year}]
  schools jsonb not null default '[]',     -- [{school, degree, start_year, end_year}]
  sender_name text, sender_email text, physical_address text);

-- ===== Intake / Product =====
create table products (id uuid primary key default gen_random_uuid(), org_id uuid not null references organizations(id),
  name text not null, one_liner text, domain text, is_demo boolean not null default false, created_at timestamptz default now());
create table intake (id uuid primary key default gen_random_uuid(), org_id uuid not null references organizations(id),
  product_id uuid not null references products(id),
  data jsonb not null default '{}',        -- features[], capabilities[], constraints[], reference_customers[], pricing{}, geography_notes
  missing_fields text[] not null default '{}', completeness numeric(3,2) not null default 0,
  updated_at timestamptz default now());
create table intake_messages (id uuid primary key default gen_random_uuid(), org_id uuid not null, intake_id uuid not null references intake(id),
  role text not null check (role in ('user','assistant','system')), content text not null, created_at timestamptz default now());

-- ===== ICP =====
create table icp_versions (id uuid primary key default gen_random_uuid(), org_id uuid not null, product_id uuid not null references products(id),
  version int not null, status text not null default 'draft' check (status in ('draft','approved','archived')),
  fields jsonb not null,   -- { industries:[{value:{name,naics,sic},confidence,rationale,evidence[]}], headcount_tiers[], revenue_bands[], personas[], trigger_events[], geography[] }
  approved_by uuid, approved_at timestamptz, created_at timestamptz default now(), unique (product_id, version));
create table expansion_segments (id uuid primary key default gen_random_uuid(), org_id uuid not null, icp_version_id uuid not null references icp_versions(id),
  segment text not null, kind text not null check (kind in ('adjacent','emerging_vertical','non_obvious')),
  reasoning text not null, evidence_source_ids uuid[] not null default '{}', confidence numeric(3,2) not null,
  status text not null default 'suggested' check (status in ('suggested','accepted','adjusted','dismissed')),
  adjusted_value jsonb, decided_by uuid, decided_at timestamptz);

-- ===== Sources and RAG =====
create table sources (id uuid primary key default gen_random_uuid(), org_id uuid not null, url text not null, title text,
  source_type text not null check (source_type in ('website','filing','press','job_posting','other')),
  text text not null, content_hash text not null, fetched_at timestamptz not null default now(), is_demo boolean not null default false,
  unique (org_id, url, content_hash));
create table source_chunks (id uuid primary key default gen_random_uuid(), org_id uuid not null, source_id uuid not null references sources(id) on delete cascade,
  chunk_index int not null, text text not null, embedding vector(768));
create index on source_chunks using hnsw (embedding vector_cosine_ops);

-- ===== Companies, people, leads =====
create table companies (id uuid primary key default gen_random_uuid(), org_id uuid not null, name text not null, domain text,
  industry text, naics text, employee_count int, revenue_band text, hq_location text, hq_country text, description text,
  embedding vector(768), is_demo boolean not null default false, created_at timestamptz default now(), unique (org_id, domain));
create table people (id uuid primary key default gen_random_uuid(), org_id uuid not null, company_id uuid references companies(id),
  full_name text not null, title text, seniority text, linkedin_url text, is_demo boolean not null default false, created_at timestamptz default now());
create table person_history (id uuid primary key default gen_random_uuid(), org_id uuid not null, person_id uuid not null references people(id),
  kind text not null check (kind in ('employment','education')), org_name text not null, role text, start_year int, end_year int, source_id uuid references sources(id));
create table leads (id uuid primary key default gen_random_uuid(), org_id uuid not null, product_id uuid not null references products(id),
  icp_version_id uuid not null references icp_versions(id), company_id uuid not null references companies(id), primary_person_id uuid references people(id),
  stage text not null default 'matched' check (stage in ('matched','contacted','replied','meeting_booked','closed_won','closed_lost')),
  fit_score smallint, intent_score smallint, composite_score smallint, tier text check (tier in ('A','B','C')), score_breakdown jsonb,
  excluded boolean not null default false, exclusion_reason text, exclusion_source_id uuid references sources(id),
  likely_evaluating boolean not null default false, evaluating_reason text,
  owner_type text not null default 'ai' check (owner_type in ('ai','rep','system')), owner_user_id uuid,
  is_demo boolean not null default false, created_at timestamptz default now(), updated_at timestamptz default now(),
  unique (product_id, company_id));
create table lead_citations (id uuid primary key default gen_random_uuid(), org_id uuid not null, lead_id uuid not null references leads(id) on delete cascade,
  subject text not null check (subject in ('company','person','signal','fit')), source_id uuid not null references sources(id),
  url text not null, quote text not null check (char_length(quote) between 8 and 400),
  verified boolean not null, verification_method text, verified_at timestamptz default now());
-- RULE enforced in code AND by trigger: a lead cannot exist unless >=1 lead_citations(verified=true) exists (deferred constraint trigger).

-- ===== Signals =====
create table signals (id uuid primary key default gen_random_uuid(), org_id uuid not null, company_id uuid not null references companies(id),
  type text not null check (type in ('funding','job_posting','exec_hire','tech_change','churn_risk','renewal_window','competitor_negative','pricing_change','press_mention','hiring_freeze')),
  title text not null, detail text, strength numeric(3,2) default 0.5, source_id uuid not null references sources(id), quote text,
  detected_at timestamptz not null default now(), is_demo boolean not null default false);

-- ===== Competitors =====
create table competitors (id uuid primary key default gen_random_uuid(), org_id uuid not null, product_id uuid not null references products(id),
  name text not null, domain text, category text not null check (category in ('direct','indirect','alternative')),
  positioning text, value_props jsonb not null default '[]', advantages_vs_us jsonb default '[]', differentiators_ours jsonb default '[]',
  is_demo boolean not null default false, created_at timestamptz default now());
create table competitor_claims (id uuid primary key default gen_random_uuid(), org_id uuid not null, competitor_id uuid not null references competitors(id) on delete cascade,
  claim text not null, source_id uuid not null references sources(id), quote text not null, verified boolean not null);
create table competitor_pricing (id uuid primary key default gen_random_uuid(), org_id uuid not null, competitor_id uuid not null references competitors(id) on delete cascade,
  tier_name text, price_text text, packaging text, is_public boolean not null default true, source_id uuid references sources(id)); -- is_public=false => "pricing not public"
create table company_uses_competitor (org_id uuid not null, company_id uuid not null references companies(id), competitor_id uuid not null references competitors(id),
  source_id uuid not null references sources(id), quote text, primary key (company_id, competitor_id));

-- ===== Enrichment =====
create table enrichment_results (id uuid primary key default gen_random_uuid(), org_id uuid not null, lead_id uuid references leads(id), person_id uuid references people(id), company_id uuid references companies(id),
  provider text not null check (provider in ('hunter','apollo','clearbit','prospeo','own_dns')),
  field text not null check (field in ('email','linkedin_url','domain','hq_location')),
  value text, status text not null check (status in ('verified','unverified','risky','not_found','not_configured','error')),
  confidence numeric(3,2), mx_valid boolean, deliverability text, raw_ref text, created_at timestamptz default now());

-- ===== Outreach =====
create table templates (id uuid primary key default gen_random_uuid(), org_id uuid not null, name text not null, channel text not null check (channel in ('email','linkedin')),
  persona text, tone text, max_words int not null, cta text, body_skeleton text);
create table value_props (id uuid primary key default gen_random_uuid(), org_id uuid not null, product_id uuid not null references products(id), label text not null, text text not null);
create table sequences (id uuid primary key default gen_random_uuid(), org_id uuid not null, lead_id uuid not null references leads(id), person_id uuid references people(id),
  status text not null default 'draft' check (status in ('draft','active','paused','completed')), created_at timestamptz default now());
create table outreach_messages (id uuid primary key default gen_random_uuid(), org_id uuid not null, sequence_id uuid not null references sequences(id), lead_id uuid not null references leads(id),
  step int not null, channel text not null check (channel in ('email','linkedin')), send_after_days int not null default 0,
  template_id uuid references templates(id), value_prop_id uuid references value_props(id), variant_id text not null,
  subject text, body text not null, signal_ids uuid[] not null default '{}', risk_flags jsonb not null default '[]',
  status text not null default 'needs_review' check (status in ('draft','needs_review','approved','rejected','queued','sent','delivered','opened','clicked','replied','bounced','failed','suppressed','manual_pending','manual_sent')),
  body_hash text generated always as (md5(coalesce(subject,'') || E'\n' || body)) stored,
  approved_hash text, approved_by uuid, approved_at timestamptz, rejected_reason text,
  provider text, provider_message_id text, sent_at timestamptz, created_at timestamptz default now(), updated_at timestamptz default now(),
  -- HITL GATE (OUT-05): anything beyond review requires an approval of the CURRENT text
  constraint hitl_gate check (status in ('draft','needs_review','rejected') or (approved_by is not null and approved_hash = body_hash)));
create table approvals (id uuid primary key default gen_random_uuid(), org_id uuid not null, message_id uuid not null references outreach_messages(id),
  decision text not null check (decision in ('approve','edit_approve','reject')), editor_user_id uuid not null, before_body text, after_body text, reason text, created_at timestamptz default now());
create table email_events (id uuid primary key default gen_random_uuid(), org_id uuid not null, message_id uuid references outreach_messages(id),
  provider text not null, event_type text not null, raw jsonb, occurred_at timestamptz not null, received_at timestamptz default now());
create table suppression_list (org_id uuid not null, email text not null, reason text not null, created_at timestamptz default now(), primary key (org_id, email));

-- ===== Pipeline / audit =====
create table pipeline_events (id uuid primary key default gen_random_uuid(), org_id uuid not null, lead_id uuid not null references leads(id),
  from_stage text, to_stage text not null, actor_type text not null check (actor_type in ('ai','rep','system')), actor_id text, reason text, created_at timestamptz not null default now());
create table audit_log (id bigint generated always as identity primary key, org_id uuid not null, ts timestamptz not null default now(),
  actor_type text not null check (actor_type in ('ai','rep','system')), actor_id text, action text not null,
  entity text not null, entity_id uuid, lead_id uuid, details jsonb not null default '{}');
create or replace function audit_log_immutable() returns trigger language plpgsql as $$ begin raise exception 'audit_log is append-only'; end $$;
create trigger audit_log_no_update before update or delete on audit_log for each row execute function audit_log_immutable();

-- ===== Infra =====
create table jobs (id uuid primary key default gen_random_uuid(), org_id uuid not null, type text not null, status text not null default 'queued'
  check (status in ('queued','running','done','failed','cancelled')), params jsonb not null default '{}', cursor jsonb not null default '{}',
  progress int not null default 0, total int, error text, created_at timestamptz default now(), updated_at timestamptz default now());
create table agent_events (id bigint generated always as identity primary key, org_id uuid not null, run_id uuid, ts timestamptz not null default now(),
  type text not null, message text not null, data jsonb not null default '{}');
create index on agent_events (org_id, run_id, id);
create table provider_quota (org_id uuid not null, provider text not null, period text not null, used int not null default 0, limit_ int not null, primary key (org_id, provider, period));
create table api_cache (key text primary key, org_id uuid, value jsonb not null, expires_at timestamptz not null);

-- ===== Billing / metering (FLX) =====
create table usage_events (id uuid primary key default gen_random_uuid(), org_id uuid not null, event_name text not null check (event_name in ('leads_enriched','outreach_sent','graph_queries_evaluated','llm_tokens')),
  quantity numeric(12,3) not null, lead_id uuid, properties jsonb not null default '{}', occurred_at timestamptz not null default now(),
  synced boolean not null default false, sync_error text);
create table credit_wallet (org_id uuid primary key, balance numeric(12,2) not null default 0, updated_at timestamptz default now());
create table cost_ledger (id uuid primary key default gen_random_uuid(), org_id uuid not null, lead_id uuid, kind text not null, credits numeric(12,2) not null, usd_equiv numeric(12,4), created_at timestamptz default now());

-- ===== Agents (BND) =====
create table agent_runs (id uuid primary key default gen_random_uuid(), org_id uuid not null, goal text not null, status text not null default 'running', tokens_in bigint default 0, tokens_out bigint default 0, cost_usd numeric(12,4) default 0, band_session_id text, created_at timestamptz default now());
create table agent_messages (id uuid primary key default gen_random_uuid(), org_id uuid not null, run_id uuid not null references agent_runs(id), from_role text not null check (from_role in ('planner','builder','reviewer','security','human')),
  to_role text, text text not null, tokens_in int default 0, tokens_out int default 0, cost_usd numeric(12,6) default 0, ts timestamptz default now());
create table agent_approvals (id uuid primary key default gen_random_uuid(), org_id uuid not null, run_id uuid not null references agent_runs(id), kind text not null check (kind in ('schema_change','outreach_dispatch')),
  payload jsonb not null, status text not null default 'pending' check (status in ('pending','approved','rejected')), decided_by uuid, decided_at timestamptz);

-- ===== Saved views (DSN-06) =====
create table saved_views (id uuid primary key default gen_random_uuid(), org_id uuid not null, user_id uuid not null, name text not null, route text not null, filters jsonb not null default '{}', created_at timestamptz default now());
```

## 3. Indexes (minimum)
`leads(org_id, stage)`, `leads(org_id, tier, composite_score desc)`, `lead_citations(lead_id)`, `signals(company_id, detected_at desc)`, `outreach_messages(org_id, status)`, `pipeline_events(lead_id, created_at)`, `audit_log(org_id, ts desc)`, `usage_events(org_id, synced)`.

## 4. Integrity rules (must be enforced; each needs a test)
1. No lead without ≥1 `lead_citations.verified = true` (constraint trigger, deferred to end of transaction).
2. `outreach_messages.hitl_gate` check (above). Editing `subject/body` changes `body_hash`; the app must set status `needs_review` and clear `approved_*` in the same UPDATE.
3. `audit_log` immutable (trigger).
4. A suppressed email can never reach `queued` (trigger checks `suppression_list`).
5. `is_demo` rows never join real rows in analytics unless the "include sample data" toggle is on.

## 5. RLS template
```sql
alter table <t> enable row level security;
create policy "<t>_org_read"  on <t> for select using (org_id in (select org_id from org_members where user_id = auth.uid()));
create policy "<t>_org_write" on <t> for all    using (org_id in (select org_id from org_members where user_id = auth.uid()))
                                                 with check (org_id in (select org_id from org_members where user_id = auth.uid()));
```
`audit_log`: select only. `agent_events`: select for members, insert by service role.

## 6. Neo4j graph schema

### 6.1 Labels and properties
| Label | Spec? | Properties |
|---|---|---|
| `Company` | spec | `id, org_id, name, domain, industry, naics, employee_count, hq_country, tier, composite_score, is_demo` |
| `Person` | spec | `id, org_id, name, title, seniority, linkedin_url, is_founder, is_demo` |
| `Product` | spec | `id, org_id, name, is_ours` |
| `Competitor` | spec | `id, org_id, name, category, domain` |
| `Signal` | spec | `id, org_id, type, title, source_url, strength, detected_at (datetime)` |
| `School` | **ADD** | `id, org_id, name` — needed for education-based warm intros (spec: "shared employment or educational backgrounds") |

### 6.2 Relationships (spec names → Neo4j types; UI displays the spec's lowercase names)
| Spec name | Neo4j type | Direction | Properties |
|---|---|---|---|
| works_at | `WORKS_AT` | (Person)→(Company) | `title, since_year, source_url` |
| previously_employed_at | `PREVIOUSLY_EMPLOYED_AT` | (Person)→(Company) | `title, start_year, end_year, source_url` |
| competes_with | `COMPETES_WITH` | (Competitor)→(Product) | `category` |
| matched_to | `MATCHED_TO` | (Company)→(Product) | `fit, intent, tier, icp_version_id` |
| contacted_via | `CONTACTED_VIA` | (Product)→(Person) | `channel, message_id, sent_at, actor_type` |
| replied_to | `REPLIED_TO` | (Person)→(Product) | `message_id, replied_at, sentiment` |
| *(ADD)* | `HAS_SIGNAL` | (Company)→(Signal) | — |
| *(ADD)* | `STUDIED_AT` | (Person)→(School) | `degree, start_year, end_year, source_url` |
| *(ADD)* | `USES` | (Company)→(Competitor) | `source_url` |

`contacted_via` / `replied_to` are written by real outreach events (P12–P14), so all six spec edges exist from real app activity.

### 6.3 Constraints (`graph/constraints.cypher`)
```cypher
CREATE CONSTRAINT company_id    IF NOT EXISTS FOR (n:Company)    REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT person_id     IF NOT EXISTS FOR (n:Person)     REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT product_id    IF NOT EXISTS FOR (n:Product)    REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT competitor_id IF NOT EXISTS FOR (n:Competitor) REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT signal_id     IF NOT EXISTS FOR (n:Signal)     REQUIRE n.id IS UNIQUE;
CREATE CONSTRAINT school_id     IF NOT EXISTS FOR (n:School)     REQUIRE n.id IS UNIQUE;
CREATE INDEX company_org IF NOT EXISTS FOR (n:Company) ON (n.org_id);
CREATE INDEX signal_type IF NOT EXISTS FOR (n:Signal) ON (n.type, n.detected_at);
```

### 6.4 Sync (idempotent)
All writes use `MERGE (n:Label {id:$id}) SET n += $props`. Relationships: `MERGE (a)-[r:TYPE]->(b) SET r += $props`. `scripts/rebuild-graph.ts` wipes by `org_id` and re-projects from Postgres.

### 6.5 Named queries (client may only call these by name; params always bound, never string-concatenated)

**Q1 `warm_paths`** — shared employer/school between founder and a person at the target company (2 hops = real shared affiliation; time overlap shown when known)
```cypher
MATCH (f:Person {id:$founderId, org_id:$orgId})-[r1:PREVIOUSLY_EMPLOYED_AT|WORKS_AT|STUDIED_AT]->(shared)
      <-[r2:PREVIOUSLY_EMPLOYED_AT|WORKS_AT|STUDIED_AT]-(t:Person {org_id:$orgId})
MATCH (t)-[:WORKS_AT]->(c:Company {id:$companyId, org_id:$orgId})
WHERE (shared:Company OR shared:School) AND t.id <> f.id
RETURN t.id AS personId, t.name AS person, t.title AS title,
       labels(shared)[0] AS sharedType, shared.name AS sharedName,
       type(r1) AS founderRel, type(r2) AS targetRel,
       r1.start_year AS fStart, r1.end_year AS fEnd, r2.start_year AS tStart, r2.end_year AS tEnd
ORDER BY (shared:Company) DESC, person LIMIT 10
```
UI label: "Warm path (shared employer/school)". If zero rows → "No warm path found in public data."

**Q2 `job_req_evaluations`** — job postings suggesting an active evaluation (inference, shown as "likely", with evidence)
```cypher
MATCH (c:Company {org_id:$orgId})-[:MATCHED_TO]->(:Product {id:$productId})
MATCH (c)-[:HAS_SIGNAL]->(j:Signal {type:'job_posting'})
WHERE j.detected_at >= datetime() - duration({days:45})
  AND any(k IN $keywords WHERE toLower(j.title) CONTAINS toLower(k))
WITH c, collect(DISTINCT j) AS jobs
OPTIONAL MATCH (c)-[:HAS_SIGNAL]->(o:Signal)
  WHERE o.type IN ['funding','exec_hire','tech_change'] AND o.detected_at >= datetime() - duration({days:90})
WITH c, jobs, collect(DISTINCT o.type) AS corroborating
RETURN c.id AS companyId, c.name AS company, size(jobs) AS relevantJobs,
       [x IN jobs | {title:x.title, url:x.source_url}] AS evidence, corroborating,
       CASE WHEN size(jobs) >= 2 OR (size(jobs) >= 1 AND size(corroborating) >= 1)
            THEN 'likely_evaluating' ELSE 'possible' END AS stage
ORDER BY stage DESC, relevantJobs DESC
```

**Q3 `displacement`** — customers of a competitor with churn/renewal signals
```cypher
MATCH (c:Company {org_id:$orgId})-[u:USES]->(k:Competitor)-[:COMPETES_WITH]->(:Product {id:$productId})
MATCH (c)-[:HAS_SIGNAL]->(s:Signal)
WHERE s.type IN ['churn_risk','renewal_window','competitor_negative','pricing_change']
RETURN c.id AS companyId, c.name AS company, k.name AS competitor, u.source_url AS usesSource,
       collect({type:s.type, title:s.title, url:s.source_url, detectedAt:toString(s.detected_at)}) AS signals
ORDER BY size(signals) DESC
```

**Q4 `neighbourhood`** — nodes within 2 hops of a lead, for the graph view
```cypher
MATCH (c:Company {id:$companyId, org_id:$orgId})
CALL { WITH c MATCH p=(c)-[*1..2]-(x) WHERE x.org_id = $orgId RETURN p LIMIT 150 }
RETURN p
```

**Q5 `pipeline_edges`** — contacted/replied edges for one product (proves `contacted_via` and `replied_to` are live)
```cypher
MATCH (p:Product {id:$productId, org_id:$orgId})-[c:CONTACTED_VIA]->(x:Person)
OPTIONAL MATCH (x)-[r:REPLIED_TO]->(p)
RETURN x.id AS personId, x.name AS person, c.channel AS channel, c.sent_at AS sentAt, r.replied_at AS repliedAt
```

Each query execution: `agent_event(type='graph', data={name, cypher, params, rows})` + `meter('graph_queries_evaluated', 1)`.

## 7. Seed / demo data
- `db/seed/demo/*.json`, loaded by `scripts/seed-demo.ts` with `is_demo=true` everywhere.
- UI shows a `SAMPLE DATA` pill (navy context pill in page header, `design.md` §5.3) and a badge on every demo row. Analytics excludes demo rows unless toggled.
- Demo data must be **plausible but obviously fictional** companies (e.g. `Northwind Example Ltd`, domains ending `.example`) — never real people or real company names with invented contacts.
