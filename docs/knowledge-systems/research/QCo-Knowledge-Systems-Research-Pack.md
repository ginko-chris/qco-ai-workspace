# QCo Knowledge Systems — Landscape Research Pack

**From:** Knowledge Systems Researcher  
**To:** QCo Analyst (analysis-ready; research only — no QCo recommendations)  
**Date:** 2026-09-05  
**Audience context:** Director of AI Technology / QCo AI Council — enterprise framing (governance, freshness, permissions/ACL, humans + AI agents as consumers of sources of truth). Zero-retention / no-training-on-company-data constraints are deployment choices layered on these architectures, not properties of the approaches themselves.

**Approaches covered:**
1. Classical / vector RAG (incl. hybrid BM25+vector, agentic RAG)
2. Enterprise search + ACL-aware retrieval
3. Knowledge graphs (curated enterprise KG / semantic layer)
4. GraphRAG and graph+retrieval hybrids
5. Data catalogs / metadata platforms
6. Structured sources of truth + retrieval (SoR, warehouse/lakehouse, semantic layers, cited docs)
7. Material hybrids/alternatives (lakehouse+vector, Text-to-SQL, agent memory, shared context repositories)

**How to use:** Each section covers what it is; truthfulness / freshness / permissions; strengths; failure modes; typical stack; fit for agents vs humans; notable vendors/patterns; sources. Cross-cutting tables appear at section ends. Synthesize options and recommendations for QCo yourself — this pack does not rank or recommend.

---


# Part A — RAG & Enterprise Search + ACL

---

## 1) Classical / Vector RAG (Retrieval-Augmented Generation)

### What it is

