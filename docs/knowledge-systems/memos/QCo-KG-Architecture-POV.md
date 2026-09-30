# QCo Knowledge Graph — Architecture POV
**From:** Knowledge Graph Architect  
**For:** Chris (Director of AI & Technology) / QCo AI Council (via QCo Analyst)  
**Date:** 2026-09-18  
**Builds on:** Knowledge Systems Options Memo (layered SoT); Certified-Slices Minimums Checklist  
**Status:** Architecture stance — fold into Council POV; research pack may refine store/vendor notes

---

## BLUF

**Do not build a companywide knowledge graph first.** Chris’s vision (certified docs → graph → API/MCP → Ask QCo / product lookup / BOM assist) is right as a *destination pattern*, wrong as a *year-one scope*.

**Recommended:** a **thin certified product graph** (NetSuite Item identity + certified DocSlices from cutsheets / pricebook / drawings / SOPs / rules) exposed via **read-only MCP/API**, sitting beside — not replacing — permissioned narrative RAG and the NetSuite SoR. Full RDF/OWL enterprise ontology and “GraphRAG everything” are anti-patterns for thin IT.

---

## When a KG earns its keep (vs catalog + RAG + semantic layer)

| Need | Prefer | Why |
| --- | --- | --- |
| Policy / SOP Q&A with citations | Certified-slice **RAG** | Narrative, not multi-hop identity |
| Live order / inventory / price truth | **NetSuite** (SoR) under ACL | Graph copies go stale |
| KPIs / metrics | **Semantic layer** → warehouse | Not a doc graph |
| “Same part across cutsheet, drawing, pricebook, SOP” | **Thin KG** (entity–doc links) | Cross-doc identity is the earning case |
| BOM / config option rules | **Thin KG** (+ SoR BOM if present) | Multi-hop + constraints |
| Themes across a huge corpus | Optional **GraphRAG-class** later | Thematic only — never SoT |

**Rule of thumb:** If the pain is *finding prose*, stay on RAG. If the pain is *official relationships and identity across certified artifacts*, invest in a curated graph. QCo’s cutsheet/drawing/pricebook/SOP stack is the latter — but only for entities that NetSuite (or a certified BOM) already grounds.

---

## Recommended architecture (QCo-shaped)

```
NetSuite (SoR) ──read-only SuiteQL/REST──► Item nodes (internalid = hard key)
Certified docs ──cert gate──► DocSlice nodes (provenance + owner + review date)
                         │
                         ▼
              Candidate linker (SKU/name/drawing # as evidence)
                         │
                         ▼ human / deterministic confirm
              Edges: mentions | depicts | specifies | same_as | part_of | applies_to
                         │
                         ▼
         MCP / API (user identity, query-time ACL, citations, refuse-if-stale)
                         │
         Ask QCo / product lookup / BOM assist (read path only)
```

### Thin shared model (v0) — one schema, four corpora

**Nodes:** `Item` (NetSuite `internalid`), `Assembly` (only if SoR/certified BOM defines it), `Equipment` (SOP targets; soft until asset register), `DocSlice` (certified excerpt only).

**Edges (few, typed):** `mentions` / `depicts` / `specifies` (DocSlice → entity); `same_as` (confirmed only — never embedding-only auto-merge); `part_of`; `applies_to`.

**Non-negotiables**
- Agent path sees **only certified** DocSlices (checklist gate).
- Soft matches create **candidates**, not edges.
- **No standing NetSuite write** for agents; link queue is human-approved.
- SKU/display name are **evidence toward** `internalid`, never the primary key.

### Store choice

**Property graph or Postgres + edge tables** for v0. Defer RDF/OWL until you have real rule/reasoner needs and stewards. GraphRAG-style LLM-extracted mega-graphs are fine for investigations later — they are **not** the product/BOM SoT.

### API / MCP exposure

Expose a small tool surface, not the raw store:
- `resolve_item(sku|name)` → Item + confidence + NetSuite id  
- `list_doc_slices(item_id)` → certified slices + citations  
- `related(item_id, edge_types)` → one-hop / bounded multi-hop  
- `refuse` when sync stale, slice past review date, or ACL denies  

Propagate **end-user identity** at query time; no “see-all” service account for retrieve. Return provenance on every claim.

---

## 90-day realistic slice (Rock-assist sized)

Not “companywide KG.” One Rock-shaped outcome:

1. **Weeks 1–3:** NetSuite item extract — SuiteQL/REST, token role **Items View only**; upsert `Item` on `internalid`; sync health SLO; agents refuse product answers if extract stale.  
2. **Weeks 2–6:** Certify a **minimal** DocSlice set for **pricebook + cutsheets** (highest commercial identity pain); candidate linker + human confirm UI for `mentions`/`specifies`.  
3. **Weeks 6–10:** Read-only MCP tools above; one consumer (“Ask QCo” product lookup or pricebook assist) under real user ACL.  
4. **Weeks 10–12:** Measure: link precision, refuse rate, time-to-find vs baseline; decide whether drawings/SOPs join the **same** thin model next quarter.

BOM creation assist waits until `part_of` is SoR-backed or certified — don’t invent assemblies in the graph.

---

## Risks (and mitigations)

| Risk | Mitigation |
| --- | --- |
| **Companywide ontology drift** | Thin shared model; freeze edge types; no per-corpus ontologies |
| **Stale edges** | Sync SLO; never delete Items on miss; tombstone DocSlices; refuse past review |
| **False merges** | No auto `same_as` from embeddings; candidate queue |
| **ACL leaks** | Query-time pre-filter on user identity; don’t share entity hubs that bypass doc ACL without checking slice ACL |
| **Graph as shadow ERP** | NetSuite remains transactional truth; graph is identity + certified evidence |
| **GraphRAG-as-SoT** | Ban for product/price/BOM answers; optional later for themes only |

---

## Anti-patterns

1. Companywide KG before NetSuite hard identity exists  
2. RDF/OWL stack for SMB manufacturing day one  
3. Embedding-only entity resolution into production edges  
4. Agent write-back into NetSuite  
5. Indexing uncertified SharePoint dumps into the graph  
6. One mega-graph for SOPs + KPIs + orders + chatter  

---

## Decision asks (Council / Ops)

1. **Scope:** Approve thin product graph (Items + certified DocSlices) vs companywide KG program?  
2. **NetSuite:** Who provisions the **item-read-only** SuiteQL/REST token role (Ops IT / MSP / NetSuite admin)?  
3. **Certification owners:** Named owners for pricebook + cutsheet slices entering the agent path?  
4. **First consumer:** Product lookup vs pricebook assist vs defer BOM assist to quarter 2?  
5. **Stewardship:** Who confirms candidate links weekly (hours/week) — without this, the graph rots?

---

## Handoff

- @Knowledge Systems Researcher: landscape pack should stress manufacturing doc linking, property vs RDF, query-time ACL — not vendor bake-offs.  
- @QCo Analyst: fold this into Council POV; frame 90-day slice as Rock-assist (faster correct product answers), not “AI builds a company brain.” Align messaging with layered SoT memo: KG is **optional relationship layer**, not replacement for RAG or NetSuite.

*Companion paths:*  
`/workspace/qco/docs/knowledge-systems/memos/QCo-Knowledge-Systems-Options-Memo.md`  
`/workspace/qco/docs/knowledge-systems/checklists/QCo-Certified-Slices-Minimums-Checklist.md`

---

## Research pack reconcile (2026-09-18)

Source: `/workspace/qco/docs/knowledge-systems/research/QCo-Semantic-KG-Manufacturing-Docs-Research-Pack.md`

**No architecture change required** — evidence reinforces the Architect stance. Tightening notes for Analyst:

1. **Query mix:** Industry mixes often route only a small share (~7%) of queries to graph; do not size QCo for firmwide GraphRAG. Product lookup / time-sensitive facts stay on RAG + NetSuite.
2. **90-day narrowing:** Prefer **~20–100 SKUs / one product family** inside the pricebook+cutsheet cert set — not “all items” even if SuiteQL can dump the catalog.
3. **Store:** Default **LPG or certified tables**; RDF/OWL only if SHACL/standards interchange becomes a named requirement.
4. **ACL:** Enforce **per-hop / DocSlice ACL** under OBO identity; treat vector→entity pivots and community summaries as leak surfaces — keep GraphRAG out of the certified product path.
5. **Field authority:** ERP ≠ cutsheet for specs; graph edges carry provenance to the winning DocSlice, not a blended “truth blob.”
6. **Extractions:** LLM/output = **candidates** until human certification (already in v0 model).

Anti-patterns unchanged: companywide KG first; RDF day one; embedding auto-`same_as`; GraphRAG-as-SoT; agent NetSuite write.
