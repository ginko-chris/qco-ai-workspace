# Company Knowledge: Next-Steps Options Brief

**From:** Knowledge Graph Architect. **For:** Chris, via Chief of Staff. **Date:** 2026-09-29. **Status:** v0.5 (2026-09-29), an advisory draft. v0.5: the MCP server is built in-house, with a partner as fallback, and the Integrator seat is TBD (section 11). v0.5 also adds build task B-1, a hands-on week-1 test of grok.com custom MCP connectors against Entra ID (section 6.1). v0.5 also adds a latency budget (section 12). v0.4: v0.4 records three decisions: IT pays for the subscription, Strety is reached through its REST API with read-only scope, and Rocks is the pilot area (sections 9 and 10). v0.2 moved hosting from the MSP to Azure (confirmed by Chris). v0.3 adds hour estimates for the operator and curator roles (sections 7 and 8) and replaces the web front door with agent-native access (section 6).
**Scope:** This brief covers the Company KB lane only: certified tribal knowledge, roles and accountability, and how-we-work content. It takes Chris's Sep 28 rulings as fixed and does not revisit them:
- Fusion Manage owns engineering definition.
- NetSuite owns commercial identity.
- The MBOM owner is still in discovery, and Product Development is the likely owner.
- The product KG is ISA-95 in Stardog.
- The Company KB is separate from the product KG. It queries the product KG read-only, with citations, and stores pointers only (the C3 rule).

Related documents: memo v1.2, plan v2.1, field-authority table v0.3, two-lane MCP stub v0.2, steward role card v0.2.

## 1. Options

| | (a) Catalog + RAG + semantic layer | (b) Full company KG | (c) Hybrid (recommended) |
|---|---|---|---|
| **What it looks like at QCo** | A catalog of certified documents (SOPs, how-tos, traveler know-how, FAQs). Each document carries its owner, SME verifier, review-by date and access tags. Retrieval runs over only those documents. A small glossary gives the semantic layer (terms, synonyms, "ask X for Y"). Behind `qco-company` MCP tools. | People, seats, roles, processes, systems, sites and documents all modeled as a formal ontology. Every piece of content is tied to entities in a graph store. | Option (a) plus a thin typed index of about 5 entity types: Seat/Role, Process, System, Site/Area and Doc. Seats and Rock owners come live from Strety UUIDs and are not copied. Product questions go to `qco-product` through `ask_product`. |
| **Effort and skills** | Lowest effort. One trained associate curates, SMEs verify, and a developer or partner builds for a few weeks. It runs on standard Azure managed services, operated by Chris's team (section 6). | Highest effort. Needs an ontology engineer and ongoing modeling. This is the same skill gap already flagged for the product KG. It would compete for scarce 2–3 FTE or partner time. | Low to moderate effort. The same associate curates, plus a small, fixed schema with no ontology owner. Adds one index table and joins to Strety. |
| **Good at** | "How do we…" answers with citations. Fast to stand up. Easy to certify and expire content. | "Who is accountable for process X across sites" and multi-step relationship questions. | Answers with citations plus a reliable "who owns this" and "which system holds this". Pointers only. |
| **Bad at** | Relationship questions. It will guess "who owns" from text. | Slow to deliver value. The model outpaces the content. Thin IT can't sustain it. Risks becoming a second org chart and a shadow product master. | Needs discipline so the entity list doesn't grow into (b). Retrieval quality still depends on how good the documents are. |
| **Fit with the rulings** | Good fit. It stays separate and follows the pointers-only rule, but role and accountability modeling is weak. | Poor fit. It duplicates Strety and edges toward the product lane. | Best fit. Roles and accountability are modeled by reference to Strety. The product KG is queried rather than copied. The C1 ban list is enforced in the tools. |

## 2. Recommendation: option (c), the hybrid

