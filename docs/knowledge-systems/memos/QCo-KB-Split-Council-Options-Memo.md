# QCo KB Split — Council Options Memo (Product KG + Company Knowledge Base)

**From:** QCo Analyst  
**For:** Chris (Director of AI & Technology) / QCo AI Council  
**Date:** 2026-09-28 (America/New_York)  
**Status:** v1.2 — Chris rulings and MBOM discovery answers of 2026-09-28 folded in (see below). Council-ready options; not a build plan  
**Builds on:** `research/QCo-KB-Split-Stardog-ISA95-and-Company-KB-Research-Pack.md` (evidence) · `memos/QCo-FusionManage-ISA95-Stardog-Architecture-Brief.md` · `memos/QCo-KG-Sprint-1-Work-Plan.md` (v2.0) · `memos/QCo-KG-Council-POV.md`

Analyst recommendations below are interpretation of the research pack and Architect brief. The research pack itself is evidence-only and does not rank options.

---

## Chris rulings (2026-09-28) — folded into this version

1. **SoR split confirmed.** PLM (Fusion Manage) owns engineering definition; NetSuite owns commercial identity (`internalid`, price). **MBOM ownership is not settled** — it is an open discovery item in the field-authority table, not assigned to NetSuite by default.
2. **Crosswalk steward is a named role; the individual is TBD.** Drafting (field-authority table, crosswalk design, MCP stub) does not wait on a name. Analyst view: the first *confirmed* crosswalk records and any sync still need a named person with hours, because an unowned crosswalk becomes a third item master.
3. **Company KB conceptual design is green-lit.** It runs independently of the Product KG. Later it may query the Product KG through an AI query interface that is **read-only and cited**; it never copies product data.

### MBOM discovery answers (Chris, 2026-09-28)

| Question | Answer | What it means |
|---|---|---|
| Q1. FM mBOM Editor in use? | Fusion Manage is **green-lit**; initial product families targeted in **90 days**. mBOM Editor decision TBD. | FM is now the committed engineering SoR, not just a candidate. MBOM host can't be named until the mBOM Editor decision is made. |
| Q2. NetSuite Advanced BOM / dated revisions on? | **Unknown**; Chris will advise. | Blocks naming the host; does not block drafting. |
| Q3. Where do routing/operations live? | **NetSuite plus tribal knowledge**; paper travelers go with work orders. | Routing is only partly in a system. Any routing answer from the product lane is low confidence until captured and certified. |
| Q4. Who edits manufacturing structure after an ECO? | **Product Development.** | Under the decision rule, Product Development is the likely MBOM owner and leads discovery. The host system stays open until Q1 and Q2 close. |

**Paper travelers — lane boundary.** Routing steps and operations are product-lane objects (ISA-95 operations definitions under Material Definition). Traveler routing steps go in as **candidate product-lane records** until the MBOM/routing host is decided. The Company KB captures only the tribal know-how *around* a traveler (tips, workarounds, who to ask), and cites the product record rather than copying steps (C3 pointers-only). This is also a real data-quality risk: undocumented routing is where agents would be most confidently wrong.

**Timeline anchor.** The product-lane 90-day plan now tracks the Fusion Manage target for initial families. Crosswalk and field-authority work should land before FM go-live for those families, so the knowledge layer reads from FM from day one rather than being re-pointed later.

Remaining Council asks are store path, Strety write-off, and ban-list expansion. Discovery still needs Q1 (mBOM Editor) and Q2 (Advanced BOM) closed; neither blocks drafting (see Council decision asks).

---

## BLUF

Council should approve the following as the working baseline for the two-lane knowledge Rock:

1. **SoR split is confirmed (Chris, 2026-09-28); MBOM host is in discovery, with Product Development as the likely owner** (they edit manufacturing structure after an ECO). Host system stays open until the mBOM Editor decision and the NetSuite Advanced BOM check close. Fusion Manage *can* hold MBOM (mBOM Editor / 2023 template), so the field-authority table carries MBOM as an open row until four in-house facts are confirmed: whether QCo's FM tenant uses the mBOM Editor; whether NetSuite Advanced BOM with dated revisions is on; where routings/operations live today; and who edits manufacturing structure when an ECO lands. Routing lives in NetSuite plus tribal knowledge and paper travelers, so routing answers are low-confidence until captured. Until then, agents answer MBOM questions only with a cited "source not yet authoritative" caveat, never from whichever system happens to be indexed.
2. **Crosswalk steward is a named role (individual TBD).** Drafting proceeds now. Confirmed crosswalk records, sync, and linker build still wait for a named person with hours/week.
3. **Store path: Option B first** — thin crosswalk (Postgres or property-graph tables) plus governed read-only MCP — unless Council **names and funds an ontology owner** for Stardog. Stardog Cloud Free (≤1M edges) is too small for production; Enterprise price is unpublished; named-graph security is off by default; MCP authenticates with an app API key (friction vs on-behalf-of-user). Reversible: design Option B so a Stardog/ISA-95 projection is an upgrade, not a rewrite.
4. **Company KB conceptual design is green-lit (Chris, 2026-09-28)** and runs independently of the Product KG. Pattern: permissioned RAG over 1–2 certified tribal slices + thin org/role graph citing Strety seats/People by UUID; Strety remains SoR for Rocks/Issues/Scorecard. **Do not** enable Strety’s official MCP write path in any agent year one — use read OAuth / cite-only. Any later link to the Product KG is a read-only, cited AI query interface — no product data copied into the company store.
5. **Expand the company ingest ban list** to Strety `/reviews`, shoutouts, currency-format Scorecard metrics, and finance-worded seat responsibilities (e.g. P&L language in the OpenAPI examples). Ban HR/finance/transactional/product-copy at ingest; cite products by `internalid`, never copy the product graph.
6. **Operating model:** formal product KG needs engineering/ontology stewardship; company KB may be associate-tended for *content queues* only if SME verifiers and ACL/allowlist gates stay in the loop. Evidence shows associate-alone fails on verification drift, ACL mistakes, and HR/finance leakage.
7. **Reject explicitly:** one mega store; company lane as a second product graph; Stardog without a named ontology owner; GraphRAG (or uncertified SharePoint) as product/price SoT.
8. **90-day Rock framing:** decision-ready foundation and gated thin tracks — not a live companywide brain. Product build unlocks after a named steward, the field-authority table (MBOM resolved or explicitly fenced), and store path; company design is already green-lit and runs in parallel.

---

## Why this matters (QCo)

QCo already decided that product knowledge comes first and that a companywide “brain” day one is the wrong Rock. The room then split knowledge strategy into **two lanes with two stores and two systems of record**. That is the right shape for an SMB with thin IT: engineering definition lives in Fusion Manage (PLM); commercial identity and transactional truth stay in NetSuite; Rocks and scorecard live in Strety; tribal process notes become certified slices under human verification — not a third ERP.

What fails without Council clarity is quieter and more expensive: a knowledge graph that mints its own part numbers (third item master); agents that write Rocks or log Scorecard numbers through Strety MCP; MBOM answered from whichever system happens to be indexed; Stardog stood up with no one who owns SPARQL/SHACL; or a lone associate ingesting SharePoint and Strety until HR reviews and currency metrics leak into answers. The Rock is faster correct answers for Ops and commercial work under real user identity — not theater.

---

## Locked constraints

Treat the following as room-locked. This memo does not re-argue them.

| Lane | System of record | Store / pattern | Job |
| --- | --- | --- | --- |
| **Product** | PLM (Fusion Manage) = engineering definition (EBOM, revision, change, Material Definition). NetSuite = commercial identity (`internalid`, price). **MBOM: open discovery item.** KG is **read-only beside** the transaction path. | Stardog/ISA-95 **only if** ontology owner named; else **Option B** (thin crosswalk + governed MCP) first | Same part across certified engineering + commercial evidence |
| **Company** | Strety = SoR for Rocks / Issues / Scorecard. Humans + approved systems for tribal → certified slices | Separate system: permissioned RAG over certified tribal + thin org/role/Rock graph | Internal NLP Q&A; roles and accountability |

**Company ingest ban (locked):** no HR, no finance, no NetSuite transactional fields, no product-graph copy. Cite product by `internalid` when a Rock or SOP mentions a SKU.

