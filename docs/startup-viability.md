# startup-viability.md — Why ICPilot is a believable business (15% of the score)

> DRAFT for Capt to sharpen. Statements marked **[ASSUMPTION]** are not proven. Verify competitor features and prices before presenting; I did not verify them.

## 1. The pain (one sentence)
Early-stage B2B teams burn money and time because ICP, prospecting, enrichment, outreach and tracking live in different tools, so targeting is vague, messages are generic, and nobody can prove why a lead was chosen.

## 2. Who pays
- **Primary:** seed to Series A B2B startups with 1–5 people doing sales (founder-led sales). **[ASSUMPTION]** they currently stitch together 3–5 tools.
- **Secondary:** small agencies doing outbound for clients.
- Buyer = founder or Head of Sales. User = founder / SDR.

## 2.1 Why now
LLMs can read messy public pages and write copy, but the market is flooded with tools that produce **unverified leads and generic AI emails**. The open gap is *trust*: proof for every lead, and a human who approves every message.

## 3. Differentiation (what we claim, and the proof inside the product)
| Claim | Proof in the product |
|---|---|
| **Every lead has evidence** | Verified quote + URL per lead; eval harness reports 100% verified (`ADD-08`) |
| **Explainable targeting** | Confidence on every ICP field; Fit & Intent breakdown per lead |
| **Relationships, not lists** | Knowledge graph: warm-intro paths, job-req → evaluation, competitor displacement |
| **Human-in-the-loop by design** | Approval enforced in API **and** database; audit log of AI vs rep actions |
| **One loop, no silos** | Intake → ICP → leads → outreach → pipeline → analytics learn from each other |
| **Usage-aligned pricing** | Metered credits (Flexprice) map cost to each lead |

## 4. Competitive landscape (categories, verify specifics)
- **Contact databases / sales engagement:** Apollo, ZoomInfo, Outreach, Salesloft. Large, mature, broad. We don't beat them on database size.
- **Workflow/enrichment builders:** Clay and similar. Powerful, but the user designs the workflow.
- **Cold-email senders:** Instantly, Smartlead and similar. Strong deliverability tooling, weak on targeting proof.
- **AI SDR products:** autonomous agents sending emails. Risky on accuracy and compliance.
**Our wedge:** *evidence-first, human-approved* prospecting for founder-led sales; we do not compete on database size.

## 5. Business model **[ASSUMPTION — to validate]**
- Free: demo, small credit grant.
- Pay-as-you-go credits (enrich, send, graph queries) + monthly seat plan for teams.
- Illustrative unit economics to fill with real numbers: `cost per qualified lead = search + scrape + LLM + enrichment credits`. Measure it from `cost_ledger` in the app and show the real figure in the demo.

## 6. Go-to-market (first 100 users) **[ASSUMPTION]**
Founder communities, build-in-public posts showing real before/after funnels, templates for common ICPs, partnerships with startup accelerators.

## 7. Risks and honest answers
| Risk | Answer |
|---|---|
| "Apollo/Clay already do this" | We target trust and workflow; incumbents are database/workflow tools. We should say what we do **not** do |
| Data quality depends on public web + vendor APIs | Citation verification drops unprovable leads; coverage is lower but precision is high |
| Cold-email legality/deliverability | Dry-run default, suppression, unsubscribe, human approval; legal review needed before live use |
| LinkedIn restrictions | No automation; drafts + manual send |
| Vendor free-tier limits | Cache, quota guard, adapters swappable |

## 8. Metrics we will show in the demo
Citation precision (100% target) · time from intake to first approved message · reply rate by variant (with sample sizes) · cost per lead.

## 9. To complete before the pitch (Capt)
- Replace each **[ASSUMPTION]** with a real data point or remove it.
- Verify competitor claims and current prices.
- Add one real founder quote or interview if available.
