# QCo Knowledge Systems — Council Options Memo
**From:** QCo Analyst  
**Based on:** Knowledge Systems Researcher pack (`/workspace/research/QCo-Knowledge-Systems-Research-Pack.md`)  
**Date:** 2026-09-05  
**Audience:** Director of AI Technology / QCo AI Council  

---

## BLUF

Do **not** pick “RAG vs Knowledge Graph vs Catalog” as a bake-off. Those are **layers**, not mutually exclusive products. QCo should adopt a **layered Source-of-Truth (SoT) model** for humans *and* agents:

1. **Permissioned retrieval fabric** (enterprise search / ACL-aware RAG) for narrative knowledge  
2. **Governed structured path** (SoR → warehouse/lakehouse → **semantic layer**) for numbers and entities  
3. **Catalog / context plane** for “what may agents use / what does it mean / who owns it”  
4. **Optional later:** curated enterprise KG for relationship-heavy domains; GraphRAG-class tools only for thematic multi-doc sensemaking  

**ZDR / no-training-on-company-data** are deployment overlays on whichever stack you buy — not a reason to prefer RAG over graphs.

**Nerd-trap guard:** “Which vector DB?” and “Should we GraphRAG everything?” are wrong-altitude. Right question: **How do we architect knowledge so agents and humans share the same authoritative layers under identity, freshness, and audit — without cognitive surrender to fluent but ungrounded answers?**

---

## Why this matters (QCo)

Agents (Astra / Claude / Grok Bot) will amplify whatever retrieval you give them. Without a SoT design:

- Wrong or stale chunks → confident wrong answers (cognitive surrender)  
- Service-account agents → ACL bypass  
- Free-form Text-to-SQL → metric hallucination  
- Agent memory treated as company truth → shadow CRM  

Council needs a **shared mental model** before tool selection or coaching can stick.

---

## How to think about SoT (operating model)

| Question type | Authoritative layer | Agent tool pattern |
| --- | --- | --- |
| “What does policy / runbook say?” | Docs via **ACL-aware** search/RAG + **citations** | `retrieve` under **user** identity |
| “What is the KPI / revenue / headcount?” | **Semantic layer** → warehouse (not raw SQL invention) | `query_metrics` / certified explores |
| “What is the live order / ticket status?” | **System of record** API under ACL | Scoped SoR read |
| “What may I use / is it certified / who owns it?” | **Data catalog / context plane** | Discover certified products only |
| “How are X and Y related officially?” | **Curated EKG** (if you invest) | Graph traverse with provenance |
| “What are the themes across this corpus?” | **GraphRAG / LazyGraphRAG-class** (optional) | Thematic retrieve — not transactional truth |
| “What did this user prefer last session?” | Agent memory | Preferences only — **never** org SoT |

**Rule:** Agents may treat a layer as authoritative only when (a) identity is user-propagated, (b) enforcement is at retrieve/query time (not post-filter), (c) provenance is returned (citation, metric ID, entity ID), (d) refusal is allowed when evidence is insufficient.

---

## Options comparison (decision-oriented)

### Option A — Permissioned work search first (turnkey or hyperscaler)
**What:** Glean-class or M365 Copilot / Vertex / Amazon Q-style fabric: connectors + inherited ACLs + assistant/agents on one index.  
**Strengths:** Fastest path to “humans + agents see what I can see”; reuses source ACLs; reduces shadow indexes.  
**Weaknesses:** Vendor concentration; ACL sync lag; oversharing becomes discoverable; weak alone for governed metrics.  
**Fit:** Default **narrative SoT** if QCo’s pain is “find stuff across SaaS.”  
**Hype:** High vendor marketing; validate per-connector permission freshness in pilot.

### Option B — Build-on hybrid search (Elastic / Azure AI Search / OpenSearch)
**What:** You own hybrid BM25+vector, DLS/ACL design, connectors.  
**Strengths:** Control, residency flexibility, deep tuning.  
**Weaknesses:** You own relevance + ACL ops; classical RAG demos without query-time ACL are unsafe.  
**Fit:** If IT already runs Elastic/Azure Search and will staff it.  
**Hype:** Medium — power with ops cost.

### Option C — Structured SoT + semantic layer (dbt MetricFlow / Cube / Looker) as agent KPI path
**What:** Warehouse truth + certified metrics exposed via MCP/`query_metrics`; agents forbidden from unconstrained Text-to-SQL for KPIs.  
**Strengths:** Highest authority for numbers; fights metric hallucination; pairs with GitHub/dbt workflows QCo already lives in.  
**Weaknesses:** Doesn’t solve wiki/policy Q&A alone.  
**Fit:** **Non-negotiable companion** to any RAG/search if agents will answer analytical questions.  
**Hype:** Low drama, high leverage.