**Sprint 1 v2.0:** product-lane build parked until crosswalk steward + SoR split + field-authority. Company track is green-lit (2026-09-28) and runs under Analyst/Architect gates; steward drafting proceeds with the individual TBD.

---

## Decision Facts that change the plan

These are Analyst interpretations of Decision Facts DF-1–4 and related evidence in the research pack. Confidence tags follow the pack.

### Strety (company SoR)

Seats (API object `Role`) and People are first-class with UUIDs. The 5–7 EOS “roles” *inside* a seat are structured text (`responsibilities[]`) **without IDs**. Rock, Issue, Metric, and To-Do ownership is keyed to a **Person UUID, not a seat**. Deriving seat accountability as Rock → Person → `seats[]` is **lossy** when one person holds multiple seats. External seat assignees are free text.

Strety exposes a real public REST API (OAuth 2.0, `read`/`write` scopes, **10 requests / 10 seconds per token**) and an **official MCP that is read/write by design**. No webhooks appear in the OpenAPI spec; change detection means polling `updated_after`. The API also exposes **performance reviews** (`/reviews`), shoutouts, metrics with `number_format` including `currency`, and seat examples that mention P&L — all HR/finance-adjacent for the ban list.

**Implication:** org graph may key seats and People by UUID. Do **not** key Rocks to seat-responsibility text. Cite Rocks/Issues/Scorecard; do not duplicate them. Keep agent path on **read OAuth**; leave Strety MCP write off for agents in year one.

### Fusion Manage (engineering SoR)

Fusion Manage covers Material Definition well (items, revisioning, EBOM, change orders, lifecycle, where-used; REST v3 with bulk reads and `If-Modified-Since`). It also **can hold MBOM** (Autodesk University class on mBOM Editor; 2023 tenant template stores EBOM and MBOM in one Items workspace — Medium confidence; whether QCo’s tenant has mBOM Editor is unverified). Fusion Manage uses date-based revision effectivity; NetSuite Advanced BOM uses its own non-overlapping Effective Start/End dates — **two effectivity schemes**. No first-party Autodesk FM–NetSuite connector was found (partner connectors only).

**Implication:** Per Chris's ruling, MBOM is a discovery row in U3, not a default. The four facts that decide it are internal, not vendor-doc questions: (a) FM tenant mBOM Editor in use, or EBOM only; (b) NetSuite Advanced BOM with dated revisions on, or flat BOMs; (c) where routings/operations live (NetSuite, spreadsheets, or future Tulip); (d) who edits manufacturing structure after an ECO. Whoever edits it today is the likely owner; the table should then map FM revision to NetSuite BOM revision. Leaving both live without a fence creates dual-SoR confusion the KG cannot resolve.

### Stardog (product store candidate)

Enterprise pricing is **unpublished** (sales call). Cloud Free is shared, ≤1M stored edges — too small for a production multi-source slice. Named-graph security is **off by default**; unauthorized graphs are **silently dropped** (empty results, not errors), which complicates `acl_deny` detection. Cloud MCP is NL→SPARQL (three tools), remote mode beta, authenticated with an **app API key** plus optional SSO token override — friction against G8 on-behalf-of-user. Voicebox API access is documented as currently limited / not enabled for most users. Required skills are RDF, SPARQL, OWL/RDFS, and SHACL.

**Implication:** software is not the gate; **named ontology ownership and skills** are. That matches Architect trigger #3 and Sprint gate U4.

### ISA-95 coverage gaps

ISA-95 / B2MML covers Material Definition, equipment, personnel as MOM resources, and operations definition/schedule/performance. It has **no native objects** for cutsheets, pricebooks, photometrics, engineering change orders, or CPQ configuration rules. There is no standards-body OWL rendering of full ISA-95.

**Implication:** any Stardog or Option B model needs a thin QCo extension vocabulary (plus schema.org-style family/variant patterns where useful). Do not pretend “ISA-95 ontology” already ships those marketing and commercial artifacts.

---

## Options (decision-oriented)

### Product store: Option B first (recommended default) vs Stardog now

