# Enterprise Knowledge / Data Approaches — Research Section

*Research only (no recommendations). As of 2026-09-05. Emphasizes governance, lineage, ACL, and whether agents can treat the layer as authoritative.*

---

## 1) Data catalogs / metadata platforms (governance + discovery layer)

### What it is

A **data catalog / metadata platform** inventories technical and business metadata across warehouses, lakes, pipelines, BI tools, and (increasingly) AI assets. It provides search/discovery, business glossary, ownership, stewardship workflows, lineage, quality signals, and policy context. It is typically a **control and context plane over data**, not the system of record for business facts themselves.

**2026 positioning (verified):**

| Platform | Current positioning |
| --- | --- |
| **Collibra** | Governance-first “control and context plane” for data **and** AI: catalog, lineage, quality, privacy, stewardship workflows, AI inventory/registry; Leader framing in Gartner Data & Analytics Governance Platforms. |
| **Alation** | Evolved from classic catalog into **AIOS (Alation Intelligence Operating System)**: agents + context + data + governance + feedback loops; open edges (agents may run elsewhere); emphasis on self-improving context. |
| **Atlan** | Positions as **“context layer for AI”** / Context Lakehouse: active metadata graph + Iceberg-native store; MCP/SQL/API serving of certified definitions, lineage, and policy to agents. |
| **DataHub** (Acryl) | Active open-source, event-driven metadata platform (Kafka MCL); scale/automation focus; managed **DataHub Cloud**; native agent/MCP wiring in ecosystem comparisons. |
| **Amundsen** | **Effectively legacy for new builds** (stale releases/docs through 2025–2026; amundsen.io reported non-serving by mid-2026). Historical Lyft catalog; not a current peer to DataHub/OpenMetadata. |
| **OpenMetadata** (Collate) | Active OSS alternative to DataHub: simpler ops (no Kafka), governance + quality in one product; Collate managed option. |
| **Microsoft Purview** | Unified **Data Map + Unified Catalog**: federated governance domains, data products, glossary/CDEs with attached policies, self-service access; metadata-only (catalog roles ≠ data access). |
| **Google Knowledge Catalog** | **Rebrand/evolution of Dataplex Universal Catalog**: Gemini-powered context graph for discovery, quality, lineage, data products; MCP integrations; explicit agent grounding narrative. Legacy **Data Catalog** enters phased shutdown from **2026-06-01**. |
| **Snowflake Horizon Catalog** | Platform-native “agentic catalog”: discovery + semantic context + engine-enforced governance (RBAC, masking, row access) for humans, BI, and Cortex agents. |
| **Databricks Unity Catalog** | Lakehouse governance spine (tables, models, AI Search indexes, lineage); permissions bind to compute/query path. |

### How truthfulness, freshness, and permissions work

| Concern | Typical behavior |
| --- | --- |
| **Truthfulness** | Catalogs assert **truth about assets** (what exists, owners, certified status, lineage, glossary meaning). They do **not** automatically make query results correct. Authority for agents is strongest when the catalog exposes **certified data products / metrics / example queries** and agents are constrained to those objects. |
| **Freshness** | Ingestion via connectors, scanners, OpenLineage/events (DataHub), or native platform sync. Active-metadata vendors push near-real-time change streams; batch scanners lag schema/ownership changes. Glossary and certification often remain **human-gated**, so semantic freshness ≠ technical freshness. |
| **Permissions** | Two layers: (1) **who can see/edit metadata** in the catalog; (2) **who can access underlying data**. Purview docs emphasize catalog permissions do not grant data access. Platform catalogs (Horizon, Unity, Knowledge Catalog + BigQuery IAM) can bind closer to **runtime enforcement**. Cross-platform catalogs usually **document or orchestrate** policy and push to source systems; runtime ACL still lives in warehouses/apps unless integrated carefully. |

**Authoritativeness for agents:** Treat as authoritative for **“what may I use / what does it mean / who owns it / is it certified?”** Treat as **non-authoritative** for computed facts unless coupled to a semantic/query layer that executes governed definitions.

### Strengths

- Enterprise-scale **discovery** and shared vocabulary (glossary, domains, data products).
- **Lineage** and impact analysis for change management and AI training-data provenance.
- Stewardship, approval workflows, and **compliance artifacts** (esp. Collibra, Purview, Horizon).
- Emerging **MCP / agent APIs** so agents discover certified assets instead of scraping schemas.
- Bridges humans (stewards, analysts) and machines (agents needing context).

