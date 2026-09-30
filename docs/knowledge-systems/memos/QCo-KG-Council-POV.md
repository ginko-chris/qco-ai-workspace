# QCo Council POV — Product Knowledge First, Then Build Outwards
**North Star:** We’re going to start with a **solid foundation of product knowledge** and build outwards. Products are the core of the business; everything else attaches to that knowledge.  
**From:** QCo Analyst  
**For:** Director of AI & Technology / QCo AI Council  
**Date:** 2026-09-18  
**Builds on:** Knowledge Systems Options Memo (layered SoT); Certified-Slices Checklist; KG Architecture POV  
**Status:** Council-ready options — not a build plan  

**Sources:**  
- `/workspace/qco/docs/knowledge-systems/memos/QCo-KG-Architecture-POV.md`  
- `/workspace/qco/docs/knowledge-systems/memos/QCo-Knowledge-Systems-Options-Memo.md`  
- `/workspace/qco/docs/knowledge-systems/checklists/QCo-Certified-Slices-Minimums-Checklist.md`  
- `/workspace/qco/docs/knowledge-systems/research/QCo-Semantic-KG-Manufacturing-Docs-Research-Pack.md` (evidence appendix)
- Claro product evidence graph guide (Jul 2026): start 20–100 products / one family; store optional — https://getclaro.ai/resources/guides/build-product-evidence-graph/

---

## BLUF

**Foundation first:** solid **product** knowledge (NetSuite Item spine + certified product evidence), then build outwards — drawings → SOPs/rules → BOM consumers. Not parallel “company brain” tracks.

Chris’s destination pattern is sound: **certified product docs → graph → API/MCP → Ask QCo / product lookup / BOM assist**.  
**Year-one scope of “companywide KG from every knowledge doc” is not.**

**Recommendation:** Approve a **thin certified product graph** — NetSuite Item identity (`internalid`) + certified DocSlices (start with **pricebook + cutsheets** for **~20–100 SKUs / one product family**) — exposed via **read-only MCP/API**, sitting **beside** permissioned narrative RAG and NetSuite as SoR.  
**Do not** replace RAG, NetSuite, or a metrics semantic layer with a mega-graph.

**EOS / Rock framing:** Rock-assist = *faster correct product answers for Ops/commercial work* — not “AI builds a company brain,” and not Corporate headcount theater.

**Ambition vs caution:** The thin first slice is a **quality gate**, not a ceiling. Ambition = **velocity of certified expansion** (families → doc types → consumers) once link precision + ACL hold — see Expansion roadmap below. Mega-graph day one is still rejected.

**Governance reality (landing zone, not freeze):** Departments are already running independent KB / POC work. Chris’s mandate is **“do it here, not there”** — shared NetSuite Item `internalid` + certified DocSlices + the same read-only MCP — with an on-ramp for teams mid-build. A build moratorium just pushes shadow graphs further underground.

---

## Why this matters (QCo)

| Layer (already approved direction) | Job | KG role |
| --- | --- | --- |
| Permissioned RAG | SOP/policy prose + citations | Stay here for narrative |
| NetSuite SoR | Live order / inventory / transactional price | Graph must not become shadow ERP |
| Semantic layer | KPIs | Out of scope for doc KG |
| **Thin KG** | Same part across cutsheet, drawing, pricebook, SOP | **Earning case** — cross-doc entity identity |

Without hard NetSuite identity, a “companywide” graph invents parts. With only RAG, agents miss multi-hop “which certified cutsheet governs this SKU?”

---

## Options (decision-oriented)

### Option A — Companywide KG first (not recommended)
Index all certified (or worse, uncertified) docs into one ontology/graph.  
**Pros:** Matches the big vision story.  
**Cons:** Ontology drift, thin-IT stewardship failure, ACL leaks via shared entities, stale edges, GraphRAG-as-fake-SoT. Violates certified-slices discipline at scale. Evidence: GraphRAG often underperforms vanilla RAG on fact lookup; production systems route only a small query share to graph; product evidence graphs can start as tables on 20–100 SKUs (research pack §1.2).  
**Hype:** High. **Fit:** Poor for SMB / 90-day realism.

### Option B — Catalog + RAG + semantic layer only (defer KG)
Ship Ask QCo on certified narrative slices + NetSuite reads; no graph.  
**Pros:** Fastest; matches dual-pilot path already on the table.  
**Cons:** Leaves cross-doc identity pain unsolved if that’s a real Ops/commercial Rock.  
**Fit:** Right if the pain is prose lookup only — wrong if SKU↔cutsheet↔pricebook linking is the Rock.