| | **Option B first (Analyst recommendation)** | **Stardog now** |
| --- | --- | --- |
| **What** | Thin crosswalk tables (Postgres or managed property graph) + ISA-95-aligned names/annotations + same governed MCP tools (`resolve_item`, `get_product_context`, `change_impact`, `list_doc_slices`; refuse `uncertified` · `stale` · `acl_deny` · `unknown_item`) | Full RDF/OWL in Stardog Cloud Enterprise (or equivalent), SHACL, named-graph security deliberately enabled, Voicebox/MCP as expert-only — never the product-answer path |
| **When it fits** | Default for thin IT; SQL/Cypher skills available; no funded ontology owner | External interchange or formal reasoning need **and** named ontology owner with sustained hours |
| **Risks** | Less query-time subclass reasoning; validation is app-level | Unpublished cost; silent ACL drops if misconfigured; API-key MCP vs OBO; MSP unfamiliarity with SPARQL/SHACL (unverified); Cloud Free insufficient |
| **Reversible path** | Design model and MCP so Option A is a **projection**, not a rewrite (Architect ADR-6) | RDF data is portable; Stardog-specific Voicebox/VG features are not — isolate consumers behind MCP |

**Analyst recommendation:** Record **Option B first** unless Council fills the ontology-owner RACI cell with a person and funded capacity. ISA-95 alignment happens in the model now either way (B2MML-shaped names and annotations), not by buying a graph database.

### Company KB pattern (Analyst recommendation)

Use **permissioned RAG** over certified tribal slices (named verifier, review cadence, query-time ACL) plus a **thin org/role graph** that cites Strety Chart / seat (`Role`) / Person UUIDs. Rocks, Issues, and Scorecard stay cite-only to Strety as SoR — read OAuth, polling within rate limits. Do **not** enable Strety MCP write in the agent path year one.

