# Research Pack: Enterprise Knowledge & Data Approaches

*Research only — no recommendations. Landscape as of 2026-09. Sources cited inline.*

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