### Option C — Thin certified product graph + read-only MCP (**recommended**)
Architecture (from Architect POV):

```
NetSuite ──read-only SuiteQL/REST──► Item nodes (internalid)
Certified docs ──cert gate──► DocSlice nodes
        → candidate links → human confirm → typed edges
        → MCP/API (user ACL, citations, refuse-if-stale)
        → Ask QCo / product lookup (read only)
```

**Store v0:** property graph or Postgres + edge tables. **Defer** RDF/OWL. **Ban** GraphRAG as product/price/BOM SoT. Soft matches = candidates only — no embedding auto-merge into production edges. **No agent NetSuite write.**

**90-day Rock slice:**
1. Weeks 1–3: **Scoped** Item extract — **~20–100 SKUs / one product family** (not a full catalog dump even if SuiteQL can pull it); Items View–only token; sync SLO; refuse if stale  
2. Weeks 2–6: Certify pricebook + cutsheet DocSlices **for that family**; candidate linker + human confirm  
3. Weeks 6–10: Read-only MCP; one consumer under real user ACL  
4. Weeks 10–12: Measure link precision, refuse rate, time-to-find; decide next family / drawings/SOPs  

BOM assist waits until `part_of` is SoR- or certified-backed.


### Expansion roadmap (ambition = certified velocity)

*v0 stays Option C / certified gates. Scale is parallelizing proven slices — not widening before quality holds.*

| Horizon | Scope growth | Doc / edge growth | Consumers | Exit criteria to unlock next |
| --- | --- | --- | --- | --- |
| **D30** | One family live (~20–100 SKUs); SuiteQL sync + refuse-if-stale | Pricebook + cutsheets certified for that family; candidate→confirm loop working | Internal pilot (Ops/commercial power user) on MCP | Link precision ≥ target; ACL spot-checks pass; steward hours sustainable |
| **D90** | **+2–3 families** (same thin model) — family 2’s cert playbook must be **copy-paste proven** before family 3 starts; do **not** parallelize without a **second named steward** (one-person queue kills velocity) | Add **drawings** (title-block / rev links) for families that clear D30 bar | Second consumer (e.g. Ask QCo product lookup GA-ish for those families) | Replicate cert playbook without new ontology; refuse rate acceptable; no ACL pivot leaks; steward capacity ≥ families in flight |
| **D180** | Majority commercial catalog on **Item spine** (still family-batched, not dump-all) | **SOPs / rules** for high-pain workflows; unlock `part_of` / BOM assist **only** when **NetSuite BOM or a certified BOM** exists for that family — **never infer assemblies from drawings** | BOM / config assist prototype; optional thematic GraphRAG **off** the certified path | Named stewards per family; sync SLOs green; Council decides RDF/SHACL only if standards need appears |

**How we go fast without mega-graph day one:**
1. Freeze the thin shared model (Item / DocSlice / few edge types) — expansion is **more instances**, not new ontologies.  
2. Parallelize **certification playbooks** (owners + review cadence) once D30 metrics clear — Researcher’s scale pattern: quality gate, then concurrent families.  
3. MCP tool surface stays stable; new families appear as more resolvable Items, not new APIs.  
4. Do **not** unlock companywide uncertified SharePoint, embedding auto-merge, or agent NetSuite write as “acceleration.”  
5. **Ambition metric:** certified families/month + consumers on the **same MCP surface** — not raw node count.


### Landing zone for dept POCs already in flight

| Shadow pattern today | Prefer instead (“here”) | On-ramp (days–weeks, not a veto) |
| --- | --- | --- |
| Dept vector DB with local SKU strings as keys | Join on NetSuite **`internalid`**; treat display names as aliases only | Map existing keys → `internalid` once; keep local UI |
| Homegrown “product graph” / Notion-as-KB with invented part ids | Attach to **shared Item spine** + certified DocSlices | Publish freeze of edge types; import candidates into confirm queue |
| Bot/agent with standing ERP write or shared admin | Read-only MCP; human commit in NetSuite | Strip write tools; keep retrieve |
| Separate Ask-bot with no ACL / citations | Same MCP tools under user SSO + refuse-thin | Point consumer at shared endpoint |

**Publish early (Architect pattern) — keep it small:**
1. **Join key now:** NetSuite Item `internalid` is the only product spine. Dept POCs may keep their own indexes, but every product-facing answer must resolve to that ID (or refuse).
2. **MCP contract v0 (read-only):** `resolve_item` · `list_doc_slices` · `related` · refuse-if-stale/uncertified — same tools for every consumer; no per-dept tool forks.
3. **Migration, not ban:** Map shadow entities → `internalid`; promote only certified DocSlices into the shared slice; retire duplicate spines. Absorb evidence, not rival IDs.