The questions people will actually ask the Company KB are mostly "how do we do this" and "who do I ask". Certified documents with citations answer the first. A thin set of entity types answers the second without building a second org chart. A full KG only pays for itself when relationships are the product, as with BOMs, configuration and cross-document linking. That is exactly why it sits in the product lane. Putting a second formal graph on thin IT doubles the ontology burden for little gain. Option (c) also leaves a clean upgrade path: the small set of entity types can later map onto a formal model if a real need for relationship reasoning appears.

## 3. Decisions for Chris

1. **Pilot domain and SME verifier.** Choose one bounded area for the first 30–45 days. Candidates are order entry and customer service know-how, or shop-floor traveler know-how.
2. **KB curator.** Name the trained associate who curates the KB and set their weekly hours. This is separate from the crosswalk steward.
3. **Store and search setup (Azure is confirmed).** For the pilot, the recommendation is Azure Database for PostgreSQL with search built into the same database. Add Azure AI Search only if the pilot shows search quality needs it. In either case agents get read-only access, and every query is checked under the user's own Entra ID identity.
4. **Certification policy.** Set the default review-by period, and decide what happens when content expires: the agent refuses with `stale`, or answers with a warning.
5. **Strety access.** Strety's MCP can read and write. Decide between a read-only scoped token and nightly read-only snapshots. Either way, agents get no write path.

## 4. Top risks and mitigations

| Risk | Mitigation |
|---|---|
| Tribal content goes stale and becomes a confident wrong answer. | Every document carries a review-by date. The tools refuse with `stale` once it passes. Stale-document counts go on the Scorecard. |
| HR or finance content leaks in, including through live Strety citations. | The C1 ban list runs at ingestion and at response time, as hard rule 6 in the stub. Golden tests include adversarial prompts. |
| The KB copies product facts and becomes a shadow product master. | The C3 pointers-only rule. Product answers come only through `ask_product` with a citation. Ingestion rejects BOM, spec and price tables. |
| Thin IT can't run it. | Use Azure managed services only, so Azure handles patching, backups and uptime. There is no graph database in this lane. The operator role sits on Chris's team with set hours, and there is one Azure runbook. The MSP stays on network, laptops, printers and basic infrastructure, with no platform accountability. |
| People fear "AI takes jobs", or distrust wrong answers. | Frame it as help for current work. Show citations and the name of the SME who verified each answer. Report accuracy openly. |
| Curator or verifier hours disappear. | Name people with set hours before ingestion starts. The pilot reports measured hours per document. |

## 5. First milestone (30–45 days)

**Deliverable:** "Company KB Pilot v0". One domain behind a read-only `qco-company` MCP with a small set of tools: `search_kb`, `who_owns`, and `ask_product` passthrough.

**Exit criteria:**
- Certified documents for the chosen domain are loaded. Each has an owner, a verifier, a review-by date and access tags.
- A golden set of about 50 real questions, written by the SME, is answered with citations. The proposed target is at least 85% judged correct by the SME. This is a proposal to set, not a benchmark.
- Zero banned-content leaks on the adversarial test set.
- Refusals are correct, meaning `stale`, `uncertified`, `acl_deny` and `non_authoritative` fire when they should.
- Every `ask_product` answer cites the product lane and copies nothing. Until the product KG is live, product questions return `non_authoritative` or are routed to a person.
- Measured curator and verifier hours per document, to size the steady state.
- At least two of the three clients (Claude Cowork, Grok, Grok Bots) call `qco-company` under the person's own Entra ID identity. A test user outside a restricted group gets `acl_deny`.
- The Azure environment is live with private endpoints only, Entra ID sign-in and a Key Vault. One point-in-time restore test has been run. The operator runbook is signed off by the named operator on Chris's team.

**Not in scope:** HR or finance content, writes to any system of record, and relationship modeling beyond the five entity types.

## 6. Hosting on Azure (added in v0.2)

