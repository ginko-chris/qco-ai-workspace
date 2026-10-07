# Semantic Knowledge Graphs from Manufacturing / SMB Documents — Research Pack

**From:** Knowledge Systems Researcher  
**To:** QCo Analyst / parent agent (merge-ready; **research only — no QCo product recommendations**)  
**Date:** 2026-09-18 (America/New_York)  
**Audience context:** Director of AI & Technology (Chris) / QCo AI Council  

**Premise under test (descriptive):** A companywide semantic knowledge graph (KG) built from certified-accurate cutsheets, product drawings, SOPs, pricebook, rules, etc., exposed via API/MCP for Ask-style agents, product lookup, BOM traversal, and related workloads.

**Prior art this pack builds on (do not redo):**
- Layered SoT already framed: permissioned narrative + semantic/SoR + catalog; curated EKG and GraphRAG deferred pending sponsor — [`QCo-Knowledge-Systems-Options-Memo.md`](../../memos/QCo-Knowledge-Systems-Options-Memo.md)
- Agent path limited to **certified slices** — [`QCo-Certified-Slices-Minimums-Checklist.md`](../../checklists/QCo-Certified-Slices-Minimums-Checklist.md)
- Landscape RAG/ACL/catalog/EKG/GraphRAG — [`QCo-Knowledge-Systems-Research-Pack.md`](./QCo-Knowledge-Systems-Research-Pack.md)

**Constraints addressed throughout:** Thin IT; NetSuite (or equivalent ERP) as structured SoR; **no standing agent write** to production SoR; **query-time ACL + user identity**; only certified slices enter the agent path.

**How to use:** Evidence and patterns for humans *and* AI agents. This pack does **not** rank options for QCo, prescribe a build, or say what QCo “should” do. Analysts synthesize.

---

## Table of contents