Still not “acceleration”: a second SKU namespace, embedding auto-merge into production edges, or agent NetSuite write. Landing zone = attach to the spine.



---

## Reference architecture (stack detail)

*Adds concrete store / embed / host / SSO choices under Option C. Still not a vendor bake-off or build plan.*

### Logical planes (keep separate)

```
┌─ Identity ─────────────────────────────────────────────┐
│  IdP SSO (Entra ID / Okta) → MCP gateway + admin UI    │
│  Agent calls: OBO / user token (no see-all service ID) │
└────────────────────────────────────────────────────────┘
        │
        ▼
┌─ Relationship SoT (thin KG) ──┐   ┌─ Narrative SoT (certified RAG) ─┐
│ Graph store (LPG or tables)   │   │ Vector index + chunk store       │
│ Items + DocSlices + edges    │   │ Embeddings (Voyage-class)        │
│ Sync from NetSuite SuiteQL    │   │ Same DocSlice cert gate          │
└───────────────┬───────────────┘   └──────────────┬──────────────────┘
                │                                  │
                └──────────┬───────────────────────┘
                           ▼
              Read-only MCP / API (business verbs)
              resolve_item · list_doc_slices · related · refuse
                           │
              Ask QCo / product lookup (human still owns NetSuite write)
```

**Rule:** Graph answers *identity and links*; vector RAG answers *prose*; NetSuite answers *live transactional state*. Do not collapse these into one “company brain” database.

### Graph store (GraphDB options)

| Option | What it is | Fit for QCo v0 | Caveat |
| --- | --- | --- | --- |
| **A. Postgres + edge tables** (or `pgvector` sibling) | Relational “product evidence graph” | **Default for thin IT** — one DB, backups the MSP already understands | You write traversal SQL/API yourself |
| **B. Property graph (LPG)** — Neo4j Aura, Amazon Neptune PG, Azure Cosmos Gremlin | Native multi-hop + edge properties (`confidence`, `valid_to`, ACL tags) | Good if multi-hop product questions dominate and someone owns Cypher | Another managed service to pay for / monitor |
| **C. RDF / OWL “GraphDB”** (Ontotext GraphDB, Stardog, Neptune RDF) | IRI ontology + SPARQL + SHACL | **Defer** — only if standards interchange / SHACL validation becomes a named Rock | Ontology skill scarcity; slow incremental change for SMB |

**v0 recommendation:** Start **A** (Postgres + edges) *or* **B** if Chris already prefers a managed LPG and Ops can staff it. Treat vendor “GraphDB” marketing carefully — Ontotext **GraphDB** is RDF-class (option C), not the same as “any graph database.” Freeze the thin shared model either way; store is replaceable behind MCP.

### Vector store + embeddings (companion RAG lane)

Needed for certified cutsheet/SOP *narrative* retrieve — **not** a substitute for typed edges.

| Component | v0 recommendation | Notes |
| --- | --- | --- |
| **Vector index** | `pgvector` on the same Postgres *or* managed (Azure AI Search / OpenSearch / Pinecone) | Prefer one ops surface with Option A store; managed search if MSP already runs it |
| **Chunk store** | DocSlice text + page locus + cert metadata | Same certification gate as graph DocSlices |
| **Embeddings** | **Voyage AI** (`voyage-3` / voyage-context class) or peer (OpenAI `text-embedding-3-large`, Cohere embed-v3) | Choose one; store model id + version on every vector; re-embed on model change |
| **Hybrid retrieve** | BM25 + vector under **query-time ACL** | No post-filter-only; no GraphRAG community index on the certified product path |

Embeddings may **propose** candidate Item↔DocSlice links; they must **not** auto-write production `same_as` / `mentions` edges (Architect anti-pattern).

### Hosting options (SMB / dual control-plane)

| Hosting pattern | Pros | Cons | When |
| --- | --- | --- | --- |
| **1. Managed SaaS** (Neo4j Aura + Voyage API + Bot/MCP host) | Fastest; least MSP lift | Data residency / DPA; egress allowlists still required | Preferred if legal/DPA clears and slice stays small |
| **2. Customer VPC** (AWS/Azure: Neptune or Postgres RDS + private endpoints) | Stronger boundary vs SaaS; MSP-familiar | You own patching/cost; needs cloud owner | If QCo already has a cloud account and security prefers VPC |
| **3. On NetSuite host or corp MSP “extra VM”** | Feels convenient | **Avoid for SoT** — wrong RACI; couples graph outages to ERP/MSP tickets | Do not put certified KG here |
| **4. On-prem hypervisor** | Air-gap story | Thin IT ops tax; backup/DR weak | Defer unless compliance forces |