- **Components.** Azure Blob Storage holds the certified library, with versioning and soft delete turned on. Azure Database for PostgreSQL (flexible server) holds the five-entity index and the search index. An App Service hosts the `qco-company` remote MCP server, which is the only way in (see Access below). There is no standalone web UI. Key Vault holds secrets. Log Analytics holds the answer logs, which are read-only to everyone except auditors. Azure AI Search is optional and added later.
- **Identity.** Everyone signs in with Entra ID through the existing Microsoft 365 tenant. Access tags map to Entra groups. Agents act on behalf of the user and hold no standing account with broader rights.
- **Networking.** Data services have no public access and are reached only through private endpoints. The MCP server is the only public entry point. It accepts only calls that carry an Entra ID token. Where a client supports it, inbound traffic can also be limited to that client's published egress addresses.
- **Access (agent-native).** People reach the KB through the agents they already use: Claude Cowork, Grok and Grok Bots. "Claude Cowork" is our reading of "Claude Codework" and needs confirming. No web UI is the primary path. Copilot and ChatGPT are out of scope.
  - `qco-company` is a remote MCP server over HTTPS. It exposes only the fixed tools: `search_kb`, `who_owns`, `get_doc` (citation fetch) and `ask_product` (passthrough). It exposes no raw SQL, free-form query or write tools.
  - The same tools are available as a small REST API, for any agent that can call HTTP but not MCP.
  - **Identity.** A person connects the server once in their agent. That opens a Microsoft (Entra ID) sign-in using OAuth, and the agent then holds a token for that person. Every tool call runs as that person. Access tags are matched to their Entra groups.
  - **Agent permissions.** An agent never gets more than the person it acts for. Group-chat or background bots such as Grok Bots must act for a named person. Otherwise they are limited to an allowlisted "public-to-staff" slice. They never use a shared service account with broad rights.
  - **Logging.** Each call is logged with the person, the client (Claude, Grok or Grok Bot), the tool, what was retrieved and the answer or refusal.
  - **Answer generation.** The server returns cited passages and records. The calling agent's own model writes the answer, so Azure OpenAI isn't needed for answers. QCo's existing Claude and Grok subscriptions carry that cost. Embeddings for search are the only model cost in Azure.
  - **Must verify before build.** Each client's support for remote MCP connectors with OAuth sign-in, including admin controls for org-wide connectors, has to be confirmed on the current versions. This matters most for Grok Bots, which may need the REST fallback. The allowed clients are recorded in the Entra app registration.
  - **Grok status (see 6.1).** Claude Cowork's static client ID and secret path is confirmed. Grok on grok.com is in progress, with the test scheduled as build task B-1. Grok Bots lean no: per Cursor staff (Sep 2026), Bot connectors support only OAuth with dynamic client registration, which Entra doesn't offer.
- **Backup and recovery.** Postgres takes automatic backups and supports point-in-time restore. Blob Storage keeps earlier versions and recovers deleted files. Recovery stays in one region for the pilot, since the KB can be rebuilt from its certified sources. This still needs Chris's confirmation.
- **Operations.** The operator role sits on Chris's team. It covers access, spending alerts, updates to the MCP server and restore tests. If extra help is needed, it comes from a separately scoped cloud partner, not the core MSP. The curator owns content only.
- **Cost.** The pilot's monthly list-price range and the sources for it are in the cost breakdown Chris received on 2026-09-29.

## 6.1 Build task B-1: grok.com and Entra ID connector test (week 1, added in v0.5)

This is the top open verification item in `../research/QCo-Grok-MCP-Entra-Research-Pack.md`. It runs before or during build week 1, because it decides which access path Grok users get.

- **Why it's needed.** `qco-company` signs users in through Entra ID. Entra doesn't support dynamic client registration, so an agent can connect only if it accepts a pre-registered (fixed) client ID and secret. Claude Cowork does, and that's confirmed. xAI's docs say grok.com custom connectors handle "OAuth or API keys", but they document no fixed client-ID field and no redirect URI. Community reports conflict on whether a client-ID screen exists.
- **Test.** Register a test app in Entra ID. Stand up a minimal Entra-protected MCP test endpoint with a single harmless tool. On a Grok Business or Enterprise workspace, add it as a custom connector (admin-provisioned), then connect it as an ordinary member.
- **Pass/fail criterion.** The test passes only if all three of these hold:
  1. grok.com accepts the fixed Entra client ID (and the secret, if it asks for one).
  2. The redirect URI grok.com uses is captured exactly and registered in the Entra app.
  3. A member completes Microsoft sign-in and makes one tool call that the server logs under that member's own identity.

  The test fails if grok.com has no fixed client-ID entry, if it insists on dynamic registration, or if sign-in or the tool call doesn't complete.