### Failure modes

- **Empty / stale catalog**: low adoption; metadata debt; agents inherit wrong owners or obsolete tables.
- **Catalog ≠ enforcement**: agents with warehouse credentials bypass catalog certifications.
- **Semantic drift**: glossary and “certified” flags lag business reality.
- **Implementation weight**: Collibra-class programs often 6–12 months; OSS DataHub ops complexity (Kafka, ES, graph).
- **False authority**: treating catalog search hits as ground-truth answers (they are pointers, not results).
- **Amundsen-era references** in RFPs that point teams at unmaintained stacks.

### Typical stack

Sources (Snowflake/BigQuery/Databricks/Redshift, dbt, Airflow/Spark, Tableau/Power BI/Looker) → metadata connectors / OpenLineage → catalog graph (Collibra / Alation / Atlan / DataHub / Purview / Knowledge Catalog / Horizon / Unity) → stewardship UI + APIs/MCP → optional policy sync to IAM/RLS/masking → consumers: humans, BI, agents.

### Fit for agents vs humans

| Audience | Fit |
| --- | --- |
| **Humans** | Primary UX historically: search, request access, understand lineage/quality, steward workflows. |
| **Agents** | Strong as **router/context broker** (find certified products, read definitions, respect tags). Weak as sole SOT for answers unless paired with executable semantics and source ACLs. 2026 products (Alation AIOS, Atlan MCP, Knowledge Catalog MCP, Horizon) explicitly target agent grounding. |

### Notable vendors / patterns

- Governance-first: **Collibra**
- Discovery → AIOS: **Alation**
- Active metadata / context layer: **Atlan**
- OSS event-driven: **DataHub**; simpler OSS: **OpenMetadata**
- Hyperscaler: **Purview**, **Knowledge Catalog (ex-Dataplex)**, **Horizon**, **Unity Catalog**
- Pattern: **federated data products** (domains + products + contracts) + **agent MCP over certified metadata**

### Sources / links

- Collibra positioning / Gartner Leader post: https://www.collibra.com/blog/collibra-recognized-as-a-leader-for-the-second-consecutive-year-in-the-gartner-r-magic-quadrant
- Collibra “Why Collibra”: https://www.collibra.com/why-collibra
- Alation AIOS announcement: https://www.alation.com/blog/introducing-aios-alation-intelligence-operating-system/
- Alation AIOS product page: https://www.alation.com/aios/
- Atlan context layer: https://atlan.com/context-layer/
- Atlan Context Lakehouse: https://atlan.com/context-lakehouse/
- Atlan docs — context layer: https://docs.atlan.com/agents/faq/context-layer
- Microsoft Purview Unified Catalog: https://learn.microsoft.com/en-us/purview/unified-catalog
- Purview data governance overview: https://learn.microsoft.com/en-us/purview/data-governance-overview
- Google Knowledge Catalog overview: https://docs.cloud.google.com/dataplex/docs/introduction
- Google Knowledge Catalog product: https://cloud.google.com/products/knowledge-catalog
- Knowledge Catalog FAQ (Data Catalog shutdown note): https://docs.cloud.google.com/dataplex/docs/faq
- Snowflake Horizon Catalog: https://docs.snowflake.com/en/user-guide/snowflake-horizon
- DataHub vs OpenMetadata (2026 field comparison): https://fastero.com/blog/datahub-vs-openmetadata-open-source-data-catalogs
- Amundsen maintenance warning: https://datatrail.ai/data-catalog-tools
- Collibra vs Alation vs Atlan vs Purview (2026): https://promethium.ai/guides/data-governance-tools-comparison-collibra-alation-atlan-purview/

---

## 2) Structured sources of truth + retrieval

*Systems of record / warehouses / lakes / document systems with citations; semantic layers; wiki/docs as agent grounding.*

### What it is

This approach treats **authoritative systems** as the place truth is computed or stored, and retrieval as the way humans/agents get answers **with provenance**:

1. **Systems of record (SoR)** — ERP, CRM, HRIS, billing, ticketing: operational truth for entities/transactions.
2. **Warehouses / lakehouses** — analytical truth (cleaned, modeled, historical); often the execution engine for metrics.
3. **Semantic layers** — governed metrics/dimensions/joins sitting above the warehouse so consumers (BI and agents) select certified measures instead of inventing SQL.
4. **Document / wiki systems with citations** — Confluence, SharePoint, Notion, Google Drive, etc., retrieved via RAG with **source links** and **ACL-aware** retrieval for policies, runbooks, and narrative knowledge.

These are complementary: SoR/warehouse answer “what is the number?”; docs answer “what is the policy / how do we do X?”

### How truthfulness, freshness, and permissions work

| Layer | Truthfulness | Freshness | Permissions |
| --- | --- | --- | --- |
| **SoR** | Authoritative for operational state if writes are controlled. | Real-time or near-real-time in apps; analytics copies lag. | App RBAC; agents need user-impersonation or scoped service roles. |
| **Warehouse / lakehouse** | Authoritative for **defined** analytical models; quality depends on pipelines/tests. | Batch/micro-batch/streaming; Time Travel / SCD patterns common. | Warehouse RBAC, RLS, column masking (Snowflake, BigQuery, Databricks). |
| **Semantic layer (dbt MetricFlow, Cube, Looker LookML)** | High for **metrics**: definitions versioned; SQL compiled from certified objects. Reduces metric hallucination. Does not fix wrong source data. | As fresh as warehouse tables + cache/pre-aggregations. | Best practice: **compile-time** row/role rules (Cube emphasis); Looker pass-through OAuth to Gemini; dbt SL via platform identity + warehouse roles. |
| **Docs + citation RAG** | “Truth” is whatever was written; citations enable human verification. Stale pages remain a risk. | Connector/change-feed dependent; ACL sync often slower than content sync. | **Must** filter by effective ACL at retrieval (SharePoint Graph, Confluence space/page restrictions); post-filter is leaky. |

**Authoritativeness for agents:**

- **Strong:** semantic-layer metric queries; SoR/API reads under user ACL; warehouse queries under RLS.
- **Moderate:** certified tables/views with tests and contracts.
- **Weak unless constrained:** free-form Text-to-SQL on raw schemas; unfiltered doc RAG.

### Strengths

- **Deterministic metrics** when semantic layer owns joins/grain/time logic (Looker→Gemini; Cube MCP; dbt Semantic Layer + MCP).
- **Citations** from doc RAG support audit and human trust.
- Warehouse/lakehouse provide scalable, auditable query history.
- Clear separation: operational SoR vs analytical SOT vs narrative knowledge.
- Platform agents (e.g., Snowflake Cortex Agents) can combine **Cortex Analyst (structured)** + **Cortex Search (unstructured)** under one RBAC model.

### Failure modes

- **Multiple versions of the truth** across BI tools without a shared semantic layer.
- **Text-to-SQL metric drift**: valid SQL, wrong business meaning; silent fan-out/chasm joins.
- **Permission stripping** in vector indexes (embed without ACL → over-sharing).
- **Doc rot**: high-citation answers from outdated Confluence pages.
- **Cache / export staleness** in semantic layers and metric exports.
- **SoR vs warehouse conflict** when agents mix live CRM with delayed warehouse snapshots without labeling freshness.

### Typical stack

**Structured path:** SoR → ELT (Fivetran/Airbyte/etc.) → warehouse/lakehouse (Snowflake/BigQuery/Databricks) → transforms (**dbt**) → semantic layer (**MetricFlow / Cube / LookML / warehouse semantic views**) → BI + **MCP/agent APIs**.

**Unstructured path:** SharePoint/Confluence/Drive/Notion → connectors → chunk/embed + **ACL metadata** → vector/hybrid search → LLM with **mandatory citations** → identity from Entra/Okta.

**Combined agents:** orchestrator routes KPI questions to semantic/SQL tools and policy questions to ACL-aware RAG.

### Fit for agents vs humans

| Audience | Fit |
| --- | --- |
| **Humans** | Dashboards, Explores, wiki search, ticket systems — familiar SOTs. |
| **Agents** | Excellent when tools are **narrow and governed** (query_metrics, Looker explores, Cortex Analyst). Risky when given broad `execute_sql` on production. Doc RAG fits agents for grounded Q&A **if** citations + ACL pre-filter are enforced. |

### Notable vendors / patterns