**Binding constraints:** Graph + vector sit in **Chris/app design** plane — not NetSuite host secrets, not MSP “we’ll click it in the UI.” NetSuite remains read-only SuiteQL/REST into the graph sync job. Eval / red-team agents stay on the **disposable default-deny VM** from the eval-hygiene memo — never against this SoT stack on open internet.

### SSO, identity, and ACL

| Control | Spec |
| --- | --- |
| **Human SSO** | Entra ID or Okta into MCP admin UI + any link-confirm UI (SAML/OIDC) |
| **Agent identity** | Propagate **end-user** (OBO / token exchange); MCP service principal for *connectivity only* — never for authorization |
| **Graph + vector ACL** | Query-time filter on DocSlice ACL + IdP groups; **per-hop** check on traversal (shared SKU pivots are a known leak mode) |
| **NetSuite token** | Separate **Items View–only** integration role; secrets owned by named Ops/NetSuite host contact — not embedded in Bot routines |
| **Audit** | Log principal, tool, item ids, hop depth, cert flags, refuse reason |
| **Break-glass** | Vaulted human admin for graph ops; not daily SSO; not reusable as agent identity |

### Suggested v0 bill of materials (illustrative)

1. **IdP:** existing Entra/Okta  
2. **Graph:** Postgres + edge tables **or** Neo4j Aura Free/Pro for the family slice  
3. **Vectors:** `pgvector` (same Postgres) **or** Azure AI Search if already licensed  
4. **Embeddings:** Voyage AI API (zero-retention / enterprise terms as required)  
5. **Sync:** scheduled SuiteQL pull → Item upsert (`internalid`); cert DocSlice ingest job  
6. **MCP:** small read-only tool surface behind SSO + OBO  
7. **Human confirm:** lightweight queue UI for candidate links  

Swap any single component later; freeze the **model** (Item / DocSlice / typed edges / refuse rules).


---

## Risks (Council should own)

| Risk | Mitigation |
| --- | --- |
| Ontology / scope creep | Freeze thin model; reject companywide program |
| Stale graph vs NetSuite | Sync SLO; refuse past review / miss |
| False SKU merges | Candidate queue; no auto `same_as` |
| ACL leaks via entity hub | Query-time user identity; check DocSlice ACL |
| Shadow ERP | NetSuite remains transactional truth |
| No steward → rot | Named weekly link confirmer (hours) or don’t start |

---

## Alignment with standing QCo rules

- **Certified slices only** on the agent path (owners, review dates, invalidation).  
- **Tier-by-job:** KG/MCP for consequential product identity; routine assist stays right-sized.  
- **Local weather:** This is seats/contracts/SoT we control — not lab KG hype.  
- **Messaging:** Better Ops/commercial answers — never “AI cut Corporate.”

---

## Decision asks (Council / Ops)

1. **Scope:** Thin product graph (Items + certified DocSlices) vs companywide KG program?  
2. **NetSuite:** Who provisions the **item-read-only** SuiteQL/REST token (Ops IT / MSP / NetSuite host)?  
3. **Certification owners:** Named owners for pricebook + cutsheet slices?  
4. **First consumer:** Product lookup vs pricebook assist vs defer BOM to Q2?  
5. **Stewardship:** Who confirms candidate links weekly — without this, do not start?  
6. **Stack defaults:** Postgres+edges vs managed LPG for v0? SaaS vs customer VPC? Confirm IdP (Entra/Okta) and whether Voyage (or peer) is acceptable under vendor DPA?  
7. **Landing zone:** Who inventories in-flight dept KB/POCs this month, and who owns publishing the MCP + `internalid` contract so those teams can attach?

---

## Recommendation

**Approve Option C** as a Rock-assist-sized initiative contingent on answers to asks 2, 3, and 5. Keep Option B (RAG + NetSuite) running in parallel for narrative SOPs. Reject Option A.

*Architecture detail:* `QCo-KG-Architecture-POV.md` · *Layered SoT:* `QCo-Knowledge-Systems-Options-Memo.md` · *Evidence:* `QCo-Semantic-KG-Manufacturing-Docs-Research-Pack.md`