1. [Executive framing: when a semantic KG earns keep](#1-executive-framing-when-a-semantic-kg-earns-keep)
2. [Ontology / taxonomy design patterns (manufacturing)](#2-ontology--taxonomy-design-patterns-manufacturing)
3. [Construction pipeline](#3-construction-pipeline)
4. [Store models: RDF/OWL vs property graph vs hybrid](#4-store-models-rdfowl-vs-property-graph-vs-hybrid)
5. [GraphRAG / hybrid Graph+vector vs curated EKG](#5-graphrag--hybrid-graphvector-vs-curated-ekg)
6. [ACL / identity at query time on graph traversal](#6-acl--identity-at-query-time-on-graph-traversal)
7. [API / MCP exposure patterns for agents](#7-api--mcp-exposure-patterns-for-agents)
8. [Freshness, stale edges, ontology drift, invalidation](#8-freshness-stale-edges-ontology-drift-invalidation)
9. [Realistic 90-day slice patterns (industry-observed)](#9-realistic-90-day-slice-patterns-industry-observed)
10. [Notable stacks / vendors / patterns (with caveats)](#10-notable-stacks--vendors--patterns-with-caveats)
11. [Sources](#11-sources)

---

## 1. Executive framing: when a semantic KG earns keep

### 1.1 The decision is not “KG vs RAG”

Industry practice and prior QCo research treat retrieval, semantic/metrics layers, catalogs, curated graphs, and GraphRAG-class indexes as **layers**, not mutually exclusive products ([prior Options Memo](../../memos/QCo-Knowledge-Systems-Options-Memo.md); [Hogan et al., ACM Computing Surveys, 2021](https://dl.acm.org/doi/10.1145/3447772); [Semantic Layer vs Knowledge Graph framing, 2026](https://colrows.com/blogs/semantic-layer-vs-knowledge-graph/); [Forrester on semantics/ontologies for agentic AI](https://www.forrester.com/blogs/build-meaning-before-machines-why-semantics-ontologies-and-knowledge-graphs-matter-for-agentic-ai/)).

For manufacturing / SMB document workloads, a useful contrast is:

| Pattern | What it optimizes | Typical authority |
| --- | --- | --- |
| **Catalog + ACL-aware RAG + semantic/SoR tools** | Narrative findability; KPI/entity reads under identity; certified asset discovery | High for docs (with citations) and numbers (via semantic/SoR); weak for official multi-hop *relationships* unless joins are already modeled |
| **Thin curated graph (certified slice)** | Typed product↔BOM↔pricebook↔doc links that SoR/RAG cannot express cleanly | High **inside the certified slice**; coverage deliberately incomplete |
| **Companywide curated EKG** | Shared vocabulary + relationship SoT across domains | High where stewarded; expensive; coverage gaps become “confident holes” |
| **LLM-extracted GraphRAG index** | Themes / multi-doc sensemaking over corpora | Chunk-grounded retrieval aid — **not** enterprise SoT ([GraphRAG paper](https://arxiv.org/abs/2404.16130); [prior pack §B](./QCo-Knowledge-Systems-Research-Pack.md)) |

### 1.2 Evidence that challenges “companywide KG first”

Several independent lines of evidence argue against starting with a firmwide graph:

1. **GraphRAG underperforms vanilla RAG on many real tasks.** GraphRAG-Bench / “When to use Graphs in RAG” (ICLR 2026 track) reports GraphRAG frequently underperforms traditional RAG; cited prior work shows ~13% lower accuracy on Natural Questions-style fact retrieval and especially weak time-sensitive queries, while multi-hop gains come with ~2.3× latency ([arXiv:2506.05690](https://arxiv.org/html/2506.05690v3); [ICLR poster](https://iclr.cc/virtual/2026/poster/10007992)). Implication: graph structure is not a free upgrade for product lookup or cutsheet Q&A.

2. **Production routing often reserves graph for a small query share.** Practitioner write-ups of large GraphRAG deployments report routing ~7% of queries to GraphRAG, ~22% to standard RAG, and the majority to cheaper cache/CAG paths — with the claim that value concentrates in the multi-hop minority ([Particula, GraphRAG at 12M nodes](https://particula.tech/blog/graphrag-implementation-enterprise-data-platform)). Implication: companywide graph infrastructure can be oversized relative to query mix.

3. **Curated enterprise KG success correlates with prior KM / taxonomy maturity, not tooling.** Case contrasts (e.g., EY/Graphwise vs managed GraphRAG pitches) emphasize that firmwide graphs fail without systematic curation culture; technology alone does not fix content lifecycle debt ([GraphRAG Curator, Aug 2025](https://graphrag.info/2025/08/25/graphrag-compare-and-contrast-aws-lettria-versus-the-ey-graphwise-approach/)).

4. **Ontology cost and lag are first-class failure modes.** Property-graph advocates note that heavy RDF/ontology-first programs delay value and that ontologies perpetually lag the business ([Neo4j: RDF vs property graphs](https://neo4j.com/blog/knowledge-graph/rdf-vs-property-graphs-knowledge-graphs/)). Thin IT organizations feel this acutely.

5. **Product evidence graphs can be relational tables.** Manufacturing-adjacent “product evidence graph” guidance states a dedicated graph DB is **optional**; the operational model (canonical identity, document coverage, field-level provenance, human review) matters more than the store label — and recommends starting with **20–100 products / one family**, not the full catalog ([Claro product evidence graph guide, Jul 2026](https://getclaro.ai/resources/guides/build-product-evidence-graph/)).

6. **ACL leakage worsens as graphs grow and share entities.** Hybrid vector→graph expansion can amplify leakage via shared entities (vendors, SKUs, standards) that bridge authorized and unauthorized neighborhoods; community summaries mix restricted and public sources ([Retrieval Pivot Attacks, arXiv:2602.08668](https://arxiv.org/html/2602.08668v3); [GraphRAG community ACL notes](https://theneuralbase.com/graphrag/learn/advanced/access-control-for-communities/)). Larger companywide graphs increase shared-entity surface area.

7. **Prior QCo layered SoT already parks EKG/GraphRAG behind named sponsors** with certified-slice discipline on the agent path ([Options Memo](../../memos/QCo-Knowledge-Systems-Options-Memo.md); [Certified Slices Checklist](../../checklists/QCo-Certified-Slices-Minimums-Checklist.md)).

### 1.3 When a semantic KG *does* earn keep (evidence-based keep criteria)

A curated or hybrid graph earns keep when **named questions fail** under catalog + ACL-RAG + semantic/SoR — typically:

| Keep signal | Example manufacturing question | Why thinner layers struggle |
| --- | --- | --- |
| **Typed multi-hop** | “Which active SKUs share a superseding revision with cutsheet C, and what pricebook line applies in region R?” | Joins span ERP + PDFs + rules with different keys |
| **Contextual BOM / substitution** | Plant-specific BOM views, disposition levels, rule-based substitutions across PLM↔supply chain ([PLM–supply chain KG challenge framing](https://www.linkedin.com/pulse/challenge-knowledge-graph-vs-alternative-approaches-plm-supply-smith-vps4f)) | Rigid BOM tables lack context edges |
| **Evidence provenance as first-class** | “Show the page/section that supports rated voltage for variant B” | Chunk RAG cites loosely; evidence graphs bind field→page ([Claro guide](https://getclaro.ai/resources/guides/build-product-evidence-graph/)) |
| **Official relationship vocabulary** | Shared Product / BOM / Drawing / SOP / Rule types for agents and humans | Soft LLM schemas drift across extractors |
| **Audit / workspace isolation** | Provenance + principal-scoped traversal for enterprise agents ([Oxagen: KG vs RAG for agents](https://www.oxagen.ai/blog/knowledge-graphs-vs-rag-for-ai-agents)) | Vector-only indexes struggle to prove isolation |

**Earn-keep test (descriptive):** If three named, recurring agent/human failures are not multi-hop / provenance / substitution problems, graph investment is likely premature relative to certified ACL-RAG + SoR/semantic tools ([Oxagen upgrade heuristic](https://www.oxagen.ai/blog/knowledge-graphs-vs-rag-for-ai-agents); GraphRAG-Bench “when graphs help” framing ([arXiv:2506.05690](https://arxiv.org/html/2506.05690v3))).

### 1.4 Thin certified graph + MCP vs companywide KG (surface the counter-evidence)

| Dimension | Thin certified graph + read-only MCP | Companywide KG |
| --- | --- | --- |
| Scope | Initiative-scoped certified entities/edges only | Ambition of all products, docs, rules, orgs |
| Stewardship | Owners named per certified slice | Requires standing ontology + ER program |
| Agent safety | Smaller graph ⇒ fewer shared-entity pivots; refuse outside slice | Larger blast radius; community/edge leakage harder |
| Time-to-evidence | Weeks for one product family / pricebook slice | Quarters–years for firmwide coverage |
| Failure mode | “I don’t know / not in certified slice” (desirable) | Fluent traversal over incomplete/wrong edges (dangerous) |
| Fit with thin IT | Aligns with certify-critical-few discipline | Conflicts with thin stewardship capacity |

**Bottom line for framing:** Evidence favors treating companywide KG as a **hypothesis that must earn keep**, not a default architecture. Many manufacturing document problems are better framed as **certified product/BOM/evidence slices** plus ACL-RAG for narrative SOPs and SoR reads for live price/inventory — with graph edges only where typed relationships are the bottleneck.

---

## 2. Ontology / taxonomy design patterns (manufacturing)

*Patterns only — not a finished QCo ontology.*

### 2.1 Reusable industrial patterns

| Pattern | Core idea | Relevance to cutsheet / BOM / SOP corpora |
| --- | --- | --- |
| **ISA-95 / B2MML Product Definition** | Hierarchical product definitions, manufacturing bills, materials, quantities, production rules ([ISA-95](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard); [B2MML / MESA](https://mesa.org/topics-resources/b2mml/); [Digital Twin Consortium ISA95 ontologies](https://github.com/digitaltwinconsortium/ManufacturingOntologies/tree/main/Ontologies/ISA95)) | Anchors Product / Material / BOM hierarchy language when mapping ERP exports |
| **PPR (Product–Process–Resource)** | Separate product structure from process steps and resources | Links drawings/SOPs to operations without collapsing into one “doc” node |
| **FBS (Function–Behaviour–Structure)** | Design reasoning ontology populated from catalogs/spec sheets ([arXiv:2412.05868](https://arxiv.org/abs/2412.05868)) | Useful when cutsheets encode capabilities, not only SKUs |
| **Task-centric ontology (TCO) / OmEGa** | Task-centric KG extraction from multimodal manufacturing docs ([OmEGa, AEI 2025](https://doi.org/10.1016/j.aei.2024.103001)) | Aligns graph to *work tasks* (maintenance, setup) rather than only products |
| **AIC (Abstraction–Instance–Capability)** | Multi-grain industrial KG for customization / BOM verify ([Springer JIM](https://link.springer.com/article/10.1007/s10845-023-02216-y)) | BOM modification & capability matching patterns |
| **Industrial standards → hierarchical + propositional KG** | Sections, conditional rules, tables as atomic propositions ([alphaXiv:2512.08398](https://www.alphaxiv.org/abs/2512.08398)) | Pattern for **rules** and standards-like SOPs with exceptions |
| **Product evidence graph** | Product ↔ source record ↔ document ↔ attribute evidence ↔ review decision ([Claro](https://getclaro.ai/resources/guides/build-product-evidence-graph/)) | Directly maps cutsheet/drawing certification workflows |

### 2.2 Candidate concept families (illustrative taxonomy)

```
Identity & commerce
  ProductFamily → Model → Variant → Pack / SellableSKU
  Revision / EffectiveWindow
  PricebookLine / CurrencyList / CurrencyRule (region, qty break, currency)
  CustomerSegment / Channel (if needed for price applicability)

Structure
  BOM (EBOM / MBOM / plant-context BOM)
  BOMLine (qty, UoM, find-number, alternate, substitute)
  Component / Material / Assembly

Engineering artifacts
  Drawing (DWG/PDF, sheet, revision)
  Cutsheet / Datasheet / SpecSheet
  AttributeEvidence (attribute, value, unit, page/section, method, confidence)

Operations & compliance
  SOP / WorkInstruction / ProcedureStep
  Rule / Constraint / Exception (conditional propositions)
  Certificate / TestReport / Declaration (expiry)

Provenance & governance (agent-critical)
  SourceRecord (system, raw id, ingested_at)
  CertificationStatus (candidate | certified | stale | tombstoned)
  Owner / ReviewDecision
  ACLPrincipal tags (or pointers to live authz)
```

### 2.3 Relationship patterns (typed edges that usually matter)

| Edge (illustrative) | From → To | Why it exists |
| --- | --- | --- |
| `HAS_VARIANT` / `HAS_PACK` | Family/Model → Variant/SKU | Prevents family-level evidence leaking to unit claims |
| `CONTAINS` / `BOM_LINE` | Assembly → Component | Structural multi-hop |
| `SUPERSEDES` | Revision → Revision | Stale-edge control |
| `DOCUMENTED_BY` / `COVERS` | Product → Drawing/Cutsheet | Scope: model vs variant vs exclusions |
| `EVIDENCES` | Document locus → Attribute | Field-level provenance |
| `PRICED_BY` | SKU → PricebookLine | Commerce queries without inventing SQL |
| `GOVERNED_BY` | Product/Process → Rule/SOP | Rule applicability |
| `SUBSTITUTES` / `ALTERNATE` | Component → Component | Supply-chain flexibility (contextual) |
| `EXTRACTED_FROM` / `SUPPORTED_BY` | Claim/Edge → Source chunk | Agent citations |

### 2.4 Design heuristics observed in literature

- **Separate levels of applicability** (family vs model vs variant vs pack) for every attribute and document ([Claro](https://getclaro.ai/resources/guides/build-product-evidence-graph/)).
- **Field-level authority matrix** (ERP owns price; engineering datasheet owns dimensions; compliance owns certificates) — avoid “one SoR wins all fields.”
- **Prefer shallow, strict schemas for agent-facing slices**; allow richer research ontologies offline. Schema-guided extractors (e.g., LlamaIndex `SchemaLLMPathExtractor`) exist specifically to constrain LLM triples ([LlamaIndex PropertyGraph docs](https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/)).
- **Map to ISA-95/B2MML concepts where ERP already speaks that language**; do not invent parallel vocabularies for the same BOM fact ([ISA-95](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard)).
- **Rules as propositions**, not free text only — industrial-standard KG work decomposes conditional/numerical rules for multi-hop QA ([alphaXiv:2512.08398](https://www.alphaxiv.org/abs/2512.08398)).

---

## 3. Construction pipeline

### 3.1 Reference pipeline (manufacturing documents → certified graph)

```
Sources (certified-slice candidates only for agent path)
  PDFs: cutsheets, drawings, SOPs, price lists
  Structured: ERP/NetSuite exports, BOM tables, pricebook
  Optional: CAD metadata, supplier XLSX
        │
        ▼
[1] Ingest & preserve raw   → immutable SourceRecord + bytes/pointer + ACL snapshot
[2] Parse / layout          → sections, tables, figures, page coords (OCR if needed)
[3] Classify doc type       → cutsheet | drawing | SOP | pricebook | rule | other
[4] Extract candidates      → entities, attributes, relations (rules + LLM + table parsers)
[5] Entity resolution       → link to canonical Product/SKU/BOM ids (SoR keys preferred)
[6] Conflict & gap detect   → dual values, expired docs, family→variant misuse
[7] Human-in-loop certify   → approve / reject / needs-review (gate to agent path)
[8] Materialize edges       → only certified assertions; provenance mandatory
[9] Index dual plane        → graph store + (optional) vectors on chunks/entities
[10] Serve read-only API/MCP under user identity + query-time ACL
```

This aligns with product-evidence workflows ([Claro 16-step guide](https://getclaro.ai/resources/guides/build-product-evidence-graph/)), OmEGa multimodal IE ([doi:10.1016/j.aei.2024.103001](https://doi.org/10.1016/j.aei.2024.103001)), industrial-standard hierarchical+propositional extraction ([alphaXiv:2512.08398](https://www.alphaxiv.org/abs/2512.08398)), and 2026 datasheet pipelines/benchmarks ([Journal of Intelligent Manufacturing datasheet pipeline](https://link.springer.com/article/10.1007/s10845-026-02894-4); related AAS/PDF work: [eclipse-basyx/basyx-pdf-to-aas](https://github.com/eclipse-basyx/basyx-pdf-to-aas), [AAS-RAIL arXiv:2609.07334](https://arxiv.org/abs/2609.07334)).

### 3.2 Extraction notes by artifact type

| Artifact | Hard parts | Patterns seen |
| --- | --- | --- |
| **Cutsheets / datasheets** | Tables, multi-column layout, units, variant matrices | Schema-level extraction to hierarchical JSON; page-level evidence; company-specific few-shot ([datasheet pipeline 2026](https://link.springer.com/article/10.1007/s10845-026-02894-4)) |
| **Drawings** | Title block, rev, balloon/find-numbers, sparse text | Often metadata + human link to SKU; full geometry IE rarely worth SMB cost |
| **BOM tables** | Multiple BOM types, alternates, plant context | Prefer SoR/ERP as identity spine; docs *annotate* rather than invent BOM |
| **Pricebook** | Qty breaks, regions, effective dates | Structured import > LLM; graph links SKU→price line with validity windows |
| **SOPs / rules** | Conditionals, exceptions, “shall/should” | Propositional decomposition + hierarchy ([industrial standards KG](https://www.alphaxiv.org/abs/2512.08398)) |
| **Catalog PDFs lacking context** | Structured but non-ontological | Rule-based FBS mapping from legacy catalogs ([arXiv:2412.05868](https://arxiv.org/abs/2412.05868)) |

### 3.3 Entity resolution (ER)

Observed practice:

- Prefer **SoR keys** (NetSuite item id, MPN+manufacturer, internal SKU) as canonical ids; treat PDF strings as aliases.
- Store match status: `candidate | auto-approved | approved | rejected | needs_review` with signals and reviewer ([Claro](https://getclaro.ai/resources/guides/build-product-evidence-graph/)).
- ER quality bounds graph quality — large GraphRAG postmortems put ER *before* graph tech ([Particula](https://particula.tech/blog/graphrag-implementation-enterprise-data-platform)).
- Clustering/dedup of LLM-extracted entities is a distinct stage (synonym/cluster pipelines in OmEGa-class systems).

### 3.4 Human-in-the-loop certification gates (agent-path aligned)

Consistent with certified-slice minimums ([checklist](../../checklists/QCo-Certified-Slices-Minimums-Checklist.md)):

| Gate | Enter agent path only if… |
| --- | --- |
| Criticality | Accuracy of a named initiative rides on the assertion |
| Owner | Named human owner + review cadence |
| Provenance | Source doc/SoR id + locus (page/row) + timestamps |
| ACL | Query-time principal filter possible; no service-superuser retrieve |
| Conflict policy | Conflicts visible; silent overwrite forbidden for high-impact fields |
| Refuse-thin | Missing/expired/uncertified → agent must refuse or escalate |
| Write policy | **No standing agent write** to production SoR; exports are human- or workflow-approved |

LLM extraction produces **candidates**. Certification produces **agent-visible truth**. Conflating the two recreates GraphRAG-as-SoT failure modes.

---

## 4. Store models: RDF/OWL vs property graph vs hybrid

### 4.1 Comparison (SMB / thin-IT lens)

| Dimension | RDF / OWL (+ SPARQL, SHACL) | Property graph (LPG; Cypher/GQL/Gremlin) | Hybrid (common enterprise pattern) |
| --- | --- | --- | --- |
| Strength | Shared IRIs, ontology reasoning, SHACL validation, federation | Developer ergonomics, edge properties, fast traversal, incremental modeling | RDF as canonical semantics; LPG as serving/projection |
| Weakness for thin IT | Ontology skill scarcity; slower incremental change; verbose n-ary relations ([Neo4j comparison](https://neo4j.com/blog/knowledge-graph/rdf-vs-property-graphs-knowledge-graphs/)) | Weaker built-in open-world reasoning; schema discipline must be imposed | Sync lag between layers; highest ops cost ([Data AI Hub](https://www.dataaihub.co/learn/rdf-vs-property-graph); [Enterprise Knowledge](https://enterprise-knowledge.com/cutting-through-the-noise-an-introduction-to-rdf-lpg-graphs/)) |
| Query | SPARQL; reasoning may be non-terminating if misused | Pattern matching; predictable locality | Split: SPARQL for ontology; Cypher for apps |
| Managed options | Neptune RDF mode; GraphDB/Stardog-class | Neo4j; Neptune PG; Fabric Graph | Neptune supports both modes but **not auto-synced** ([AWS Neptune docs](https://docs.aws.amazon.com/neptune/latest/userguide/migration-compatibility.html); [AWS Prescriptive Guidance semantic layer](https://docs.aws.amazon.com/prescriptive-guidance/latest/semantic-layer-agentic-ai-ontology-reasoning-virtual-knowledge-graph/technology-tradeoffs-alternatives.html)) |
| SMB fit signal | Higher when multi-org standards exchange is the goal | Higher when one team owns app traversals and speed-to-slice | Usually overkill until multi-system semantic integration is proven |

Contemporary guidance often: **property graph by default for application graphs; layer ontology/taxonomy principles when needed** ([Neo4j](https://neo4j.com/blog/knowledge-graph/rdf-vs-property-graphs-knowledge-graphs/)). Product evidence graphs explicitly allow **relational tables** as the first store ([Claro](https://getclaro.ai/resources/guides/build-product-evidence-graph/)).

### 4.2 Fabric Ontology (preview) as a third pattern

Microsoft Fabric Ontology (preview) binds entity types/properties/relationships to OneLake sources, materializes an ontology graph, and supports NL2Ontology — with explicit note that upstream updates require **graph refresh** before visibility ([Microsoft Learn: Ontology overview](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)). This is closer to a **governed semantic binding layer** than to LLM GraphRAG. Fit depends on already living in Fabric/OneLake; preview maturity and refresh ops are caveats.

### 4.3 Practical store selection factors (descriptive)

- Existing cloud gravity (AWS → Neptune/Bedrock; Azure/Fabric → Ontology/Graph; multi-cloud → Neo4j Aura or dual).
- Need for **edge properties** (confidence, effectiveAt, ACL tags) → LPG natural.
- Need for **SHACL validation of extractions** → RDF or external validator onto LPG.
- Thin IT → fewer moving parts: one LPG *or even* certified tables + API before a graph product.

---

## 5. GraphRAG / hybrid Graph+vector vs curated EKG

*Reuses prior pack distinctions; updated with 2025–2026 sources.*

### 5.1 Definitional contrast (do not conflate)

| Dimension | **Curated enterprise KG (EKG)** | **GraphRAG-style / LLM-extracted graph** |
| --- | --- | --- |
| Primary input | SoR + stewarded ontology + certified docs | Unstructured corpora |
| Schema | Explicit, governed | Soft / emergent (optionally schema-guided) |
| Role | Relationship / meaning SoT | Retrieval & sensemaking index |
| Truth model | Asserted facts + provenance + certification | Chunk-grounded generation |
| Best questions | “What is X officially related to Y?” | “What themes span this corpus?” / some multi-hop narrative |
| Governance | High | Lower schema load; **ACL still hard** |
| Agent authority | Only if certified + ACL’d | Tool for exploration — not transactional truth |

(Prior pack cross-cut: [QCo-Knowledge-Systems-Research-Pack §B](./QCo-Knowledge-Systems-Research-Pack.md); Hogan et al. [ACM CSUR](https://dl.acm.org/doi/10.1145/3447772); GraphRAG [arXiv:2404.16130](https://arxiv.org/abs/2404.16130).)

### 5.2 Hybrid Graph+vector patterns (2025–2026)

| Pattern | Mechanism | Caveat |
| --- | --- | --- |
| **Microsoft GraphRAG / LazyGraphRAG** | Extract entities/claims → communities (Leiden) → summaries; Lazy defers LLM cost ([MSR LazyGraphRAG](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/); [docs](https://microsoft.github.io/graphrag/); [GitHub](https://github.com/microsoft/graphrag)) | OSS GraphRAG largely maintenance-mode; indexing cost historically high; community ACL weak |
| **Amazon Bedrock KB GraphRAG (GA Mar 2025)** | Managed extraction into **Neptune Analytics** + vectors; vector + traversal ([AWS What’s New](https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-bedrock-knowledge-bases-graphrag-generally-available/); [build graphs guide](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-build-graphs-build.html)) | Still LLM-extracted; validate ACL/tenant isolation; not a stewarded BOM SoT |
| **LlamaIndex PropertyGraphIndex** | Modular extractors (`SchemaLLMPathExtractor`, etc.) + Neo4j/Neptune stores; synonym/vector/TextToCypher retrievers ([docs](https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/)) | Text-to-Cypher risk; schema enforcement optional (`strict`) |
| **Neo4j Knowledge Layer / GraphRAG** | Production LPG + hybrid retrieval + MCP ([Neo4j MCP](https://neo4j.com/developer/genai-ecosystem/model-context-protocol-mcp/); [Knowledge Layer](https://neo4j.com/product/knowledge-layer/)) | Buyer still owns ontology/ER/ACL design |
| **Fast GraphRAG & variants** | Lower-cost exploration ([circlemind-ai/fast-graphrag](https://github.com/circlemind-ai/fast-graphrag)) | Research/ops maturity varies |

### 5.3 When graphs help RAG (updated evidence)

GraphRAG-Bench argues graphs help more on hierarchical retrieval and deep contextual reasoning than on simple factoid lookup; vanilla RAG often wins on fact retrieval and time-sensitive questions ([arXiv:2506.05690](https://arxiv.org/html/2506.05690v3)). Manufacturing implication:

- **Cutsheet attribute lookup, SKU price, SOP step text** → certified RAG / SoR / thin evidence tables often sufficient.
- **Multi-hop “which variants share drawing rev + price rule + substitute?”** → curated edges earn keep.
- **“Themes across five years of quality narratives”** → GraphRAG-class optional tool — not pricebook truth.

### 5.4 Hybrid architecture pattern commonly described

```
Certified curated slice (Product/BOM/Price/Doc links)  ← agent SoT for relationships
        +
ACL-aware vector/hybrid RAG over certified narrative docs ← SOPs, explanations
        +
SoR/semantic tools (ERP reads, metrics)                 ← live numbers; no agent write
        +
Optional GraphRAG index over broad corpora              ← thematic only; not certified SoT
```

Routing layers that keep GraphRAG to a small % of traffic appear repeatedly in production narratives ([Particula](https://particula.tech/blog/graphrag-implementation-enterprise-data-platform)).

---

## 6. ACL / identity at query time on graph traversal

### 6.1 Why graphs are harder than document RAG

In vector RAG, query-time pre-filter on chunk ACL is relatively well understood ([prior ACL research](./enterprise-knowledge-rag-acl-research.md); [Azure document-level access](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)). Graphs add:

1. **Shared entity pivots:** An authorized chunk mentions a shared SKU/vendor → traversal reaches unauthorized chunks via the entity ([Retrieval Pivot Attacks, arXiv:2602.08668](https://arxiv.org/html/2602.08668v3)). Measured RPR ≈ 0.95 in synthetic multi-tenant tests; leakage at pivot depth 2 as a structural bipartite chunk–entity effect.
2. **Community summaries:** Leiden communities merge entities across ACL domains; pre-computed summaries can leak restricted content into “global” answers ([Neural Base: access control for communities](https://theneuralbase.com/graphrag/learn/advanced/access-control-for-communities/)).
3. **Edge inference:** Even without returning forbidden node properties, *existence* of an edge can be sensitive (customer↔price exception, etc.).
4. **Label-free entities:** Pipelines often leave entity nodes without ownership labels because many docs mention them — the structural root cause called out in pivot-attack work.

### 6.2 Controls described in literature / practice

| Control | What it does | Notes |
| --- | --- | --- |
| **Query-time pre-filter on entry points** | Only ACL-visible docs/entities seed traversal | Necessary but **insufficient** alone if expansion is unchecked |
| **Per-hop authorization** | Re-check labels on every entity→chunk (or node→node) expansion | Eliminated measured RPR in pivot-attack evaluation with &lt;1 ms overhead ([arXiv:2602.08668](https://arxiv.org/html/2602.08668v3)) |
| **Index-per-role / source isolation** | Separate graphs or community indexes by clearance | Scales poorly beyond few roles; fragments semantics ([community ACL patterns](https://theneuralbase.com/graphrag/learn/advanced/access-control-for-communities/)) |
| **Certified-slice containment** | Agent tools only see certified subgraph | Shrinks shared-entity blast radius; aligns thin IT |
| **Policy-protected VKG / RBAC policies** | Role-sensitive policies in virtual KGs ([IJCKG 2025 PPVKG](https://www.inf.unibz.it/~calvanese/papers-html/IJCKG-2025-privacy.html)) | Strong for ontology-mediated DB access; different stack |
| **No post-filter-only** | Dropping hits after global retrieve | Same anti-pattern as RAG; worse with graph fan-out |

### 6.3 OBO / user-identity patterns for agents

Agents must not retrieve under a privileged service identity. Patterns:

- **OAuth 2.0 Token Exchange (RFC 8693)** for delegation; preserve user as `sub`, agent as actor (`act`) ([RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693)).
- Emerging **OAuth AI agents on-behalf-of-user** draft for explicit user consent to named agents ([draft-oauth-ai-agents-on-behalf-of-user](https://datatracker.ietf.org/doc/html/draft-oauth-ai-agents-on-behalf-of-user-02); [WorkOS explainer](https://workos.com/blog/oauth-on-behalf-of-ai-agents)).
- **Per-hop PBAC** on delegated calls so scope cannot widen mid-chain ([Cerbos: per-hop PBAC](https://www.cerbos.dev/blog/per-hop-pbac-delegation)).
- Microsoft-style OBO / `x-ms-query-source-authorization` patterns for search; equivalent user context required on graph APIs ([Azure ACL overview](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)).

**Audit minimum:** log principal, resolved groups, entry node ids, hop depth, edge ids returned, certification flags.

### 6.4 Manufacturing-specific ACL notes

- Pricebook and customer-specific rules are high-sensitivity — prefer SoR ACL + thin certified price edges over broad GraphRAG communities.
- Drawings may be ITAR / customer-restricted — community summaries over mixed corpora are especially hazardous.
- NetSuite roles ≠ graph roles; map explicitly; never assume ERP permission sync covers derived edges.

---

## 7. API / MCP exposure patterns for agents

### 7.1 Tool surface patterns (read-oriented)

Observed MCP / agent-tool patterns for graphs:

| Tool class | Example operations | Rationale |
| --- | --- | --- |
| **Schema discover** | `get_schema` / list entity types | Ground Text-to-Cypher; limit invention ([Neo4j MCP](https://neo4j.com/developer/genai-ecosystem/model-context-protocol-mcp/)) |
| **Read query** | `read_cypher` / `sparql_query` (read-only role) | Traversal under DB role that cannot write |
| **Constrained templates** | `CypherTemplateRetriever`-style param fill ([LlamaIndex](https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/)) | Safer than free Cypher generation |
| **Entity get** | `get_product`, `get_bom`, `get_price_lines` | Business verbs &gt; generic graph query for SMB |
| **Evidence / citations** | Return `entity_id`, `document_id`, `page`, `certification_status` | Anti–cognitive-surrender; matches certified-slice contract |
| **Provenance** | `get_provenance(entity_or_edge_id)` | CIO/audit questions ([Oxagen](https://www.oxagen.ai/blog/knowledge-graphs-vs-rag-for-ai-agents)) |
| **Refuse-thin** | Empty certified evidence → structured `insufficient_evidence` | Explicit non-answer |
| **Write tools** | Generally **absent** or human-gated for production SoR | Constraint: no standing agent write to SoR |

Neo4j’s official MCP servers expose schema, read/write Cypher, GDS, memory — production hardening implies **disable write**, bind OAuth, and prefer templates ([Neo4j MCP overview](https://neo4j.com/developer/genai-ecosystem/model-context-protocol-mcp/); [MCP technical deep dive](https://neo4j.com/blog/developer/model-context-protocol/)). Catalog MCP patterns similarly expose *certified* assets only ([prior approaches research](./enterprise-knowledge-approaches-research.md)).

### 7.2 Citation / entity ID contract

Agent responses grounded in a graph should carry, at minimum:

- Stable **entity IDs** (SKU, BOM id, doc id)
- **Edge type** used for the claim
- **Source locus** (doc + page/row or SoR record id)
- **Certification + freshness** flags (`certified`, `last_reviewed`, `tombstoned`)
- **ACL context** (that results were filtered — without leaking denied neighbors)

### 7.3 Refuse-thin / routing

Descriptive agent routing table for manufacturing questions:

| Question type | Prefer tool | Refuse when |
| --- | --- | --- |
| Policy / SOP narrative | ACL-RAG retrieve | No certified doc hit |
| Live price / inventory / order | SoR API under user ACL | Agent would need elevated ERP role |
| Official BOM / substitute / supersession | Certified graph traverse | Edge uncertified or stale |
| Themes across tickets/PDFs | Optional GraphRAG-class | Treated as SoT |
| Preference / chat memory | Agent memory | Never as company SoT |

### 7.4 Security notes for MCP

- Remote MCP: OAuth 2.0 / JWT per 2025 MCP auth updates; no long-lived DB passwords in prompts ([Neo4j MCP deep dive](https://neo4j.com/blog/developer/model-context-protocol/)).
- Treat free-form generated Cypher as untrusted — read-only DB user, allowlisted labels, row limits, timeout.
- Propagate end-user identity into every tool call (OBO); service accounts for *connectivity* only, not authorization.

---

## 8. Freshness, stale edges, ontology drift, invalidation

### 8.1 Freshness dimensions

| Dimension | What goes stale | Typical control |
| --- | --- | --- |
| Content | Cutsheet rev, SOP text, price | Connector/CDC; re-extract; version nodes |
| Edges | `COVERS`, `PRICED_BY`, `SUPERSEDES` | Validity windows; tombstones; cascade delete of derived edges |
| ACL | Group membership, SharePoint ACLs | Sync SLO + query-time resolve |
| Ontology | Types/predicates meaning | Versioned ontology releases + migration |
| Certification | “Approved” older than review cadence | Auto-demote to stale; agent refuse |

Fabric Ontology documents that upstream OneLake updates need **manual/scheduled graph refresh** before visibility ([Learn](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)) — a concrete freshness footgun.

### 8.2 Stale edges & tombstones

Patterns:

- Soft-delete / `tombstoned_at` on nodes and edges; agent queries filter `tombstoned IS NULL AND (valid_to IS NULL OR valid_to > now())`.
- On source delete or rev supersession: cascade removal of **derived** extractions while retaining audit history (seen in graph-memory systems’ delete planners / episode cascade PRs, e.g. Graphiti-style cascades — [getzep/graphiti#1491](https://github.com/getzep/graphiti/pull/1491)).
- `SUPERSEDES` edges preferred over silent overwrite so agents can explain “rev C replaces B.”
- Community summaries and embeddings must be **invalidated** when membership-changing edges change — otherwise global answers cite ghosts.

### 8.3 Ontology drift

Ontology drift = types/predicates/constraints diverge from business reality or from extraction prompts.

Controls described in enterprise KG architecture practice:

- Versioned ontology (semver); competency-query regression suite
- SHACL or application-level validation on write of certified edges
- Steward approval for predicate changes; backfill jobs for renamed types
- Avoid locking extractors to an unversioned prompt schema

RDF/OWL environments make drift visible via validation failures; LPG environments must **impose** equivalent gates or drift is silent ([Data AI Hub EKG architecture](https://www.dataaihub.co/learn/enterprise-knowledge-graph-architecture); Neo4j ontology-lag critique ([blog](https://neo4j.com/blog/knowledge-graph/rdf-vs-property-graphs-knowledge-graphs/))).

### 8.4 Invalidation aligned to certified slices

Per certified-slice checklist: owner can mark stale/tombstone; agent path drops on next sync; `last_reviewed` visible ([checklist](../../checklists/QCo-Certified-Slices-Minimums-Checklist.md)). Graph systems should treat certification demotion as a **first-class event**, not only document deletion.

---

## 9. Realistic 90-day slice patterns (industry-observed)

*Descriptive patterns seen in industry guides and manufacturing KG practice — **not** a QCo plan or recommendation.*

### 9.1 Common thin slices

| Slice | Entities / edges in scope | Typical sources | What gets demonstrated |
| --- | --- | --- | --- |
| **Product + evidence** | 20–100 products in one family; docs; attribute evidence; review states | ERP sample, supplier PDF, datasheets | Identity resolution, conflicts, provenance ([Claro](https://getclaro.ai/resources/guides/build-product-evidence-graph/)) |
| **Product + BOM** | One product line EBOM/MBOM; alternates | ERP/PLM export + drawings | Multi-hop contains; revision impact |
| **Product + pricebook** | SKUs + price lines + validity | NetSuite/price list + ACL | Commerce Q&A without Text-to-SQL sprawl |
| **Product + BOM + pricebook** | Above combined | ERP-centric | Classic “sellable configuration” demo |
| **Standards / rules QA** | One standards corpus → propositional KG | PDF standards/SOPs | Conditional rule QA ([alphaXiv:2512.08398](https://www.alphaxiv.org/abs/2512.08398)) |
| **Task-centric maintenance** | Tasks ↔ resources ↔ docs | Manuals + work orders | OmEGa-style task KG ([doi:10.1016/j.aei.2024.103001](https://doi.org/10.1016/j.aei.2024.103001)) |

### 9.2 What 90-day pilots usually *exclude*

- Firmwide ontology of all departments
- Unbounded GraphRAG over Teams/email
- Agent write-back to ERP
- Perfect entity resolution across all historical aliases
- Community-global search over mixed ACL corpora

### 9.3 Success metrics observed (descriptive)

- % SKUs with canonical id + ≥1 certified cutsheet link
- Conflict rate & mean time to review
- Permission-boundary tests (user A never traverses to doc B)
- Citation rate / refuse rate on eval questions
- Stale/tombstone propagation lag
- Query mix: share answered by SoR vs RAG vs graph (expect graph to remain minority if routing is sane)

---

## 10. Notable stacks / vendors / patterns (with caveats)

*Research inventory — not a pitch or ranking for QCo.*

| Offering / pattern | What it is | Caveats for manufacturing SMB / thin IT |
| --- | --- | --- |
| **Neo4j** (+ Aura, Knowledge Layer, MCP, GraphRAG) | Mature LPG; Cypher/GQL; agent MCP servers; hybrid retrieval ([MCP](https://neo4j.com/developer/genai-ecosystem/model-context-protocol-mcp/); [Knowledge Layer](https://neo4j.com/product/knowledge-layer/); [RDF vs PG](https://neo4j.com/blog/knowledge-graph/rdf-vs-property-graphs-knowledge-graphs/)) | Still need ER, ontology lite, ACL design; write MCP tools dangerous if enabled; cost/ops |
| **Amazon Neptune** (DB + Analytics) | Managed RDF and/or property graph; Bedrock GraphRAG backend ([compatibility](https://docs.aws.amazon.com/neptune/latest/userguide/migration-compatibility.html); [AWS PG semantic layer guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/semantic-layer-agentic-ai-ontology-reasoning-virtual-knowledge-graph/technology-tradeoffs-alternatives.html)) | RDF vs PG modes not auto-synced; OWL reasoner external; AWS gravity |
| **Amazon Bedrock Knowledge Bases GraphRAG** | Managed LLM graph+vector over S3 corpora; GA Mar 2025 ([What’s New](https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-bedrock-knowledge-bases-graphrag-generally-available/)) | Extracted ≠ curated BOM SoT; validate ACL; thematic/multi-hop assist |
| **Microsoft Fabric Ontology / IQ (preview)** | Entity types, bindings to OneLake, ontology graph, NL2Ontology ([Learn](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)) | Preview; refresh lag; best if already on Fabric |
| **LlamaIndex PropertyGraphIndex** | Extractors + retrievers over Neo4j/Neptune/etc. ([docs](https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/)) | Framework ≠ governance; Text-to-Cypher risk; schema `strict` optional |
| **Microsoft GraphRAG / LazyGraphRAG** | Research → OSS → managed paths ([paper](https://arxiv.org/abs/2404.16130); [Lazy](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/); [GitHub](https://github.com/microsoft/graphrag)) | Maintenance-mode OSS nuances; community ACL gaps; cost of full indexing |
| **Diffbot Knowledge Graph** | Web-scale **public** KG + extraction APIs ([product](https://www.diffbot.com/products/knowledge-graph); [approach](https://blog.diffbot.com/diffbots-approach-to-knowledge-graph/)) | Not an internal cutsheet/BOM SoT; useful for enriching public org/product facts only |
| **Ontop / VKG** | Virtual KG over relational DBs with ontology mappings; privacy/RBAC research ([PPVKG IJCKG 2025](https://www.inf.unibz.it/~calvanese/papers-html/IJCKG-2025-privacy.html)) | Strong when SoR already relational; different skill set |
| **Digital Twin Consortium Manufacturing Ontologies (ISA95)** | Reusable OWL modules ([GitHub](https://github.com/digitaltwinconsortium/ManufacturingOntologies/tree/main/Ontologies/ISA95)) | Starting vocabulary — not a populated graph |
| **Eclipse BaSyx PDF→AAS / AAS-RAIL** | Datasheet → Asset Administration Shell properties ([BaSyx](https://github.com/eclipse-basyx/basyx-pdf-to-aas); [arXiv:2609.07334](https://arxiv.org/abs/2609.07334)) | Industrial IoT/AAS path; may overfit if AAS not otherwise used |
| **Catalog “knowledge graphs” (Atlan, etc.)** | Metadata/lineage graphs for AI context ([Atlan framing](https://atlan.com/know/ai-agent/knowledge-graph/knowledge-graph-vs-graph-database/)) | Navigation authority ≠ product/BOM fact authority |
| **Relational “evidence graph”** | Tables for product–doc–evidence–review without a graph DB ([Claro](https://getclaro.ai/resources/guides/build-product-evidence-graph/)) | Often the rational 90-day substrate for thin IT |

---

## 11. Sources

### Prior QCo artifacts

- QCo Knowledge Systems Options Memo — `/workspace/qco/docs/knowledge-systems/memos/QCo-Knowledge-Systems-Options-Memo.md`
- Certified Slices Minimums Checklist — `/workspace/qco/docs/knowledge-systems/checklists/QCo-Certified-Slices-Minimums-Checklist.md`
- Knowledge Systems Research Pack — `/workspace/qco/docs/knowledge-systems/research/QCo-Knowledge-Systems-Research-Pack.md`
- Enterprise knowledge approaches / RAG-ACL research — sibling files in the same `research/` folder

### Foundational & surveys

- Hogan et al., “Knowledge Graphs,” *ACM Computing Surveys* (2021) — https://dl.acm.org/doi/10.1145/3447772
- GraphRAG: *From Local to Global* — https://arxiv.org/abs/2404.16130
- When to use Graphs in RAG / GraphRAG-Bench — https://arxiv.org/html/2506.05690v3 · https://iclr.cc/virtual/2026/poster/10007992
- Retrieval Pivot Attacks in Hybrid RAG — https://arxiv.org/html/2602.08668v3
- RFC 8693 OAuth 2.0 Token Exchange — https://datatracker.ietf.org/doc/html/rfc8693
- OAuth AI agents on-behalf-of-user draft — https://datatracker.ietf.org/doc/html/draft-oauth-ai-agents-on-behalf-of-user-02

### Manufacturing / industrial KG & extraction

- OmEGa ontology-based IE for manufacturing docs — https://doi.org/10.1016/j.aei.2024.103001
- FBS KG from product catalogues — https://arxiv.org/abs/2412.05868
- Ontology KG for industrial standard documents — https://www.alphaxiv.org/abs/2512.08398
- AIC industrial KG / BOM — https://link.springer.com/article/10.1007/s10845-023-02216-y
- Automated technical datasheets pipeline (2026) — https://link.springer.com/article/10.1007/s10845-026-02894-4
- AAS-RAIL PDF extraction — https://arxiv.org/abs/2609.07334
- eclipse-basyx/basyx-pdf-to-aas — https://github.com/eclipse-basyx/basyx-pdf-to-aas
- ISA-95 — https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard
- B2MML / MESA — https://mesa.org/topics-resources/b2mml/
- Digital Twin Consortium ISA95 ontologies — https://github.com/digitaltwinconsortium/ManufacturingOntologies/tree/main/Ontologies/ISA95
- Product evidence graph guide (Claro, Jul 2026) — https://getclaro.ai/resources/guides/build-product-evidence-graph/
- PLM–supply chain KG vs alternatives (challenge framing) — https://www.linkedin.com/pulse/challenge-knowledge-graph-vs-alternative-approaches-plm-supply-smith-vps4f

### Platforms & engineering blogs

- Microsoft Fabric Ontology overview — https://learn.microsoft.com/en-us/fabric/iq/ontology/overview
- Azure AI Search document-level access — https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview
- Microsoft LazyGraphRAG — https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/
- Microsoft GraphRAG docs / GitHub — https://microsoft.github.io/graphrag/ · https://github.com/microsoft/graphrag
- Bedrock Knowledge Bases GraphRAG GA — https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-bedrock-knowledge-bases-graphrag-generally-available/
- Bedrock build graphs — https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-build-graphs-build.html
- AWS Neptune Neo4j compatibility — https://docs.aws.amazon.com/neptune/latest/userguide/migration-compatibility.html
- AWS Prescriptive Guidance: semantic layer tech tradeoffs — https://docs.aws.amazon.com/prescriptive-guidance/latest/semantic-layer-agentic-ai-ontology-reasoning-virtual-knowledge-graph/technology-tradeoffs-alternatives.html
- Neo4j RDF vs property graphs — https://neo4j.com/blog/knowledge-graph/rdf-vs-property-graphs-knowledge-graphs/
- Neo4j MCP — https://neo4j.com/developer/genai-ecosystem/model-context-protocol-mcp/
- Neo4j MCP deep dive — https://neo4j.com/blog/developer/model-context-protocol/
- Neo4j Knowledge Layer — https://neo4j.com/product/knowledge-layer/
- LlamaIndex PropertyGraphIndex — https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/
- Diffbot Knowledge Graph — https://www.diffbot.com/products/knowledge-graph
- Enterprise Knowledge: RDF & LPG intro — https://enterprise-knowledge.com/cutting-through-the-noise-an-introduction-to-rdf-lpg-graphs/
- Data AI Hub: RDF vs property graph — https://www.dataaihub.co/learn/rdf-vs-property-graph
- GraphRAG Curator: AWS/Lettria vs EY/Graphwise — https://graphrag.info/2025/08/25/graphrag-compare-and-contrast-aws-lettria-versus-the-ey-graphwise-approach/
- Particula: GraphRAG at 12M nodes — https://particula.tech/blog/graphrag-implementation-enterprise-data-platform
- Oxagen: Knowledge graphs vs RAG for agents — https://www.oxagen.ai/blog/knowledge-graphs-vs-rag-for-ai-agents
- Neural Base: GraphRAG community access control — https://theneuralbase.com/graphrag/learn/advanced/access-control-for-communities/
- WorkOS: OAuth OBO for AI agents — https://workos.com/blog/oauth-on-behalf-of-ai-agents
- Cerbos: per-hop PBAC — https://www.cerbos.dev/blog/per-hop-pbac-delegation
- Forrester: semantics/ontologies for agentic AI — https://www.forrester.com/blogs/build-meaning-before-machines-why-semantics-ontologies-and-knowledge-graphs-matter-for-agentic-ai/
- Semantic layer vs knowledge graph (2026 framing) — https://colrows.com/blogs/semantic-layer-vs-knowledge-graph/
- PPVKG / Ontop access control (IJCKG 2025) — https://www.inf.unibz.it/~calvanese/papers-html/IJCKG-2025-privacy.html
- circlemind-ai/fast-graphrag — https://github.com/circlemind-ai/fast-graphrag

---

## Appendix A — One-page contrast card (for council packets)

| If the pain is… | Evidence leans toward… | Graph role |
| --- | --- | --- |
| Can’t find SOPs / cutsheets | ACL-aware hybrid RAG + citations | None or doc nodes only |
| Wrong KPIs / prices invented | Semantic layer / SoR tools | Link only, don’t compute |
| Don’t know which doc supports which variant attribute | Product evidence graph (even relational) | Optional |
| Multi-hop BOM / substitute / supersession under ACL | Thin **certified** curated graph + MCP | Primary for that slice |
| Themes across large messy corpora | Lazy/managed GraphRAG-class | Optional retrieval tool |
| “We need a companywide KG” without named failures | Challenge; demand earn-keep questions | Deferred |

---

## Appendix B — Explicit non-goals of this pack

- No QCo build recommendation, vendor shortlist ranking, or “Chris should…” guidance
- No redo of the full RAG/catalog landscape (see prior research pack)
- No finished QCo ontology or NetSuite field mapping
- No exploit-oriented security content — ACL patterns are defensive/architectural only

---

*End of research pack. Analyst synthesis and Council options remain separate work products.*