- **dbt MetricFlow / Semantic Layer** + **dbt MCP** (`list_metrics`, `query_metrics`): https://docs.getdbt.com/docs/build/about-metricflow · https://docs.getdbt.com/docs/dbt-ai/about-mcp · https://docs.getdbt.com/docs/use-dbt-semantic-layer/sl-architecture
- **Cube** (Cube Core OSS semantic layer; MCP; compile-time RLS): https://cube.dev/articles/semantic-layer-for-ai-agents-2026 · https://github.com/cube-js/cube
- **Looker LookML** as governed semantic foundation for Gemini Enterprise (A2A, OAuth pass-through, no DB copy): https://cloud.google.com/blog/products/business-intelligence/integrating-looker-and-gemini-enterprise · golden queries context: https://docs.cloud.google.com/gemini/data-agents/conversational-analytics-api/data-agent-authored-context-looker
- **Snowflake Cortex Agents** (Analyst + Search + Horizon governance): https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents
- **Permission-aware enterprise RAG** (SharePoint/Confluence patterns): https://wavect.io/blog/rag-permissions-sharepoint-confluence-drive/ · https://tianpan.co/blog/2026/04/17/vector-store-access-control-rag-rls

### Sources / links (additional)

- Looker agentic BI (Next ’26): https://cloud.google.com/blog/products/business-intelligence/looker-updates-for-agentic-bi-at-next26
- dbt MCP blog: https://www.getdbt.com/blog/mcp

---

## 3) Material hybrid / alternative patterns (brief)

*Compared as alternatives or complements to “plain RAG” or standalone knowledge graphs for **company knowledge as SOT**.*

### 3A) Lakehouse + vector / hybrid search indexes

**What it is:** Keep governed tables/files in a lakehouse and add **vector or hybrid (BM25 + embedding)** indexes co-governed with the same catalog (e.g., Databricks **AI Search** on Delta under Unity Catalog; Snowflake **Cortex Search**; Lakebase/pgvector-style app DBs). Unstructured and structured share one permission spine.

**Truth / freshness / permissions:** Truth remains lakehouse tables + indexed snapshots; freshness via Delta sync / pipelines; permissions ideally **Unity Catalog / Horizon / IAM** on the index, not a separate ungoverned vector SaaS.

**Strengths:** One platform for ETL, SQL, RAG; lineage to training/retrieval data; hybrid keyword+semantic for SKUs/IDs.

**Failure modes:** Index lag; embedding without ACL; treating similarity hits as transactional truth; cost of re-embed on schema/doc churn.

**Typical stack:** Delta/Iceberg + Unity/Horizon + AI Search/Cortex Search + agent runtime.

**Agents vs humans:** Strong for agent RAG with enterprise ACL; humans still use SQL/BI for precise metrics.

**Vendors/patterns:** https://docs.databricks.com/aws/en/ai-search/ai-search · https://docs.databricks.com/aws/en/agents/retrieval-augmented-generation · https://docs.snowflake.com/en/user-guide/snowflake-horizon

**Authoritative for agents?** Authoritative for **retrieval under platform ACL**; not a substitute for semantic metrics.

---

### 3B) RAG over structured data / Text-to-SQL (and agentic SQL)

**What it is:** NL→SQL (or multi-step agents: schema prune → generate → execute → reflect) against warehouses/APIs. Distinct from document RAG: the “answer” is a **computed result**, not a passage.

**Truth / freshness / permissions:** Correctness hinges on schema understanding and metric definitions; freshness = live query; permissions must be **user-scoped** (staged authorize-then-execute). Research (2026) stresses authorization as a first-class dimension, not prompt trust.

**Strengths:** Ad-hoc analytical reach; no need to pre-index all answers; pairs well with catalogs for schema context.

**Failure modes:** Metric hallucination; authorization bypass via clever SQL; brittle under schema evolution; hard provenance for non-technical users.

**Typical stack:** Warehouse + schema metadata (catalog/semantic) + Text-to-SQL agent + SQL sandbox + audit log; optionally semantic layer to constrain generation.

**Agents vs humans:** High leverage for agents; humans prefer semantic/BI for standard KPIs. Industry guidance: prefer semantic `query_metrics` over raw SQL for production KPIs.

**Sources:** https://arxiv.org/html/2608.19235 · https://cube.dev/articles/semantic-layer-for-ai-agents-2026 · https://www.getdbt.com/blog/mcp