- **Record.** Capture the result, the exact redirect URI, whether each member signs in individually or the admin's sign-in is shared, and screenshots. Add the result to the research pack and to section 11.
- **Fallback if it fails.** Grok and Grok Bots use the REST API option instead of the MCP connector. That means direct HTTPS calls to the same fixed tools, with a bearer token issued for the person by Entra ID. Claude Cowork keeps the MCP path. The xAI API can also call a remote MCP server with a per-request token, but only if our own app does the Entra sign-in first. That is a second route to evaluate, not a default.
- **Effect on exit criteria.** The pilot still needs at least two of the three clients under the person's own identity. If B-1 fails, Grok counts toward that only through the REST path.
- **Owner.** The builder (in-house developer), with the operator for the Entra app registration.
- **Workspace prerequisite (resolved).** Custom connectors on Grok need a Grok Business or Enterprise workspace. Chris, as Director of AI and Technology, has admin control over Claude, Grok (including the Grok Business/Enterprise workspace) and Azure, so he can provision the workspace, the Claude connector and the Entra app registration directly. No trial or outside approval is needed before week 1.

## 7. Operator role: estimated weekly hours (role unnamed)

These are architect estimates for sizing the role. They are not measurements. The pilot records actual hours.

| Work | Build weeks 1–4 | Steady state |
|---|---|---|
| Access (Entra groups, client connector approvals, joiners and leavers) | 2–3 | 0.5–1 |
| Content rules (ban list, ingestion filter, tag scheme with the curator) | 2–3 | 0.5 |
| Cost watching (budget alert, monthly review) | 0.5 | 0.25 |
| Updates (MCP server releases, Azure advisories, secret rotation) | 2–4 | 0.5–1 |
| Restore tests and runbook (monthly test in steady state) | 1–2 | 0.25–0.5 |
| Log review and refusal triage | 1 | 0.5–1 |
| **Total** | **~8–13 hrs/week** | **~2.5–4 hrs/week** |

The build-phase figures assume the build itself (writing the MCP server) is done by a developer or partner, not the operator.

## 8. Curator role: estimated weekly hours (role unnamed)

These are estimates for one pilot domain of about 40–80 documents. The pilot measures the real rate.

| Work | Load phase (weeks 1–6) | Steady state |
|---|---|---|
| Collecting documents and interviewing the know-how holders | 3–5 | 1 |
| Cleaning, splitting and tagging (owner, verifier, review-by date, access, five-entity links) | 3–5 | 0.5–1 |
| Coordinating expert review and chasing approvals | 1–2 | 0.5–1 |
| Expiry queue (re-review before the review-by date) | 0 | 0.5–1 |
| Golden-question upkeep with the expert | 1 | 0.25 |
| **Total** | **~8–13 hrs/week** | **~3–4 hrs/week per domain** |

Expert verifier time is separate. The estimate is about 15–30 minutes per document to review, so roughly 1–2 hrs a week per expert during the load phase.

## 9. Strety connection (v0.4)