### Option D — Catalog / context plane (Collibra, Alation AIOS, Atlan, Purview, Horizon, Unity, DataHub…)
**What:** Discovery, glossary, lineage, certification, agent MCP over **allowed** assets.  
**Strengths:** Stops agents from inventing schema; AI inventory/governance story for Council.  
**Weaknesses:** Catalog ≠ answer engine; empty/stale catalogs poison agents; 6–12 month programs common for heavy governance suites.  
**Fit:** **Router/context broker**, not the place truth is computed. Prefer platform-native (Unity/Horizon/Purview) if already in that cloud; buy Collibra/Alation/Atlan if cross-platform governance is the gap.  
**Hype:** “Context layer for AI” is hot — treat as navigation authority, not fact authority.

### Option E — Curated enterprise knowledge graph
**What:** Stewarded ontology + entities/edges bound to SoRs.  
**Strengths:** Official relationships, multi-hop blast radius, shared vocabulary for agents.  
**Weaknesses:** Ontology/stewardship burden; coverage gaps; not a doc store or metric compiler.  
**Fit:** **Phase 2+** for domains with clear relationship pain (Customer 360, compliance lineage, OT/asset graphs for power engineering) — not year-one default for all knowledge.  
**Hype:** High in analyst blogs; load is real.

### Option F — GraphRAG / LazyGraphRAG / Bedrock GraphRAG
**What:** LLM-built graph over documents for global/thematic and multi-hop narrative questions.  
**Strengths:** Themes across large corpora where top-k RAG fails.  
**Weaknesses:** Not enterprise SoT; extraction errors; ACL leakage via shared entities; full GraphRAG indexing cost; OSS GraphRAG in maintenance mode — prefer Lazy/managed paths.  
**Fit:** **Optional tool** for investigations/support themes — never replace semantic layer or EKG.  
**Hype:** Name recognition > default applicability. Vibecheck: “GraphRAG everything” is a nerd trap.

### Explicit non-options as company SoT
- **Agent memory** (Mem0/Zep/etc.): session continuity only.  
- **Unconstrained Text-to-SQL:** conditional at best.  
- **Classical RAG without query-time ACL:** privilege escalation surface.  
- **UI soft-hybrid** (Bot drives another AI app): rejected for production (prior BotOps analysis).

---

## Recommended stance for QCo (synthesis)

**Near-term target architecture (conceptual):**

```
Identity (IdP)
    → Permissioned retrieve (docs) ── citations ──┐
    → Semantic / SoR tools (numbers, entities) ──┼→ Agent / human answer
    → Catalog (certified assets only) ───────────┘
Optional later: EKG (relationships) | GraphRAG-class (themes)
```

**Phasing:**
1. **Now — decide principles** (this memo): layered SoT; user-propagated identity; citations; no post-filter ACL; agents refuse without evidence; ZDR/EFS as contract overlay.  
2. **90 days — two tracks:** (a) permissioned narrative retrieval pilot; (b) semantic-layer KPI path for agents (dbt/Cube/Looker — lean into existing GitHub/analytics habits).  
3. **Catalog:** adopt or extend whatever you already have; don’t start a greenfield Collibra-class program solely to “enable AI.”  
4. **EKG / GraphRAG:** park behind clear use cases and success metrics.

**Coaching link (Tri-System):** Grounded retrieve + citations = offloading. Fluent answer without provenance = surrender. Design UX and evals to force System 2 checks on consequential answers.

---

## What’s not being said

- Research pack doesn’t know QCo’s current SharePoint/Confluence/warehouse/IdP estate — vendor shortlist depends on that inventory.  
- “Enterprise search” won’t fix bad source ACLs; it will surface them.  
- No single vendor owns all layers well; hybrids are normal.  
- Freshness SLOs (content **and** ACL sync) are as important as model choice and usually under-specified in demos.

---

## Hype level (category)

| Category | Hype | Substance for QCo |
| --- | --- | --- |
| Turnkey permissioned search | High | High if connectors match your apps |
| Semantic layer for agents | Medium | Very high for KPI truth |
| Catalogs as “AI context” | High | Medium — navigation, not answers |
| Curated EKG | Medium-high | High only with steward capacity |
| GraphRAG | High | Niche thematic tool |

---

## Council decisions requested

1. **Adopt layered SoT framing** (yes/no) — not a single-technology RFP.  
2. **Authorize dual 90-day pilots:** (A) ACL-aware work search/RAG; (B) semantic-layer metric tools for agents — with shared eval harness (groundedness, ACL boundary tests, freshness SLO).  
3. **Mandate agent identity rule:** no privileged service identity for knowledge retrieve; OBO/user token required.  
4. **Defer EKG and GraphRAG** unless a named domain sponsor brings a use case.  
5. **Assign inventory owner** (IT + Director AI Tech): systems of record, doc stores, warehouse, existing catalog/search, IdP — due before vendor shortlist.

---

## Actionable next steps (Director of AI Technology)

1. 1-page estate inventory (apps + warehouse + IdP + any search/catalog today).  
2. Draft agent knowledge **routing table** (policy → retrieve; KPI → semantic; transaction → SoR).  
3. Pilot scorecard: permission boundary tests, citation rate, “I don’t know” rate, ACL revocation lag, human override rate.  
4. Hand CoS this memo for Council packet; Researcher pack remains the evidence appendix.

*Research evidence: `/workspace/research/QCo-Knowledge-Systems-Research-Pack.md`*