**Link to the Product KG (per Chris's ruling):** later, the company assistant may call the product lane's governed query interface — read-only, under the user's identity, with citations back to PLM/NetSuite. Company store holds pointers (`internalid`, FM item id) at most, never copied specs, BOMs, or prices.

Modeling rules from DF-1: key seats and People by UUID; do **not** treat seat `responsibilities[]` text as durable IDs; represent Rock ownership as Person, and treat Rock→Person→`seats[]` as derived and lossy under multi-seat holders. Optional `accountableSeat` on a DocSlice is fine when humans assign it; do not invent it from free text.

### Explicitly reject

1. **One mega store** merging product engineering, NetSuite commercial, Strety EOS, and tribal SharePoint.  
2. **Company lane as a second product graph** (copying EBOM/price/SKU facts instead of citing `internalid`).  
3. **Stardog without a named ontology owner.**  
4. **GraphRAG or uncertified SharePoint as product/price/BOM system of truth.**

---

## Operating model

Chris’s design assumption — formal product KG vs informal company KB tended by a trained associate after standup — is **partially consistent** with industry practice that separates ontology engineering from content curation. It is **not** safe as “associate alone owns the company lane.”

Evidence (research pack OM-2): associate-tended queues fail on verification drift (“Does not expire” everywhere), ACL mistakes that search and agents amplify, and HR/finance leakage (Strety reviews/currency metrics; Purview DLP gaps including citation leakage and a Feb 2026 Copilot Chat bug). Informal *document* curation is not the same as informal *graph* curation once seats and Rocks become UUID-linked.

**Analyst staffing ask (RACI-style — names for Council to fill; no invented FTE counts):**

| Role | Authority | Notes |
| --- | --- | --- |
| Crosswalk steward | R on FM↔NetSuite identity | Named role, individual TBD (Chris ruling); drafting proceeds, confirmed records/sync wait for a name + hours |
| PLM / engineering SoR admin | R on FM extracts | Fusion Manage access |
| NetSuite commercial identity | R on `internalid`, price | Not engineering SoR; MBOM ownership pending discovery |
| MBOM discovery owner | R on closing Q1/Q2 and proposing host | **Product Development** (edits MBOM after ECO per Chris); host pending mBOM Editor + Advanced BOM answers |
| Ontology owner | R only if Stardog chosen | If blank → Option B |
| Product cert owners | R on engineering/commercial DocSlices | Family SMEs |
| Company tribal cert owners | R on 1–2 SOP/process sources | Named verifiers |
| Company content steward (associate OK) | R on verification queue / triage | **Not** sole ban-list or ACL owner |
| SME verifier(s) | A/C on accuracy of certified cards | Required; not optional |
| Strety / EOS admin | C | Read path; no agent write |
| Director (Chris) | A on unlock decisions | Rock owner |

Public Stardog/EKG FTE and dollar ranges in third-party blogs are large-enterprise composites and **unverified** for QCo’s 20–100 SKU slice; do not treat them as a QCo budget. What *is* actionable: do not green-light Stardog on software enthusiasm alone, and do not staff company ingest as a lone associate without SME verifiers and endpoint/field allowlists.

---

## Phased rollout (90 days)

Aligned to Sprint 1 v2.0 gates U1–U4, C1–C2, G8. Calendar lock with Chris; this is sequencing, not a Gantt.

| Phase | Product lane | Company lane (parallel if green-lit) | Gate |
| --- | --- | --- | --- |
| **Days 1–14 (Sprint 1)** | Timeline anchored to FM 90-day target for initial families; SoR split confirmed; steward role defined (individual TBD); field-authority table with MBOM as discovery row; MBOM four-fact check; family shortlist; store path recorded; MCP stub for two-lane cite rules | Conceptual design (green-lit); ban-list policy (incl. Strety reviews/shoutouts/currency); 1–2 tribal cert candidates; Strety cite sketch | U1–U4, C1–C2 draft, G8 contract language |
| **Days 15–45** | After unlock only: certified crosswalk candidates → human confirm; PLM engineering extracts for one family; no SuiteQL sole-spine build | First certified tribal slice; Strety read OAuth cite path; thin seat/Person index | U1–U3 hold; Park gate |
| **Days 45–90** | Governed MCP consumer under OBO for that family if precision/ACL hold; expand families only after quality | One conversational consumer under OBO ACL on certified company slices; still no Strety write from agents | G8 live; C1 enforced at ingest |

**Non-goals (90 days):** live mega-catalog; Stardog production load without ontology owner; agent write to any SoR; GraphRAG as product/price truth; HR/finance content; companywide brain; NetSuite Items token as a back door before U1/U2.

---

## Council decision asks

Copy-pasteable for CoS / Chris. Answer yes/no or name a person.

**Answered by Chris (2026-09-28):** SoR split confirmed with MBOM in discovery; crosswalk steward is a named role with individual TBD; company KB conceptual design green-lit (independent of Product KG, read-only cited query later, no product copies).

**Still open:**

1. **MBOM discovery:** Product Development leads (per Chris's Q4 answer). Still open: mBOM Editor decision (Q1) and NetSuite Advanced BOM status (Q2). Neither blocks drafting.  
2. **Crosswalk steward individual + hours/week (when ready):** ________________ / ____ hrs/week. Drafting does not wait; confirmed records and sync do.  
3. **Store path:** Option B first **or** Stardog with ontology owner = ________________ (name; if blank, Option B).  
4. **Strety MCP write off for agents year one?** Y / N *(Analyst recommends Y.)*  
5. **Ban list expands to Strety `/reviews`, shoutouts, and currency-format metrics (plus finance-worded responsibilities at field filter)?** Y / N *(Analyst recommends Y.)*

---

## Link to living work

- Sprint plan (do not fork): `/workspace/qco/docs/knowledge-systems/memos/QCo-KG-Sprint-1-Work-Plan.md` (v2.0).  
- Evidence pack: `/workspace/qco/docs/knowledge-systems/research/QCo-KB-Split-Stardog-ISA95-and-Company-KB-Research-Pack.md`.  
- Architect brief (Options A/B): `/workspace/qco/docs/knowledge-systems/memos/QCo-FusionManage-ISA95-Stardog-Architecture-Brief.md`.  
- Prior north star / tone: `/workspace/qco/docs/knowledge-systems/memos/QCo-KG-Council-POV.md`.

Analyst will not edit locked scorecards or other agents’ owned artifacts elsewhere. Next reversible work after Council answers: support Architect on field-authority table (U3, MBOM as discovery row) and company ban-list policy text for C1.

---

*End of Council options memo.*