- **What exists.** Strety has an official MCP connector for Claude, ChatGPT and other clients that support custom connectors. It can both read and write, and changes it makes go straight back into Strety. Strety also has a public REST API (OpenAPI 3.1) that uses OAuth 2.0 and offers separate `read` and `write` scopes. Sources: the Strety MCP page, the Strety OpenAPI specification and the research pack.
- **Recommendation.** Use the REST API with a per-user token that has only the `read` scope. The `qco-company` MCP server calls it. The official Strety MCP should not be in the agent path, because we can't turn off its write capability.
- **Why not a nightly copy.** A nightly copy would read with one broad service identity and would store Strety data, and C3 allows pointers only. Live per-user reads keep each person's own Strety permissions and store nothing.
- **Filters.** The `qco-company` tools call only allowlisted endpoints: goals (Rocks) with their milestones and check-ins, the roles chart, and people limited to name and seat. They never call `/reviews`, shoutouts, currency-format metrics or finance-worded responsibilities (hard rule 6).
- **Limits.** The API allows 10 requests per 10 seconds per token and has no webhooks. Because reads are live, there's no need to poll for changes.
- **Build-time check (not blocking).** Confirm Strety's actual connector surface on the live QCo tenant. Specifically, confirm that a `read`-scope token is refused on writes, that Rock is the API's `goal` object, and that QCo hasn't renamed "Rocks" in Strety.
- **Policy note.** People who add the official Strety MCP to Claude on their own are using a write-capable tool outside the knowledge base. Chris may want an AI-use guideline that covers this.

## 10. Pilot area: Understanding Rocks (v0.4)

- **Scope.** Questions like "what are we working on this quarter," "who owns this Rock," "is it on track," "which Rocks touch the order-entry process or System X," and "what does this Rock mean." Answers combine live Strety Rock data with a small set of certified context documents: Rock descriptions and definitions of done, how QCo runs EOS, and a glossary.
- **Subject-matter expert.** The person who maintains the Rocks in Strety, most likely the Integrator (or the CEO if QCo has no Integrator), with Rock owners confirming their own Rocks. The Integrator signs off the context documents.
- **Curator hours (revised for Rocks).** Most of the facts are already structured in Strety, so there is less to collect. Loading (weeks 1–4) takes about 4–7 hours a week: linking Rocks to processes, systems and sites, writing definitions of done with the owners, and building the glossary. Steady state is about 1.5–3 hours a week, with a spike at each quarterly Rock reset of about 6–10 hours that quarter. The Integrator needs about 1 hour a week during loading and about 1 hour a quarter after that.
- **Milestone.** Company KB Pilot v0, Rocks, within 30–45 days.
- **Exit criteria:**
  - The current quarter's company Rocks are answerable live from Strety, with citations back to Strety.
  - About 40 golden questions, written with the Integrator, score at least 85% judged correct. This is a proposed target.
  - Zero leaks from reviews, shoutouts or finance content on the adversarial test set.
  - Every answer respects the person's own Strety permissions. A test user gets `acl_deny` on a Rock they can't see.
  - At least two of the three agent clients work under the person's own Entra ID and Strety identity.
  - The system writes nothing to Strety. A write attempt through the `read` token is shown to fail.
  - The Azure environment, restore test and operator runbook are signed off.
  - Actual curator and Integrator hours are recorded.
- **Budget.** The pilot is funded from the IT budget as a line item in this budget cycle. The spending alert is set at about $300 a month for option (a).

## 11. Decision log and open questions (v0.5)

**Decided**
- The cloud is Azure, with Microsoft 365 and Entra ID for sign-in.
- Operator and curator hours are estimated. The roles are not yet named.
- People access the KB only through the agents they already use (Claude Cowork, Grok and Grok Bots). Each person's own agent writes the answers.
- The IT budget pays for this as a line item. The spending alert is set at about $300 a month.
- Strety is read through its REST API with read-only scope, as the person asking. There is no nightly copy, and the official Strety MCP is not in the agent path.
- The pilot area is Understanding Rocks.
- Chris (Director of AI and Technology) has admin control over Claude, Grok (including the Grok Business/Enterprise workspace needed for custom connectors) and Azure. He provisions connectors, the workspace and the Entra app registration directly.
- **The MCP server is built in-house first, with a partner as fallback.** Before engaging a partner, QCo writes a spec covering the fixed tool list, refusal codes, the ban list and allowlists, Entra ID OBO, read-only Strety scope, logging and the exit criteria. The partner builds to that spec, and QCo owns the code and the Azure tenant.