**Authoritative for agents?** Only when constrained (semantic objects, RLS, evals). Unconstrained Text-to-SQL is **not** a reliable SOT.

---

### 3C) Agent memory architectures (Mem0, Zep/Graphiti, Letta, etc.)

**What it is:** Persistent memory outside the context window: extracted facts (Mem0), **temporal knowledge graphs** (Zep/Graphiti), or agent-managed memory blocks/OS paging (Letta/MemGPT lineage). Stores what agents **learned from interaction**, not the enterprise warehouse.

**Truth / freshness / permissions:** Truth is conversational/derived; freshness via invalidation windows (Zep) or overwrites; enterprise ACL/retention still immature vs data platforms—risk of storing PII or unverified “org facts.”

**Strengths:** Personalization, multi-session continuity, temporal fact succession; useful for user/agent state.

**Failure modes:** **Not a company SOT**—can contradict CRM/warehouse; poisoning via bad extractions; weak shared governance across agents unless backed by a context repository.

**Typical stack:** Agent runtime + memory API/graph + optional sync from business systems.

**Agents vs humans:** Agent-centric; humans rarely browse memory stores as SoR.

**Sources:** https://coworker.ai/blog/mem0-vs-zep-vs-letta · https://www.developersdigest.tech/blog/best-ai-agent-memory-providers-2026 · https://www.getzep.com/ · Atlan note that memory ≠ database semantic context: https://docs.atlan.com/agents/faq/context-layer

**Authoritative for agents?** Authoritative for **session/user preference state**; **not** for enterprise metrics or regulated records unless mirrored from governed systems.

---

### 3D) Multi-agent knowledge / shared context repositories

**What it is:** Shared, versioned **context stores** (Atlan Context Lakehouse; catalog-backed MCP servers; platform agent galleries) that multiple agents read/write under governance—closer to an operationalized metadata/KG than to chat memory.

**Truth / freshness / permissions:** Certified context with human-on-the-loop; bidirectional writes (observations, evals) with audit; policy attached to context delivery.

**Strengths:** Avoids per-agent private “truth”; compounds corrections (Alation AIOS feedback loops; Atlan Observe).

**Failure modes:** Certification bottlenecks; write-back spam; still dependent on underlying data quality.

**Sources:** https://atlan.com/context-lakehouse/ · https://www.alation.com/aios/ · https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents

**Authoritative for agents?** Can be treated as authoritative for **definitions, lineage, and allowed tools**; execution truth remains in warehouse/SoR.

---

## Cross-cutting: when can agents treat a layer as authoritative?

| Layer | Treat as authoritative for… | Do not treat as authoritative for… |
| --- | --- | --- |
| Data catalog / metadata | Asset identity, ownership, certification, glossary, lineage pointers, policy tags | Computed KPIs, row-level facts (unless query path enforces) |
| Semantic layer | Metric definitions, joins, grain, governed KPI results | Narrative explanations beyond the result; unmodeled ad-hoc logic |
| Warehouse / SoR under ACL | Live query results / transactional reads | Undocumented columns used as “metrics” |
| Doc RAG + citations | Quoted policy text with link + timestamp | Numeric KPIs; anything without ACL filter |
| Vector index on lakehouse | Similarity retrieval of allowed chunks | Sole SOT without SQL/semantic fallback |
| Agent memory | User/session preferences, prior tool outcomes | Org-wide financial/HR truth |

---

## Quick comparison matrix

| Approach | Primary job | Governance center of gravity | Agent authority |
| --- | --- | --- | --- |
| Catalogs / metadata platforms | Discover + govern + contextualize assets | Stewardship workflows, lineage, glossary, AI inventory | High for *navigation/certification*; low alone for *answers* |
| Structured SoT + semantic + cited docs | Compute / retrieve answers with provenance | Warehouse RLS + semantic compile-time rules + doc ACL | High when tools are constrained |
| Lakehouse + vector | Unified governed retrieval + analytics | Platform catalog (Unity/Horizon/Knowledge Catalog) | Medium–high for RAG; pair with semantics for KPIs |
| Text-to-SQL / structured agents | Ad-hoc computation | Must be designed in (authorize stages) | Conditional |
| Agent memory | Continuity / personalization | Often weakest enterprise control plane | Low for company SoT |

---

*End of research section. No product or architecture recommendations included.*