Retrieval-Augmented Generation (RAG) couples a generative language model with a non-parametric memory that is queried at inference time. The canonical formulation (Lewis et al., NeurIPS 2020) uses a dense retriever over a vector index of passages plus a seq2seq generator that conditions on retrieved evidence, so answers are grounded in fetched text rather than parametric memory alone ([arXiv:2005.11401](https://arxiv.org/abs/2005.11401); [NeurIPS PDF](https://proceedings.nips.cc/paper_files/paper/2020/file/6b493230205f780e1bc26945df7481e5-Paper.pdf)).

In enterprise practice (2024–2026), “classical / vector RAG” usually means: ingest documents → chunk → embed → store in a vector (or hybrid) index → at query time retrieve top-*k* passages → optionally rerank → stuff context into a prompt → generate with citations. Production stacks have largely moved beyond “embed + cosine only” toward **hybrid retrieval** (BM25/sparse + dense) and, for harder queries, **agentic RAG** (retrieval as a tool inside a reason–act loop). Graph-oriented variants (e.g., Microsoft GraphRAG / LazyGraphRAG) are related but distinct: they add graph/community structure for global or multi-hop synthesis and are cited here only as contrast ([Microsoft Research LazyGraphRAG](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/); [microsoft/graphrag](https://github.com/microsoft/GraphRAG); [docs](https://microsoft.github.io/graphrag/)).

**Pattern distinctions (material for architecture reviews):**

| Pattern | Mechanism | Typical use |
| --- | --- | --- |
| **Chunked vector RAG** | Dense embeddings of fixed/semantic chunks; ANN (HNSW/IVF) top-*k* | Conceptual / paraphrased questions |
| **Hybrid BM25 + vector** | Lexical (BM25/ELSER-style sparse) fused with dense via RRF or weighted fusion, often + cross-encoder rerank | Product codes, IDs, exact names *and* semantic intent ([Elastic RRF retriever](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/retrievers/rrf-retriever)) |
| **Agentic RAG** | LLM plans, calls retrieval (and other tools) iteratively, evaluates sufficiency, may rewrite/decompose | Multi-hop, multi-source, “compare X and Y” ([Azure Architecture Center — Agentic RAG](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-agentic)) |

### How truthfulness, freshness, and permissions work

- **Truthfulness / groundedness:** RAG reduces closed-book hallucination by conditioning generation on retrieved passages and enabling provenance (cite chunk/doc IDs). It does **not** eliminate hallucination: models can still misread, over-generalize, or invent when retrieval is weak or contradictory. Production practice adds citation requirements, groundedness/faithfulness evals (e.g., RAGAS-style metrics), and sometimes answer/refusal guardrails ([datadeep.tech overview, 2026](https://datadeep.tech/retrieval-augmented-generation/); NIST GenAI Profile risk framing: [NIST.AI.600-1](https://doi.org/10.6028/NIST.AI.600-1)).
- **Freshness:** The non-parametric index is updatable independently of model weights (Lewis et al. demonstrated “index hot-swap”). Freshness depends on connector/change-feed latency, re-chunk/re-embed cost, and deletion/tombstone handling—not on LLM retraining. Incremental sync keeps lag in minutes–hours for hot corpora; stale chunks without version/`last_updated` metadata silently dominate answers.
- **Permissions (when bolted onto classical RAG):** Classical demos often index a shared corpus with **no** identity. Enterprise deployments add ACL metadata on chunks and apply **query-time pre-filters** (preferred) so unauthorized vectors never enter the candidate set. Index-time ACL tags alone go stale; live IdP group resolution + ACL sync + audit logs are required for revocation. Post-retrieval filtering is widely described as unsafe (leak into app memory / empty top-*k* / accidental bypass). See §2 for the full permission mechanics that enterprise search products productize ([permission-aware retrieval explainers](https://quelvio.com/blog/permission-aware-retrieval); [Seypro auth inheritance](https://sey.pro/insights/rag-auth-inheritance); [Simplico retrieval-layer ACL](https://www.simplico.net/2026/07/18/rag-retrieval-layer-access-control/)).

### Strengths

- Grounds answers in enterprise corpora without fine-tuning the base model for every knowledge update.
- Provenance: retrieved passages support citations and human verification.
- Separates knowledge updates (re-index) from model upgrades.
- Hybrid + rerank pipelines measurably improve retrieval vs dense-only on mixed enterprise queries (exact tokens + semantics).
- Mature tooling ecosystem (embedding APIs, vector features in Elastic/Postgres/MongoDB/Azure AI Search, orchestration frameworks).
- Scales from FAQ/copilot to domain Q&A; agentic wrappers reuse the same retrieval substrate for harder tasks.

### Failure modes / weaknesses

- **Retrieval failure → confident wrong answer:** Wrong/incomplete/outdated chunks; “lost in the middle”; conflicting sources.
- **Chunking loss:** Tables, policies, and cross-page dependencies break across chunk boundaries.
- **Embedding brittleness:** Exact IDs, SKUs, error codes, and rare proper nouns often favor BM25; pure vector search under-retrieves.
- **Permission gaps:** Shared indexes without query-time ACL pre-filters become privilege-escalation surfaces.
- **Evaluation debt:** Spot-checking demos overstates production quality; need continuous groundedness, recall@k, and permission-boundary tests.
- **Agentic cost/latency:** Multi-step loops (often ~8–15s vs ~2–3s single-pass) increase tokens, failure modes (tool thrash), and attack surface ([Azure agentic RAG considerations](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-agentic)).
- **Indirect prompt injection / poisoned KB:** Retrieved content can carry instructions; NIST GenAI Profile highlights retrieval-layer threats ([NIST.AI.600-1](https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.600-1.pdf)).

### Typical stack (components, patterns)

1. **Connectors / ingest:** SharePoint, Confluence, Drive, Slack, S3, ticketing, CRM; parse PDFs/HTML; PII tagging optional.
2. **Chunking & enrichment:** Fixed, recursive, or structure-aware chunks; metadata (source, path, owner, ACL principals, timestamps, classification).
3. **Embeddings + lexical index:** Dense model + BM25/sparse; optional GraphRAG layer for global questions (contrast only).
4. **Store:** Dedicated vector DB *or* incumbent DB/search with vectors (pgvector, MongoDB Atlas, Elasticsearch/OpenSearch, Azure AI Search, etc.). Standalone vector DBs face commoditization pressure from incumbents ([2026 market note](https://datadeep.tech/retrieval-augmented-generation/)).
5. **Retrieve:** Hybrid search → optional cross-encoder rerank → metadata/ACL filters.
6. **Generate:** Prompt assembly, citations, refusal when evidence insufficient.
7. **Orchestration:** Single-pass vs agentic (LangGraph / LlamaIndex / Azure Agent Framework / MCP tool surface).
8. **Eval & ops:** Offline harness + online telemetry; ACL sync jobs; audit of identity + chunk IDs returned.

### Fit for AI agents vs humans

| Consumer | Fit of classical/vector RAG |
| --- | --- |
| **Humans (chat/copilot)** | Strong for Q&A, policy lookup, “find and explain with links.” UX expects citations, low latency, and “I don’t know” when empty. |
| **AI agents** | Strong as a **tool**: agent calls search/retrieve with filters; agentic RAG when multi-hop needed. Agents amplify permission and injection risks (many tool calls, broader context windows). MCP and provider file-search APIs increasingly standardize the tool surface (2025–2026). |

### Notable vendors / patterns (2024–2026; caveats)

| Offering / pattern | Role | Caveat |
| --- | --- | --- |
| **Azure AI Search** (+ Foundry / knowledge sources) | Hybrid retrieval, integrated vectorization, document-level ACL patterns, agentic retrieval paths | Native ACL/Purview/SharePoint permission features often **preview**; sync lag until re-index ([Document-level access control](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)) |
| **Elasticsearch / Elastic** | BM25 + kNN + RRF; document-level security (DLS); connectors with ACL sync on some sources | DLS role design and connector coverage vary; Platinum+ for some DLS features ([DLS guide](https://www.elastic.co/guide/en/elasticsearch/reference/8.19/document-level-security.html); [DLS knowledge search blog](https://www.elastic.co/search-labs/blog/dls-internal-knowledge-search)) |
| **Pinecone, Weaviate, Qdrant, Chroma, etc.** | Purpose-built vector / hybrid stores | Still need ACL metadata design + IdP sync; not a full enterprise permission fabric |
| **pgvector / MongoDB Atlas Vector / OpenSearch** | Vectors inside existing data platforms | Good when ops prefer one platform; relevance/ACL maturity varies |
| **LangChain / LlamaIndex / LangGraph / Microsoft Agent Framework** | Orchestration, agentic loops | Framework ≠ production governance |
| **OpenAI File Search / Anthropic Citations API** | Managed retrieval + citation UX | Corpus often upload-scoped; enterprise ACL across SaaS silos is limited vs full enterprise search |
| **Microsoft GraphRAG / LazyGraphRAG** | Graph + vector for global/local questions | Research/maintenance-mode nuances; indexing cost historically high for full GraphRAG; LazyGraphRAG defers LLM cost ([MSR blog](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/); repo notes maintenance mode: [GitHub](https://github.com/microsoft/GraphRAG)) |
| **Vectara, custom “RAG-as-a-service”** | Managed ground+retrieve APIs | Buyer must still map identity and data residency |

Category framing (turnkey assistants vs build-on search vs hyperscaler suites): [CIOPages Enterprise Search & RAG buyer guide](https://www.ciopages.com/buyer-guides/enterprise-search-rag).

### Sources / links (selected)

- Lewis et al., *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*, NeurIPS 2020 — https://arxiv.org/abs/2005.11401  
- Azure Architecture Center, *Develop an Agentic RAG Solution on Azure* — https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-agentic  
- Elastic RRF retriever — https://www.elastic.co/docs/reference/elasticsearch/rest-apis/retrievers/rrf-retriever  
- Microsoft Research LazyGraphRAG — https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/  
- GraphRAG docs — https://microsoft.github.io/graphrag/  
- NIST AI RMF Generative AI Profile (NIST.AI.600-1) — https://doi.org/10.6028/NIST.AI.600-1 / https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.600-1.pdf  
- Enterprise RAG landscape notes (2026) — https://datadeep.tech/retrieval-augmented-generation/ ; https://www.sphereinc.com/guides/enterprise-rag-guide  

---

## 2) Enterprise Search + ACL-Aware Retrieval (Permissioned Search as SOT Layer for Humans and Agents)

### What it is

**Enterprise search** is a unified retrieval layer over heterogeneous workplace systems (files, wiki, chat, tickets, CRM, code, email metadata, etc.). Modern products combine lexical + neural ranking, connectors, an identity/people graph, and generative answering. The distinctive enterprise property is **ACL-aware (permissioned) retrieval**: every result and every passage used for generation is constrained to what the **requesting principal** is allowed to see in the source systems of record.

In the 2025–2026 framing, this layer is increasingly treated as a **shared source-of-truth (SOT) retrieval fabric** for both interactive human search/assistants and **AI agents**—same index, same permission enforcement, different clients (UI vs tool/API/MCP). Turnkey “work AI” platforms (e.g., Glean), hyperscaler suites (Microsoft 365 Copilot / Azure AI Search knowledge bases, Google Gemini Enterprise / Vertex AI Search, Amazon Q Business), and build-on engines (Elastic, Coveo, Sinequa, OpenSearch) all compete in this space ([CIOPages buyer guide](https://www.ciopages.com/buyer-guides/enterprise-search-rag); [Glean enterprise search](https://www.glean.com/enterprise-search); [Glean connectors](https://www.glean.com/platform/connectors)).

Technically, ACL-aware retrieval is **not** “ask the LLM to respect permissions.” It is architectural: identity-propagating queries, ACL metadata (or live authz checks) applied **inside** retrieval (pre-filter), and audit of what was eligible to return.

### How truthfulness, freshness, and permissions work

**Truthfulness**

- Same grounding mechanics as RAG: answers should be synthesized from permission-trimmed hits with citations back to source URLs/IDs.
- Enterprise search adds **authority / ranking signals** (recency, popularity, verified owners, pinned docs) that influence which evidence is seen as “canonical,” which can improve or bias truthfulness depending on governance.
- Residual risks: stale “official” docs ranked above newer drafts; synthesis across partially visible docs (user sees only their slice → incomplete but authorized truth).

**Freshness**

- Dual pipelines: **content sync** (create/update/delete) and **permission sync** (ACL/group membership changes).
- Hot connectors aim for near-real-time content and permission updates; inherited folder/site ACLs may require deeper refresh than item-unique ACLs (documented explicitly for Azure AI Search SharePoint ACL sync: item unique permissions can sync incrementally; parent-inherited changes may need explicit refresh — [Azure document-level access overview](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)).
- Vendor marketing (“real-time”) should be validated per connector; lag is a first-class risk for offboarding and “need-to-know” revocation.

**Permissions — precise mechanics**

| Mechanism | What happens | Notes |
| --- | --- | --- |
| **Index-time ACL capture** | At ingest, store allowed principals/groups (or ACL hash) on each doc/chunk, inherited from source | Necessary metadata; alone goes **stale** when access is revoked |
| **Query-time ACL pre-filter** | Resolve caller identity (token → user + groups via IdP); constrain ANN/BM25 to authorized subset **before** ranking top-*k* | Industry-preferred; avoids empty post-filter top-*k* and leakage into app memory ([Seypro](https://sey.pro/insights/rag-auth-inheritance); [Mohith G](https://mohithg.com/writing/rag-with-permissions.html)) |
| **Identity-propagating retrieval** | End-user (or on-behalf-of agent) token flows to search API (`x-ms-query-source-authorization` pattern in Azure AI Search; equivalent user-context APIs elsewhere) | Service identity alone must **not** return ACL-protected content ([Azure ACL behavior notes](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)) |
| **Live permission service** | Optionally re-check authz service per doc ID (always fresh, higher latency) | Complements or replaces cached ACL metadata |
| **Post-filter (anti-pattern for security)** | Retrieve globally then drop unauthorized hits | Leakage + relevance distortion; discouraged in security write-ups ([Simplico](https://www.simplico.net/2026/07/18/rag-retrieval-layer-access-control/)) |
| **Tenant / index isolation** | Separate indexes or namespaces per tenant/BU | Defense-in-depth for multi-tenant SaaS |
| **DB-enforced DLS / RLS** | Engine applies role query or row security on every query path | Elastic DLS ([docs](https://www.elastic.co/guide/en/elasticsearch/reference/8.19/document-level-security.html)); Postgres RLS patterns for vector tables |

**Index-time vs query-time (both required):** Index-time answers “what does this chunk require?” Query-time answers “what does this principal currently have?” ([Seypro](https://sey.pro/insights/rag-auth-inheritance)).

Azure AI Search documents four approaches: security string filters (GA), POSIX-like ACL/RBAC scopes (preview), Purview sensitivity labels (preview), SharePoint M365 ACLs (preview)—all evaluated at query time against synchronized metadata ([official overview](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)).

### Strengths

- **Single permissioned SOT** for humans and agents reduces “shadow indexes” that bypass ACLs.
- Reuses years of source-system ACL investment (Drive/SharePoint/Confluence/Slack channel permissions).
- Better default UX for knowledge work: personalized ranking, connectors, activity graphs (product-dependent).
- Auditability: who queried, which principals were resolved, which docs were eligible—supports compliance narratives (HIPAA minimum-necessary style reasoning appears in industry architecture notes; map carefully to legal counsel).
- Agents can call the same trimmed retrieve API, aligning tool use with human visibility (when identity is correctly propagated—not a service superuser).

### Failure modes / weaknesses

- **ACL sync lag / inheritance bugs:** Offboarded users or revoked folders remain searchable until sync catches up.
- **Oversharing in sources:** Search faithfully exposes bad source ACLs at scale (“search makes oversharing discoverable”).
- **Partial visibility synthesis:** Authorized-but-incomplete evidence can yield misleading answers without revealing that other evidence exists (no metadata leakage about denied hits is intentional—and operationally painful).
- **Connector coverage gaps:** Custom/legacy systems need Indexing SDKs; uneven depth (title-only vs full text vs permissions fidelity).
- **Group explosion / nested groups:** Large IdP group graphs make filter cardinality and cache invalidation hard.
- **Agent identity pitfalls:** Agents running under privileged app identities bypass user ACLs unless on-behalf-of / OBO or per-user tokens are mandatory.
- **Relevance vs security tension:** Aggressive personalization can surface “popular but wrong” docs; security filters shrink recall for highly restricted users.
- **Vendor lock-in / residency:** SaaS indexes concentrate sensitive content; data residency and subprocessors matter.
- **False confidence in “permissions-aware” labels:** Must verify pre-filter vs post-filter, sync SLAs, and elevated-read admin paths.

### Typical stack (components, patterns)

1. **Identity:** Entra ID / Okta / Google Workspace as IdP; group membership resolution at query time.
2. **Connectors:** Bidirectional understanding of content + ACL model per app; webhook/change tokens.
3. **Unified index:** Lexical + vector + metadata; optional enterprise knowledge/people graph.
4. **Permission store:** Normalized principals on documents; ACL filter indexes; optional sidecar authz.
5. **Query plane:** Authenticated search API; hybrid retrieval; generative answer layer with citations; agent tool/MCP endpoint using **same** trimming.
6. **Admin / governance:** Content inclusion/exclusion, DLP/sensitivity labels, audit logs, retention.
7. **Eval:** Permission boundary test suites (user A must never retrieve doc B); freshness SLOs; relevance with ACL on.

### Fit for AI agents vs humans

| Consumer | Fit |
| --- | --- |
| **Humans** | Primary UX: search box, assistant, embedded widgets. Expects “only what I can open,” deep links into source apps, personalization. |
| **AI agents** | Natural **SOT retrieval tool**: one permissioned retrieve/search action instead of N brittle per-SaaS integrations. Requires user-delegated auth, tool allowlists, step-up approval for write actions, and logging. Platforms marketing “agents + connectors” (e.g., Glean Agents with permissioned actions — [connectors/platform](https://www.glean.com/platform/connectors); [docs on connectors & permissions](https://docs.glean.com/connectors/about.md)) position this explicitly. Hyperscalers wire agents to knowledge bases with label/ACL metadata in retrieve responses (Azure knowledge sources / Purview labels — [Azure doc](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)). |

Classical RAG without this layer is often a **project-specific index**; ACL-aware enterprise search aims to be the **shared retrieval substrate** agents and humans both call.

### Notable vendors / patterns (2024–2026; caveats)

| Vendor / product | Pattern | Caveat |
| --- | --- | --- |
| **Glean** | Turnkey work AI: 100s of connectors, inherited permissions, assistant + agents, enterprise graph | Proprietary; validate per-connector permission freshness and residency; marketing claims need pilot evidence ([enterprise search](https://www.glean.com/enterprise-search); [connectors](https://www.glean.com/platform/connectors)) |
| **Microsoft 365 Copilot + Graph / Azure AI Search / Foundry IQ** | Hyperscaler: Graph permissions + search indexes with document-level ACL/Purview/SharePoint patterns; agentic retrieval | Feature maturity mixed (many ACL paths **preview**); M365-centric strength vs heterogeneous non-Microsoft estates ([ACL overview](https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview)) |
| **Google Gemini Enterprise / Vertex AI Search** | Hyperscaler search + gen answering over Google and connected corpora | Confirm ACL fidelity outside Google Workspace |
| **Amazon Q Business** | AWS-centric enterprise assistant with indexed connectors and access control integration | Depth varies by connector; IAM vs document ACL mapping must be validated |
| **Elastic / OpenSearch** | Build-on hybrid search + DLS + some connector ACL sync | You own relevance tuning, connector ops, and ACL role design ([DLS](https://www.elastic.co/guide/en/elasticsearch/reference/8.19/document-level-security.html)) |
| **Coveo, Sinequa (ChapsVision), Lucidworks, Mindbreeze** | Enterprise/relevance platforms with GenAI grounding | Strong in specific verticals (e.g., Coveo commerce/service heritage); compare ACL models explicitly |
| **Moveworks, Kore.ai, etc.** | Work assistants / automation with search components | Often workflow-first; treat knowledge/ACL claims as product-specific |

Buyer taxonomy useful for RFP structure: turnkey assistant vs hyperscaler suite vs build-on engine vs RAG API — [CIOPages](https://www.ciopages.com/buyer-guides/enterprise-search-rag).

### Sources / links (selected)

- Azure AI Search — Document-level access control — https://learn.microsoft.com/en-us/azure/search/search-document-level-access-overview  
- Elastic document-level security — https://www.elastic.co/guide/en/elasticsearch/reference/8.19/document-level-security.html  
- Elastic Labs — DLS for knowledge search — https://www.elastic.co/search-labs/blog/dls-internal-knowledge-search  
- Glean — Enterprise search / connectors / connector docs — https://www.glean.com/enterprise-search ; https://www.glean.com/platform/connectors ; https://docs.glean.com/connectors/about.md  
- Permission-aware retrieval explainers — https://quelvio.com/blog/permission-aware-retrieval ; https://sey.pro/insights/rag-auth-inheritance ; https://mohithg.com/writing/rag-with-permissions.html ; https://www.simplico.net/2026/07/18/rag-retrieval-layer-access-control/  
- NIST.AI.600-1 GenAI Profile — https://doi.org/10.6028/NIST.AI.600-1  
- CIOPages — Enterprise Search & RAG platforms buyer guide — https://www.ciopages.com/buyer-guides/enterprise-search-rag  

---

## Cross-cutting contrast (descriptive only)

| Dimension | Classical / vector RAG | Enterprise search + ACL-aware retrieval |
| --- | --- | --- |
| Primary artifact | App- or domain-specific chunk index + generator | Unified, connector-fed, identity-aware search fabric |
| Permission model | Often optional / custom metadata filters | Core product: inherit + enforce source ACLs |
| Freshness focus | Re-embed/re-index pipelines | Content **and** ACL sync SLOs |
| Human UX | Chat over a corpus | Search + assistant across the work graph |
| Agent UX | Retrieve tool over project index | Shared permissioned retrieve/search tool (SOT) |
| Typical failure | Wrong chunk / hallucination | ACL lag, oversharing amplification, OBO mistakes |

*End of research section. No recommendations included.*

---

# Part B — Knowledge Graphs & GraphRAG

## 1) Knowledge Graphs (Enterprise KG as Source of Truth / Semantic Layer)

### What it is

An **enterprise knowledge graph (EKG)** is a graph of data intended to accumulate and convey knowledge of the real world, whose nodes represent entities of interest and whose edges represent typed relations between those entities ([Hogan et al., ACM Computing Surveys, 2021](https://dl.acm.org/doi/10.1145/3447772); PDF overview: [penni.wu.ac.at survey copy](https://penni.wu.ac.at/papers/ACMsurveys%20Knowledge%20Graphs.pdf)). In enterprise settings, the KG is typically **internal**, governed, and applied to commercial use cases (search, recommendations, risk, automation, analytics). Nodes carry identity and attributes; edges carry typed, often directional relationships with optional properties (effective dates, confidence, provenance).

As a **semantic layer / source-of-truth pattern**, the EKG sits above (or alongside) warehouses, lakes, and operational systems. It does not replace those stores; it defines a shared business vocabulary—entity types, properties, relationships, constraints—and binds those concepts to concrete data so humans and machines use the same meaning. Microsoft Fabric’s Ontology (preview) frames this explicitly: entity types, properties, relationships, data bindings to OneLake, and a queryable ontology graph with lineage and scheduled refresh ([Microsoft Learn: Fabric Ontology overview](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)). Analysts often distinguish a **metrics/BI semantic layer** (governed metrics → SQL/compilation) from a **knowledge graph** (entity–relationship reasoning); many 2025–2026 architectures treat them as complementary parts of a “meaning stack” rather than substitutes ([Semantic Layer vs. Knowledge Graph, 2026](https://colrows.com/blogs/semantic-layer-vs-knowledge-graph/); Forrester on semantics/ontologies for agentic AI: [Forrester blog](https://www.forrester.com/blogs/build-meaning-before-machines-why-semantics-ontologies-and-knowledge-graphs-matter-for-agentic-ai/)).

**Ontology/governance burden vs. RAG’s weaker schema:** A curated EKG requires explicit ontology design, entity resolution, stewards, change control, and policy attachment. Classic vector RAG typically uses weak/implicit schema (chunks + embeddings), trading governance and consistency for speed of ingestion. The EKG’s cost is upfront modeling and ongoing stewardship; its payoff is shared definitions, multi-hop joins as first-class edges, and auditable meaning.

### How truthfulness, freshness, and permissions work

| Concern | Typical EKG pattern |
| --- | --- |
| **Truthfulness** | Asserted facts come from bound source systems and curated mappings; provenance/lineage retained on nodes/edges. Inference (rules, graph algorithms) is explicit and separable from asserted facts. Truth is “what the governed model says,” not “what the LLM inferred from text.” |
| **Freshness** | Driven by ETL/ELT or streaming bindings into the graph; Fabric Ontology notes upstream updates require graph refresh before visibility ([Learn docs](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)). Diffbot-style web KGs rebuild on crawl cycles (e.g., multi-day rebuilds) ([Diffbot approach](https://blog.diffbot.com/diffbots-approach-to-knowledge-graph/)). Staleness is a pipeline/ops problem, not a retrieval-rank problem. |
| **Permissions** | Row/column/object ACLs from IdP + catalog (e.g., Purview classifications, OneLake security, graph-store RBAC). Ideal pattern: enforce access at query time on graph elements or via federated queries that inherit source policies—not post-filter LLM context alone. Purview commonly supplies catalog, lineage, sensitivity labels; Fabric IQ ontology and Purview Unified Catalog integration remain an evolving governance story ([Fabric IQ / Purview discussion](https://medium.com/@marcoOesterlin/why-fabric-iq-ontologies-needs-to-exist-in-microsoft-purview-unified-catalog-b756ad8210f6); finance EKG + Fabric/Purview narrative: [Nathan Lasnoski, 2026](https://nathanlasnoski.com/2026/03/01/building-an-enterprise-knowledge-graph-with-microsoft-foundry-a-finance-use-case-guide/)). |

### Strengths

- Explicit, typed relationships enable multi-hop reasoning (fraud rings, supply-chain blast radius, org/process digital twins) without burying joins in ad hoc SQL.
- Shared enterprise vocabulary reduces metric/entity drift across BI, apps, and agents.
- Provenance and constraints support auditability and “glass box” grounding for AI.
- Complements warehouses: structural/context layer rather than a full data-platform replacement ([enterprise KG architecture overviews](https://www.motadata.com/blog/enterprise-knowledge-graph)).
- Strong fit for Customer 360, compliance lineage, master-data-style entity hubs, and agent grounding on organizational facts.

### Failure modes / weaknesses

- **Ontology and stewardship burden:** Modeling committees, conflicting domain ontologies, slow schema evolution.
- **Entity resolution debt:** Duplicates, brittle string matching, incomplete linking across systems.
- **Coverage gaps:** Graph only knows what was modeled and loaded; silent incompleteness can be worse than RAG’s probabilistic recall.
- **Freshness lag** if bindings are batch-only; operational graphs need CDC/streaming discipline.
- **Over-claiming “source of truth”:** Without governance and quality SLAs, the KG becomes another silo.
- **Query/skill gap:** Cypher/Gremlin/SPARQL and graph ops talent; NL2Graph/NL2Ontology layers help but introduce their own error modes.
- Not a substitute for governed metric calculation; relationship graphs ≠ deterministic KPI compilation ([semantic layer vs KG](https://colrows.com/blogs/semantic-layer-vs-knowledge-graph/)).

### Typical stack (components, patterns)

1. **Ontology / semantic model** — OWL/SHACL or vendor ontology items; business glossary alignment.
2. **Graph store** — Property graph (Neo4j, Amazon Neptune) or RDF triple store; sometimes Fabric Graph / ontology graph.
3. **Identity & mastering** — Entity resolution, golden records, crosswalk tables.
4. **Ingestion/bindings** — Pipelines from ERP/CRM/lakehouse; CDC; event streams.
5. **Governance** — Catalog (Purview/Collibra/Atlan), classification, lineage, policy.
6. **Query & serving** — Cypher/openCypher/Gremlin/SPARQL; GraphQL; NL2Ontology; MCP/API exposure to agents ([Neo4j Knowledge Layer](https://neo4j.com/product/knowledge-layer/)).
7. **Optional AI overlay** — GraphRAG or vector search *over* the curated graph (distinct from LLM-built GraphRAG corpora).

### Fit for AI agents vs humans

| Audience | Fit |
| --- | --- |
| **Humans** | Exploration UIs, lineage, impact analysis, governed BI join paths, domain browsing by concept rather than table. |
| **AI agents** | Tool-callable structured context: traverse typed edges, respect policies, cite entity IDs; reduces hallucinated relationships when the graph is authoritative. Agents still need retrieval for unstructured narrative; EKG shines for “how things connect” and “what is the official definition.” |

### Notable vendors / patterns (2024–2026)

- **Neo4j** — Graph database + “knowledge layer” positioning (semantic models, GraphRAG, agent memory, governance) ([neo4j.com/product/knowledge-layer](https://neo4j.com/product/knowledge-layer/)).
- **Amazon Neptune** — Managed graph (Database + Analytics); property-graph and RDF; common backend for LlamaIndex PropertyGraph and Bedrock GraphRAG ([LlamaIndex Neptune example](https://developers.llamaindex.ai/python/examples/property_graph/property_graph_neptune/)).
- **Microsoft Fabric IQ / Ontology (preview)** — Enterprise vocabulary, bindings to OneLake, ontology graph, NL2Ontology ([Learn](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview)); Purview for catalog/lineage/AI DSPM angles ([HSO FabCon 2026 notes](https://www.hso.com/blog/what-we-learned-at-fabcon-and-sqlcon-2026/)).
- **Diffbot Knowledge Graph** — Automatically constructed, web-scale public KG (orgs, people, articles); Search (DQL) + Enhance APIs ([product](https://www.diffbot.com/products/knowledge-graph); [DQL docs](https://docs.diffbot.com/docs/kg-search)).
- **Catalog-centric “enterprise data graphs”** — Metadata/lineage graphs (e.g., Atlan-style) as governed context for agents, distinct from domain EKGs ([Atlan KG vs graph DB](https://atlan.com/know/ai-agent/knowledge-graph/knowledge-graph-vs-graph-database/)).

### Sources / links

- Hogan et al., “Knowledge Graphs,” *ACM Computing Surveys* (2021): https://dl.acm.org/doi/10.1145/3447772  
- Microsoft Fabric Ontology (preview): https://learn.microsoft.com/en-us/fabric/iq/ontology/overview  
- Neo4j Knowledge Layer: https://neo4j.com/product/knowledge-layer/  
- Diffbot Knowledge Graph: https://www.diffbot.com/products/knowledge-graph  
- Forrester — semantics, ontologies, KGs for agentic AI: https://www.forrester.com/blogs/build-meaning-before-machines-why-semantics-ontologies-and-knowledge-graphs-matter-for-agentic-ai/  
- Semantic layer vs knowledge graph (2026): https://colrows.com/blogs/semantic-layer-vs-knowledge-graph/

---

## 2) GraphRAG and Related Graph + Retrieval Hybrids

### What it is

**GraphRAG** (Microsoft Research) is a structured, hierarchical RAG approach: an LLM extracts entities, relationships, and claims from unstructured text into a graph index; community detection (e.g., Leiden) builds a hierarchy; community summaries are pre-generated; query time uses those structures plus optional vector search ([docs](https://microsoft.github.io/graphrag/); [arXiv:2404.16130 — *From Local to Global*](https://arxiv.org/abs/2404.16130); [GitHub: microsoft/graphrag](https://github.com/microsoft/graphrag); [MSR blog, July 2024](https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery-now-on-github/)).

Primary query modes in the open GraphRAG system include **Global Search** (map-reduce over community summaries for whole-corpus themes), **Local Search** (entity neighborhood fan-out), **DRIFT Search** (local + community context), and **Basic Search** (baseline vector RAG) ([GraphRAG docs](https://microsoft.github.io/graphrag/)).

**How GraphRAG differs from a curated enterprise KG:** GraphRAG’s graph is typically an **LLM-derived index over a document corpus**, optimized for retrieval and sensemaking—not a stewarded enterprise ontology bound to systems of record. Entity quality depends on extraction prompts; schema is soft/ emergent; “truth” is grounded in source text chunks, not master data. A curated EKG is the opposite: heavy ontology/governance, authoritative identities, weaker native fit for free-text theme summarization unless documents are separately indexed. Hybrids exist (store GraphRAG output in Neo4j/Neptune; overlay GraphRAG on curated graphs), but the patterns should not be conflated.

Related **graph + retrieval hybrids (2024–2026):**

- **LazyGraphRAG** — Defers LLM summarization; NLP co-occurrence graphs + iterative deepening at query time; indexing cost ≈ vector RAG (~0.1% of full GraphRAG); strong local/global trade-offs ([MSR blog, Nov 2024](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/); integrated into Microsoft Discovery / Azure Local per June 2025 editor note).
- **LlamaIndex PropertyGraphIndex** — Modular extractors (`SchemaLLMPathExtractor`, etc.) + retrievers (vector context, synonym, TextToCypher) over Neo4j/Neptune/others ([intro](https://www.llamaindex.ai/blog/introducing-the-property-graph-index-a-powerful-new-way-to-build-knowledge-graphs-with-llms); [docs](https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/)).
- **Amazon Bedrock Knowledge Bases GraphRAG** — Managed entity/relationship extraction into **Neptune Analytics** alongside embeddings; vector + graph traversal without custom graph modeling ([GA announcement, Mar 2025](https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-bedrock-knowledge-bases-graphrag-generally-available/); [build guide](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-build-graphs-build.html)).
- **Lightweight OSS variants** — e.g., Fast GraphRAG (PageRank-style exploration, lower indexing cost) ([circlemind-ai/fast-graphrag](https://github.com/circlemind-ai/fast-graphrag)).

**Ontology/governance vs RAG schema:** GraphRAG inherits RAG’s weak schema—optional entity-type hints and prompt tuning—not enterprise ontology governance. SchemaLLM extractors can *impose* a partial schema, but maintenance, ACL semantics, and golden-record identity are usually thinner than EKG programs unless deliberately engineered.

### How truthfulness, freshness, and permissions work

| Concern | Typical GraphRAG / hybrid pattern |
| --- | --- |
| **Truthfulness** | Grounding is **source-chunk citations** and extracted claims; community summaries are abstractive (second-order LLM text) and can drift or omit nuance. Better than ungrounded LLM recall for global themes; weaker than curated KG assertions for authoritative facts (prices, entitlements, legal status). Evaluation often uses LLM-as-judge (comprehensiveness, diversity) ([arXiv paper](https://arxiv.org/abs/2404.16130)). |
| **Freshness** | Full GraphRAG: expensive re-index / incremental extraction when corpus changes; community summaries go stale until rebuild. LazyGraphRAG and vector-first hybrids favor cheaper re-embedding + light NLP graphs—better for streaming/exploratory corpora. Managed Bedrock GraphRAG updates graphs from S3 ingestion pipelines. |
| **Permissions** | Usually document-/chunk-level ACLs applied at retrieval (filter before graph expansion). Graph edges can leak cross-document association if ACLs are only enforced on final chunks. Enterprise deployments must propagate labels through entity linking and community membership—non-trivial and often under-specified in research code. Open-source GraphRAG is a research/maintenance-mode project, not a full IAM product ([repo notice](https://github.com/microsoft/graphrag)). |

### Strengths

- Excels at **global / sensemaking** questions (“main themes,” “recurring failure patterns”) where top-k vector RAG fails ([MSR GitHub release blog](https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery-now-on-github/)).
- Multi-hop “connect the dots” via entity neighborhoods (local/DRIFT search).
- Hierarchical community summaries provide precomputed corpus maps.
- Hybrid stacks combine semantic similarity with structural traversal (Bedrock; LlamaIndex multi-retriever).
- Cost-quality knobs emerging (LazyGraphRAG relevance-test budget; dynamic community selection).

### Failure modes / weaknesses

- **Indexing cost & complexity** of full LLM extraction + community summarization; historically prohibitive at large scale without Lazy/fast variants.
- **Wrong-tool regression:** Using GraphRAG for simple factoid lookups can worsen latency/quality vs vector RAG ([practitioner commentary, 2026](https://medium.com/@pankaj_pandey/microsoft-graphrag-a-breakthrough-for-global-questions-a-downgrade-for-everything-else-22b294bb3292)).
- **Extraction errors:** Missed entities, duplicate nodes, prompt brittleness; soft entity matching.
- **Summary hallucination / lossy compression** in community reports; empowerment metrics mixed vs citing raw quotes ([paper discussion](https://arxiv.org/abs/2404.16130)).
- **Permission leakage risk** via shared entities/communities across ACL boundaries.
- **Not an enterprise SoT:** Should not replace curated master data or metric semantic layers.
- Open Microsoft GraphRAG repo largely in **maintenance mode** as of 2025–2026; production often means Azure/Discovery managed paths, Neo4j/LlamaIndex/Bedrock reimplementations, or LazyGraphRAG-class designs ([GitHub](https://github.com/microsoft/graphrag); [Project GraphRAG](https://www.microsoft.com/en-us/research/project/graphrag/)).

### Typical stack (components, patterns)

1. **Corpus store** — Documents in object storage / lake.
2. **Chunking + embeddings** — Vector index (baseline path).
3. **Graph construction** — LLM entity/relation/claim extraction *or* NLP co-occurrence (Lazy); optional schema-guided extractors.
4. **Graph + vector store** — Neo4j, Neptune Analytics, Fabric Graph, or research parquet/LanceDB-style indexes.
5. **Community detection & summaries** — Leiden + LLM reports (full GraphRAG) or deferred (Lazy).
6. **Query router** — Global vs local vs vector vs hybrid; agent tool selection.
7. **Generation + citation** — Map-reduce or claim extraction into final answer; eval harnesses (e.g., BenchmarkQED on Project GraphRAG timeline).

**Pattern sketch:**  
`Documents → chunks → (vectors ⊕ entity graph) → [communities/summaries] → retriever(s) → LLM answer with source refs`

### Fit for AI agents vs humans

| Audience | Fit |
| --- | --- |
| **Humans** | Analyst sensemaking over large narrative corpora (investigations, research literature, support ticket themes); browsing community reports as “table of contents.” |
| **AI agents** | Strong as a **retrieval tool** for thematic and multi-hop document questions; weaker as sole world model for transactional truth. Agents benefit from routing: vector for local facts, GraphRAG/Lazy for global synthesis, curated EKG for authoritative entities/policies. |

### Notable vendors / patterns (2024–2026)

| Pattern / vendor | Notes | Link |
| --- | --- | --- |
| Microsoft GraphRAG | Paper + OSS pipeline; community summaries; local/global/DRIFT | https://github.com/microsoft/graphrag · https://arxiv.org/abs/2404.16130 |
| LazyGraphRAG | Low indexing cost; query-time LLM; Discovery / Azure Local | https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/ |
| Neo4j GraphRAG / Knowledge Layer | Production graph + hybrid retrieval | https://neo4j.com/product/knowledge-layer/ |
| LlamaIndex PropertyGraph | Extractors + Neo4j/Neptune stores | https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/ |
| Amazon Bedrock KB GraphRAG + Neptune Analytics | Managed GA (Mar 2025) | https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-bedrock-knowledge-bases-graphrag-generally-available/ |
| Fast GraphRAG | Cost-oriented OSS alternative | https://github.com/circlemind-ai/fast-graphrag |
| Diffbot | Web KG / extraction feeding custom GraphRAG-style apps | https://www.diffbot.com/products/knowledge-graph |

### Sources / links

- GraphRAG paper: https://arxiv.org/abs/2404.16130 · PDF: https://arxiv.org/pdf/2404.16130  
- GraphRAG docs: https://microsoft.github.io/graphrag/  
- GraphRAG GitHub: https://github.com/microsoft/graphrag  
- GraphRAG on GitHub announcement: https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery-now-on-github/  
- Project GraphRAG hub: https://www.microsoft.com/en-us/research/project/graphrag/  
- LazyGraphRAG: https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/  
- Bedrock Knowledge Bases GraphRAG GA: https://aws.amazon.com/about-aws/whats-new/2025/03/amazon-bedrock-knowledge-bases-graphrag-generally-available/  
- Bedrock graph build docs: https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base-build-graphs-build.html  
- LlamaIndex Property Graph guide: https://developers.llamaindex.ai/python/framework/module_guides/indexing/lpg_index_guide/  

---

## Cross-cutting clarification (for merge context)

| Dimension | Curated enterprise KG | GraphRAG-style hybrid |
| --- | --- | --- |
| Primary input | Systems of record + ontology | Unstructured corpora |
| Schema | Explicit, governed | Soft / LLM-extracted (optionally schema-guided) |
| Role | Semantic SoT / relationship layer | Retrieval & sensemaking index |
| Truth model | Asserted + provenance | Chunk-grounded generation |
| Best questions | “What is X officially related to Y?” | “What are the themes / multi-doc stories?” |
| Governance load | High (stewards, ACL, quality) | Lower schema load; ACL still hard |
| Common combo | EKG for entities/policies + GraphRAG/vector for documents | Same stack, different stores/tools |


---

# Part C — Catalogs, Structured SoTs & Hybrids

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