**Blocking**
1. Confirm that each of Claude Cowork, Grok and Grok Bots can connect to a remote MCP server with Microsoft (Entra ID) OAuth sign-in, including org-wide connector admin. The REST fallback applies to any client that can't. Status by client:
   - **Claude Cowork:** confirmed. It accepts a fixed OAuth client ID and secret.
   - **Grok (grok.com):** in progress, with the test scheduled as build task B-1 in week 1 (section 6.1). If it fails, Grok uses the REST option.
   - **Grok Bots:** unverified, and the evidence leans no because Bot connectors support only OAuth with dynamic client registration. Plan on the REST option unless that changes.

**Not blocking (resolve before the pilot starts)**
2. The Integrator seat is TBD. The SME role stays unnamed until it's filled. It must be filled before the pilot starts, because the SME writes the roughly 40 golden questions and approves the content.

**Not blocking (resolve during the build or by weeks 2–3)**
3. Confirm Strety's actual connector surface on QCo's tenant: read-only refusal on writes, `goal` meaning Rock, and whether QCo has renamed Rocks.
4. Confirm that "Claude Codework" means Claude Cowork (assumed).
5. Set the review-by policy. The suggestion is 12 months, with refusal on expiry.
6. Confirm that single-region recovery is acceptable for the pilot.
7. Choose the search setup. The recommendation is Postgres only, with AI Search added later if needed.
8. Decide what bots that aren't acting for a named person may see: a general-staff slice, or blocked.
9. Name the operator and curator.

## 12. Latency budget (v0.5)

All figures in this section are estimates taken from public benchmarks and guidance. They are not QCo measurements, and the pilot replaces them with observed figures. The sources listed were supplied with the request and have not yet been independently re-checked against QCo's own setup.

**Target.** A typical Rocks question should take roughly 1 to 3 seconds end to end. Most of that time is the model writing the answer, not the knowledge base.

**Where the time goes (one request):**

| Step | Estimate | Source |
|---|---|---|
| MCP protocol overhead | 5–15 ms | Skillful.sh MCP server benchmarks (2026) |
| Postgres search over the small index (small result sets) | 8–15 ms | Skillful.sh (2026) |
| Strety REST API call | 100–300 ms, depending on where Strety's servers are relative to Azure East US | Estimate. Strety's hosting region is not confirmed |
| Model writes the cited answer (Claude or Grok, short response) | 1–2 s | Estimate |
| **Total, single-tool question** | **~1–3 s** | |
| **Total, multi-tool question** (for example `search_kb`, then `who_owns`, then a Strety fetch) | **up to ~4–5 s**, because round trips stack | |

**Cold starts.** If the server instance scales to zero, the first request after an idle period adds about 1 to 2 seconds. Microsoft's guidance treats scale-to-zero as a development or preview pattern and says production should keep warm instances (Microsoft Learn, "Choose an Azure service for your MCP server", 2026). A published production example (Mattrx) reports read p95 of 120 ms with minimum replicas set. Mitigation: keep 1 to 2 instances always warm in production. On App Service that means turning on Always On, which is available from the Basic tier up. The `minReplicas` setting applies if the server is instead hosted on Azure Container Apps.

**Connection timeout.** Azure's front-end load balancer drops a connection that sends no data for 230 seconds (adamtheautomator practical guide, 2026). Quick Rocks lookups are nowhere near this. It becomes a constraint only if heavier, long-running tools are added later, which would then need to stream progress or run as background jobs.

**Design principle.** Keep tool calls few. The fixed four tools (`search_kb`, `who_owns`, `get_doc`, `ask_product`) mean most questions resolve in one or two calls. There is no separate Azure OpenAI step to generate answers. The person's own agent writes the answer, so there is no second model call and no second model cost line.

**Pilot targets (proposed):**
- p50 end to end under 3 seconds.
- p95 end to end under 5 seconds.
- Measured over the roughly 40-question test set, with timings recorded alongside the accuracy results. Server-side timings per step come from the call log. End-to-end timings are measured from the agent, since the agent's own model time is outside our servers.
