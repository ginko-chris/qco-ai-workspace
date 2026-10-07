# QCo KB Split — Product KG (Stardog / ISA-95 / PLM-as-SoR) and Company Knowledge Base — Research Pack

**From:** Knowledge Systems Researcher
**To:** QCo Analyst / KG Architect / Agile PM (merge-ready; **evidence only — no QCo recommendations, no ranking**)
**Date:** 2026-09-28 (America/New_York). Web sources fetched 2026-09-28 unless noted.
**Audience context:** Chris (Director of AI & Technology) / QCo AI Council

**Decision context (as given, not evaluated here):** Chris split knowledge strategy into two lanes: (1) **product knowledge** in an **ISA-95 model in Stardog**, with **PLM (Autodesk Fusion Manage) as engineering system of record**, NetSuite remaining ERP and commercial identity; (2) **company knowledge** (internal, tribal/unstructured, certified-only, no HR/finance, org roles/accountability, Strety integration) in a separate system. Field-authority split as stated by the room: **PLM → EBOM; NetSuite → commercial identity (`internalid`) and MBOM; product director / CPQ → configuration rules.**

**Prior art this pack builds on (not redone):**
- [`QCo-Semantic-KG-Manufacturing-Docs-Research-Pack.md`](./QCo-Semantic-KG-Manufacturing-Docs-Research-Pack.md) — KG-vs-RAG evidence, ontology patterns, ACL leakage on graphs, MCP exposure patterns.
- [`QCo-FusionManage-ISA95-Stardog-Architecture-Brief.md`](../memos/QCo-FusionManage-ISA95-Stardog-Architecture-Brief.md) — FM REST v3 surface, B2MML element mapping, Stardog security matrix, Options A/B/C (Architect's brief; this pack re-verifies and extends selected facts rather than repeating them).
- [`QCo-KG-Council-POV.md`](../memos/QCo-KG-Council-POV.md), [`QCo-KG-Architecture-POV.md`](../memos/QCo-KG-Architecture-POV.md) — thin certified product graph, NetSuite `internalid` spine (now partly superseded).
- [`QCo-KG-Sprint-1-Work-Plan.md`](../memos/QCo-KG-Sprint-1-Work-Plan.md) — **v2.0 rebaselined 2026-09-28**, gates U1–U4, C1–C2, G8.
- [`QCo-Certified-Slices-Minimums-Checklist.md`](../checklists/QCo-Certified-Slices-Minimums-Checklist.md) — agent-path entry criteria.

**Confidence legend used throughout:**
- **High** — read directly in first-party documentation or a machine-readable spec fetched today.
- **Medium** — first-party marketing/help text, or consistent third-party sources; details may vary by tenant, tier, or release.
- **Low** — single third-party source, community post, or inference from adjacent facts.
- **Unverified** — could not be confirmed either way; stated as such. *Absence of evidence is not evidence of absence.*

---

## Table of contents

- [0. BLUF (evidence summary, no recommendations)](#0-bluf-evidence-summary-no-recommendations)
- [Decision Facts (requested up front)](#decision-facts-requested-up-front)
  - [DF-1. Strety: what it exposes, and are owner / seat / role first-class?](#df-1-strety-what-it-exposes-and-are-owner--seat--role-first-class)
  - [DF-2. Autodesk Fusion Manage as sole engineering SoR for ISA-95 Material Definition](#df-2-autodesk-fusion-manage-as-sole-engineering-sor-for-isa-95-material-definition)
  - [DF-3. Stardog operating cost and burden for a thin-IT SMB vs a property-graph / thin-crosswalk first step](#df-3-stardog-operating-cost-and-burden-for-a-thin-it-smb-vs-a-property-graph--thin-crosswalk-first-step)
  - [DF-4. Evidence bearing on Sprint 1 v2.0 gates (U1–U4, C1–C2, G8)](#df-4-evidence-bearing-on-sprint-1-v20-gates-u1u4-c1c2-g8)
- [Operating Model and Staffing (evidence)](#operating-model-and-staffing-evidence)
  - [OM-1. Staffing and hours by option](#om-1-staffing-and-hours-by-option)
  - [OM-2. Design assumption: formal product KG vs informal company KB tended by an associate](#om-2-design-assumption-formal-product-kg-vs-informal-company-kb-tended-by-an-associate)
  - [OM-3. Thin-IT evolution (descriptive observations)](#om-3-thin-it-evolution-descriptive-observations-from-the-evidence)
- [Part A — Product KG in Stardog on ISA-95 with PLM as SoR](#part-a--product-kg-in-stardog-on-isa-95-with-plm-as-sor)
  - [1. Stardog capabilities relevant here](#1-stardog-capabilities-relevant-here)
  - [2. ISA-95 as an ontology: renderings, coverage, commercial-data gaps](#2-isa-95-as-an-ontology-renderings-coverage-commercial-data-gaps)
  - [3. PLM as SoR: integration patterns, identity keys, ERP crosswalk, affected prior items](#3-plm-as-sor-integration-patterns-identity-keys-erp-crosswalk-affected-prior-items)
- [Part B — Company knowledge base (tribal / unstructured, internal)](#part-b--company-knowledge-base-tribal--unstructured-internal)
  - [4. Architecture patterns for certified, AI-queryable tribal knowledge](#4-architecture-patterns-for-certified-ai-queryable-tribal-knowledge)
  - [5. Tool landscape (verified-current only, with caveats)](#5-tool-landscape-verified-current-only-with-caveats)
  - [6. Exclusion controls for HR and financial data](#6-exclusion-controls-for-hr-and-financial-data)
  - [7. Modeling organization roles and accountability](#7-modeling-organization-roles-and-accountability)
  - [8. Strety integration (detail)](#8-strety-integration-detail)
- [Part C — Cross-cutting](#part-c--cross-cutting)
  - [9. Boundaries between the two systems](#9-boundaries-between-the-two-systems)
  - [10. Industry-observed phased rollout patterns (descriptive)](#10-industry-observed-phased-rollout-patterns-descriptive)
  - [11. Risks / failure modes](#11-risks--failure-modes)
  - [12. Sources](#12-sources)
- [Appendix: Open verification items](#appendix-open-verification-items)

---

## 0. BLUF (evidence summary, no recommendations)

1. **Strety has a real, public, versioned REST API** (OpenAPI 3.1, OAuth 2.0, `read`/`write` scopes, 10 req/10 s per token) covering Rocks (API resource name `goal`), Issues, Scorecard metrics + check-ins, To-Dos, Headlines, Meetings, V/TO, Docs, People, Teams, and a **read-only Accountability Chart** (`roles_charts` / `roles`). **Seats and people are first-class objects with UUIDs** (Role → parent Role tree; Role → assignee Person UUIDs; Person → `seats[]` with seat UUIDs). **The 5–7 EOS "roles" inside a seat are structured but ID-less text entries** (`responsibilities[]` title/description). Rocks, Issues, Metrics and To-Dos are owned by a **Person UUID, not a seat**. **No webhooks were found in the spec; no official Zapier/Make connector was found.** An **official Strety MCP** exists and is **read/write** by design. The API also exposes **performance reviews** (`/reviews`) and shoutouts — HR-adjacent content inside the EOS tool. (High, spec fetched today — [§DF-1](#df-1-strety-what-it-exposes-and-are-owner--seat--role-first-class))
2. **Fusion Manage covers engineering Material Definition well** (items, revisioning workspaces, EBOM, change orders, lifecycle, where-used, audit; REST v3 with bulk reads and `If-Modified-Since` incremental pulls). It **has date-based revision effectivity** (BOM viewed "as of" a date, released revision "in effect"), and it **can also hold an MBOM** (Autodesk University customer class on the FM "mBOM Editor"; 2023 tenant template stores EBOM and MBOM in one Items workspace). That capability overlaps the stated "MBOM → NetSuite" authority split. NetSuite Advanced BOM has its own **non-overlapping date-effective BOM revisions**, so two effectivity/revision schemes coexist. **No first-party Autodesk FM–NetSuite connector found**; partner connectors only. (High for FM API; Medium for mBOM Editor and templates — [§DF-2](#df-2-autodesk-fusion-manage-as-sole-engineering-sor-for-isa-95-material-definition))
3. **Stardog Enterprise pricing is not published** (sales call required). Stardog Cloud Free (shared, ≤1M edges, 3 DBs) exists; Enterprise Cloud is dedicated with 99.9% SLA, daily backups/30-day retention, Entra/OIDC SSO. Self-managed means Kubernetes + Helm backup/restore runbooks. Some features (entity resolution, graph analytics) need Spark/Databricks/EMR in self-managed setups; Voicebox in a customer VPC needs Bedrock / Azure AI Foundry / Databricks. **Voicebox API access is "currently limited and not enabled for most users"** (Stardog docs). The **Stardog Cloud MCP server is NL→SPARQL (3 tools), remote mode is beta**, and it authenticates with an app API key plus an optional SSO token override. (High for docs, Unverified for cost — [§DF-3](#df-3-stardog-operating-cost-and-burden-for-a-thin-it-smb-vs-a-property-graph--thin-crosswalk-first-step))
4. **Stardog security defaults need deliberate configuration.** Named-graph security is **off by default**. Unauthorized graphs are **silently dropped**, so queries return empty results rather than errors. Reasoning **schema axioms are visible to every reasoning user**. Virtual-graph credential passthrough is documented only for **Databricks under Entra ID**. (High)
5. **There is no standards-body OWL/RDF rendering of ISA-95.** The options are DTDL models (Digital Twin Consortium), an academic OWL ODP limited to the equipment hierarchy, the OPC UA ISA-95 model, and research ontologies. B2MML (MESA, XSD) is the canonical machine form. **ISA-95 Part 2 covers the Level 3/4 exchange** (material, equipment, personnel, process segments, operations definition/schedule/performance). It **has no native objects for cutsheets, pricebooks, product families/variants as marketed, photometrics (IES LM-63 / TM-33), engineering change orders, or CPQ configuration rules**. Those require extension vocabularies such as schema.org `ProductGroup`/`hasVariant`, GoodRelations, or a QCo namespace. (High for the ISA scope statement, Medium for the gap list — [§2](#2-isa-95-as-an-ontology-renderings-coverage-commercial-data-gaps))
6. **HR/finance exclusion controls in M365 are real but leaky.** Purview DLP for Copilot can block processing of **sensitivity-labeled** files and emails, **but the items can still appear in citations**. Policy changes take **up to 4 hours** to apply. **DLP cannot scan files uploaded into prompts.** A **Jan–Feb 2026 bug** let Copilot Chat summarize confidential-labeled email despite DLP. Microsoft documents that label behavior differs across apps. Trainable classifiers for **HR, Employee disciplinary action, Finance, Financial statements, and Financial audit** exist (English). All of these are probabilistic, label-dependent controls. They are not guarantees. (High — [§6](#6-exclusion-controls-for-hr-and-financial-data))
7. **Staffing evidence (order-of-magnitude only):** Stardog/RDF EKG guides cite multi-week standups and ~2–3 FTE ongoing or partner ontology consulting ($80K–$200K in one third-party range); single-domain graphs 6–12 weeks with a 20–40% FTE project lead; company KM medians are enterprise-scale (APQC: 8 FTE median) while vendor KB cases reclaim ~2.5–5+ hrs/week of verification triage. **Formal ontology vs content-curator split is common**; "associate tends company KB alone" fails on verification drift, ACL mistakes, and HR/finance leakage without SME verifiers and allowlists ([§Operating Model](#operating-model-and-staffing-evidence)).
8. **The Sprint 1 v2.0 gates are consistent with the evidence, with specific pressure points.** U3 must handle **FM's own MBOM capability** and **two revision/effectivity schemes**. C2 now has a factual answer: **seat and person are first-class; seat "roles" are text-only; Rock ownership is person-level**. C1 must account for **Strety `/reviews`, shoutouts, currency-format metrics and finance-worded seat responsibilities**. G8 (OBO) meets friction from **Stardog Cloud MCP's API-key model** and from **Strety's per-user OAuth (a benefit) combined with write-scoped MCP (a risk)**. ([§DF-4](#df-4-evidence-bearing-on-sprint-1-v20-gates-u1u4-c1c2-g8))

---

## Decision Facts (requested up front)

### DF-1. Strety: what it exposes, and are owner / seat / role first-class?

**Primary source:** Strety v1 OpenAPI spec, publicly downloadable at <https://2.strety.com/api/docs/v1/openapi.yaml> (fetched via `curl` 2026-09-28, 382 KB, `openapi: 3.1.0`, title "Strety API", version 1.0.0; changelog latest entry **August 2026**). The HTML docs viewer at <https://2.strety.com/api/docs/v1> redirects to a Strety login. The spec file itself was retrievable without login. **Confidence: High** for everything quoted from the spec. It is a vendor spec, not a tested integration, so runtime behavior was **not** exercised.

#### DF-1.1 Exposure matrix

| Capability | Rocks | Issues | Scorecard | Accountability Chart | To-Dos | Evidence / confidence |
| --- | --- | --- | --- | --- | --- | --- |
| **REST API (public spec)** | Yes. Resource `goal` (+ `check_ins`, `milestones`, archive/backlog); CRUD | Yes. `issue` CRUD + archive; filter `owner_id`, `resolved`, `issue_type` | Yes. `metric` CRUD + `check_ins` (daily/weekly/monthly/quarterly/annual) | **Read-only.** `roles_charts` GET, `roles_charts/{id}/roles` GET; spec: "read-only object and cannot be created or modified via the API" | Yes. `todo` CRUD | High ([spec](https://2.strety.com/api/docs/v1/openapi.yaml)) |
| **Incremental sync** | `updated_after`, `created_after`, `ids` filters on list endpoints (goals, issues, metrics, people, roles) | same | same | same (roles list supports `updated_after`) | same | High (spec parameters) |
| **Webhooks** | **None found**. The string "webhook" does not occur in the spec | — | — | — | — | High that the spec has none; **Unverified** whether webhooks exist outside the public API |
| **Zapier / Make** | **No official connector found.** Not listed on [Strety integrations](https://strety.com/integrations/) or the [Help Center integrations collection](https://help.strety.com/en/collections/3600950-integrations) (15 articles, none Zapier/Make) | — | — | — | — | Medium (absence on first-party pages); **Unverified** that none exists |
| **Other automation** | Community **n8n node** (MIT, third-party, claims coverage of goals, issues, metrics, roles charts, reviews) ([GitHub](https://github.com/ajoshuasmith/n8n-nodes-strety)). A Pipedream "Strety" app listing surfaced in search ([Pipedream](https://pipedream.com/apps/strety)); contents not verified | — | — | — | — | Low (third-party) |
| **Official MCP** | Yes. Read + update Rocks | Yes. Log/update | Yes. Pull scorecard, log numbers | **Not listed** in MCP scope (the list is To Dos, Issues, Headlines, Scorecard, Rocks, Docs, Agendas, Projects) | Yes | Medium ([Strety MCP page](https://strety.com/integrations/strety-mcp/)). Per-user sign-in; "only reaches what your Strety permissions already allow"; **writes back to Strety**; no extra cost |
| **Native integrations** | Rocks can link to MS Planner plans for progress | — | BrightGauge-powered scorecards | — | Two-way sync with Asana, Planner, MS To Do, HubSpot, Monday, ClickUp, Todoist, Google Tasks, Procore; PSA links | Medium ([integrations](https://strety.com/integrations/)) |
| **Bulk export (CSV/file)** | **Unverified.** No first-party export doc found | — | — | — | — | Unverified |
| **M365 surface** | Teams app, Outlook add-in, Microsoft SSO; SharePoint/OneDrive linking for Docs (formerly Playbooks) | | | | | Medium ([integrations](https://strety.com/integrations/)) |

**Terminology note (Rocks = `goal`):** The API has no resource named "rock". Strety lets admins rename tools ("Rocks" → "Goals") ([Adminland help](https://help.strety.com/en/articles/9579761-strety-adminland); [Changelog #8 2024](https://help.strety.com/en/articles/9702853-strety-changelog-8-2024-tool-names-slack-coaches-dashboard)). Strety's own blog says "Rocks (aka goals)" ([blog](https://strety.com/blog/having-goals-business-goals/)). The V/TO schema contains a `quarterly_rocks` section. Mapping **Rock ≡ API `goal`** is therefore **Medium-High**: strongly implied, but no spec sentence states it verbatim. The `goal` fields align with the Rock help doc (company flag, owner, due date, progress type, parent/annual-goal link) ([Rocks guide](https://help.strety.com/en/articles/8887868-strety-rocks-complete-guide)).

#### DF-1.2 Are owner, seat and role first-class structured objects with IDs?

| EOS concept | Strety API representation | First-class with ID? | Notes |
| --- | --- | --- | --- |
| **Accountability Chart** | `RolesChart` (UUID, `name`, `description`, `company_roles_chart` boolean, `space` → person/team/project UUID) | **Yes** | "Exactly one chart per organization is marked as the company chart." Read-only via API |
| **Seat** (EOS box) | `Role` (UUID, `name`, `role_type` ∈ {`standard`, `visionary`, `integrator`, `owner_box`}, `status` ∈ {`fulfilled`, `needs_hire`, `needs_more_resources`, null}, `parent` → Role UUID, `assignees` → Person UUIDs, `roles_chart` → UUID) | **Yes** | Seats form a **tree** via `parent`. Strety's object is named "Role", but it corresponds to an EOS **seat** |
| **Seat's 5–7 "roles"** (EOS accountabilities) | `Role.attributes.responsibilities[]` = `{title, description, description_html}` | **No IDs.** Structured list of text entries embedded in the seat | Ordered "canonically". Any link from a knowledge item to a specific responsibility would need a synthetic key such as (seat UUID + position/title). That key would be fragile if responsibilities are reordered or renamed (inference) |
| **Seat owner / assignee** | `Role.relationships.assignees` → Person UUIDs; **plus** `external_assignees[]` = **free-text names** for non-Person assignees | **Mixed.** Internal assignees are IDs; external assignees are free text | EOS says one owner per seat ([EOS Worldwide](https://www.eosworldwide.com/accountability-chart)). Strety's `assignees` is an **array**, so the API does not enforce a single owner |
| **Person** | `Person` (UUID, `email`, `name`, `title`, access `role` ∈ {member, admin, account_owner, observer, external}, `deactivated_at`, **`seats[]` = {id UUID, name}**) | **Yes** | Read-only. `email` enables a join to Entra UPN (inference). Whether `Person.seats[].id` equals `Role.id` is **likely but not stated** in the spec (Unverified) |
| **Rock owner** | `goal.relationships.assignee` → Person UUID (nullable); `space` → team/person/project; `parent` → goal | **Yes, person-level** | **No goal → seat relationship.** Seat accountability for a Rock can only be derived as Rock → Person → `seats[]` (inference, lossy if a person holds multiple seats) |
| **Issue owner** | `issue.relationships.owner` → Person | Yes, person-level | same caveat |
| **Measurable owner** | `metric.relationships.assignee` → Person | Yes, person-level | `number_format` includes `currency`, so financial values can appear (see [§6](#6-exclusion-controls-for-hr-and-financial-data)) |
| **To-Do owner** | `todo.relationships.assignee` → Person | Yes | — |
| **Team** | `Team` (UUID, `leadership` flag, `members` → Person UUIDs) | Yes | — |

**Plain answer:** Owner (Person), seat (API "Role") and chart are **first-class, UUID-keyed, read-only objects**. The EOS **roles/accountabilities inside a seat are structured text without IDs**. External seat holders are **free text**. Rocks, Issues, Measurables and To-Dos are keyed to a **person**, not to a seat.

#### DF-1.3 Other API facts relevant to QCo constraints

- **Auth:** OAuth 2.0 Authorization Code; scopes `read` (default) and `write`; access token **2 h**; refresh tokens rotate on use; RFC 7009 revoke; RFC 7662 introspect (spec). Per-user tokens support **on-behalf-of-user** reads (inference: a token carries that user's Strety permissions; the MCP page states the same for the MCP).
- **Rate limits:** **10 req / 10 s per token; 100 req / 10 s per application**; HTTP 429 with `Retry-After` (spec).
- **Concurrency:** all PATCH require `If-Match` (ETag) (spec).
- **HR-adjacent endpoints exist:** `/reviews` ("Operations for accessing performance reviews"; answers exposed once completed; visible via HR Center access) and `/shoutouts` (peer recognition). The `Role.status` value `needs_hire` and the spec's own example responsibility, "Produce and circulate P&L by the 5th of each month", show that Accountability Chart data can carry hiring and finance context (spec).
- **Deprecation:** `/playbooks` endpoints retire **2027-02-19**; use `/docs` (spec changelog, Aug 2026).
- **Community MCP** ([brentwpeterson/mcp-strety](https://github.com/brentwpeterson/mcp-strety)) covers To-Dos and People only, including delete. It is third-party and low-adoption (Low).

---

### DF-2. Autodesk Fusion Manage as sole engineering SoR for ISA-95 Material Definition

*Fusion Manage (formerly Fusion 360 Manage / Fusion Lifecycle; API host pattern `*.autodeskplm360.net`). The Architect brief §4.3 already maps FM objects to B2MML V0701 elements. This section re-verifies and adds effectivity, MBOM and revision evidence.*

#### DF-2.1 Data model (as documented)

| Concept | FM representation | Evidence / confidence |
| --- | --- | --- |
| **Item** | Record in a workspace (e.g., Items); sections/fields defined per tenant; URN `urn:adsk.plm:tenant.workspace.item:TENANT.{ws}.{dmsId}` | High ([Item Details endpoint](https://help.autodesk.com/cloudhelp/ENU/FLC-RestAPI/files/FLC_RestAPI_Advanced_Functionalities_item_details_endpoints_html.htm)) |
| **Lifecycle / workflow state** | `lifecycle` (e.g., Unreleased, Pre-Production, Production) and `currentState`; flags `latestRelease`, `workingVersion`, `itemLocked` | High (same payload); Pre-Production/Production release states from the Fusion client docs ([FM interface](https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-INTERFACE.htm)) |
| **Revision** | Revisioning workspaces; working vs released versions. Revision number generation **auto or manual as a tenant-wide setting**; when manual, the ECO affected-items revision column is editable | High for working/released flags; Medium for the auto/manual setting (Autodesk community answer) ([forum](https://forums.autodesk.com/t5/fusion-manage-forum/how-to-set-the-revision-of-an-existing-item-once-it-is-created/td-p/12929270)) |
| **BOM** | `bom`, `nestedBom` / `flatBom` (`/bom-items`), `whereUsed` links on each item; BOM views are tenant-defined | High (payload); BOM view field IDs are tenant-specific (Architect brief) |
| **Change** | Change Orders, Change Requests, Problem Reports, Change Tasks, Change Approval Templates workspaces; release via change order or "Quick Release" (single components, to Pre-Production without CO) | High ([FM interface](https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-INTERFACE.htm)) |
| **Effectivity** | **Date-based revision effectivity.** BOM configurations "Released Revisions: displays the released revision for all children in effect on the date given"; "If the selected date is outside of the effectivity range, the system identifies the revision that is or was in effect on that date". **Pinning** locks a child revision in a parent | High ([BOM Comparison help](https://help.autodesk.com/cloudhelp/ENU/PLM-360-User/files/UG-BOMTAB-BOMCOMPARE.htm)). **Serial/lot/unit effectivity: Unverified** in Autodesk docs (a partner blog claims SBOM date/serial/lot effectivity — [coolOrange](https://www.coolorange.com/en/blog/bom-management-in-fusion-manage-plm-why-it-matters-the-editor-features-youll-actually-use), Low) |
| **MBOM** | FM **can host MBOMs**. An Autodesk University 2024 class covers a customer converting EBOM→MBOM "using the mBOM Editor in Autodesk Fusion Manage" ([AU 2024](https://app.learn-one.autodesk.com/autodesk-university/class/Autodesk-Fusion-Manage-in-Production-How-to-Populate-CAD-Data-Convert-BOMs-and-Integrate-with-ERP-2024)). An Autodesk community answer says the 2023 tenant template stores "both EBOMs and MBOMs … in the same workspace" ([forum](https://forums.autodesk.com/t5/fusion-manage-forum/item-vs-vault-item/td-p/12708876)). An OOTB EBOM→MBOM workflow idea was still "Gathering Support" ([idea, 2022](https://forums.autodesk.com/t5/fusion-manage-ideas/ebom-and-mbom/idi-p/11229170)) | Medium. Whether QCo's tenant has the mBOM Editor enabled is **Unverified** |
| **Documents / attachments** | `/attachments` per item | High (Architect brief, FM v3 docs) |
| **Suppliers / sourcing** | Workspaces exist in templates; outside ISA-95 Part 2 scope | Medium |
| **Configuration rules / CPQ** | **No evidence found** that FM is positioned as a CPQ or configuration-rule engine | Unverified (absence) |

#### DF-2.2 API

- **REST v3** (`/api/v3/...`): item GET, **bulk item details** (`Accept: application/vnd.autodesk.plm.items.bulk+json`, **100 items/page**), **`If-Modified-Since`** incremental listing that is **permission-filtered to the calling user**, plain-text paragraph option (High — [Item Details](https://help.autodesk.com/cloudhelp/ENU/FLC-RestAPI/files/FLC_RestAPI_Advanced_Functionalities_item_details_endpoints_html.htm)).
- BOM GET, versions, workflow history, attachments, search, tenant logs (Architect brief §4.3, citing FM REST docs; not re-fetched here except Item Details).
- **Events:** APS webhook `workflow.transition` for FM (Architect brief, [APS](https://aps.autodesk.com/en/docs/webhooks/v1/reference/events/flc_events/workflow.transition)).
- **Not exposed (per Architect brief):** documented endpoints for Relationships and Managed/Affected Items tabs were not found (Unverified).
- **APS Manufacturing Data Model GraphQL** covers Fusion *design* data plus FM extension properties (item number, revision, lifecycle, change order on components). It does **not** cover FM workspaces in general (Architect brief; Medium).

#### DF-2.3 FM ↔ NetSuite and ISA-95 Material Definition coverage

| ISA-95 / B2MML Material Definition element | FM coverage | Gap / where else it lives | Confidence |
| --- | --- | --- | --- |
| `MaterialDefinition.ID`, `Version`, description, class | Item number, revision, category/classification | — | High |
| `EffectiveStartDate` / end | Revision release date and effectivity range (date) | Serial/lot effectivity Unverified. NetSuite BOM revisions carry their **own** Effective Start/End dates, non-overlapping ([Oracle: BOM Revision](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_1517399803.html); [Updating BOM revision dates](https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_1518447156.html)) | High |
| `MaterialDefinitionProperty` | Item fields (tenant-defined) | Commercial properties (price, cost, availability) are NetSuite | High |
| Assembly / BOM (engineering) | EBOM | — | High |
| `OperationsMaterialBill` (manufacturing BOM) | **Possible in FM (mBOM Editor)** | Stated authority: **NetSuite MBOM** (Advanced BOM + revisions, feature-dependent). **Two candidate MBOM hosts exist.** | Medium |
| Engineering change | Change Orders | **No ISA-95 object** (Architect brief) | High |
| Configuration / option rules | Not evidenced in FM | Stated authority: product director / CPQ. **No ISA-95 object** | Medium |
| FM → NetSuite transport | Partner connectors (vdR Nexus, KETIV DataBridge, coolOrange powerGate) or custom; **no first-party Autodesk connector found** | — | Medium (vendor claims) ([coolOrange](https://www.coolorange.com/en/blog/bom-management-in-fusion-manage-plm-why-it-matters-the-editor-features-youll-actually-use); Architect brief) |

**PLM–ERP model mismatch (industry framing):** PLM is revision-based and ERP is effectivity-based. One PLM part can become several ERP materials ("from the part in PLM, several materials in ERP are born") ([Beyond PLM, 2024](https://beyondplm.com/2024/01/14/3-benefits-of-using-graph-based-digital-thread-product-model-for-plm-erp-integrations/); vendor-authored, author discloses OpenBOM bias). This bears directly on whether the crosswalk is 1:1 or 1:N (see [§3.3](#33-identity-keys-and-the-netsuite-crosswalk)).

---

### DF-3. Stardog operating cost and burden for a thin-IT SMB vs a property-graph / thin-crosswalk first step

#### DF-3.1 Licensing and tiers (as published)

| Offering | What is published | Confidence |
| --- | --- | --- |
| **Stardog Free** (self-install) | No cost; **not open source**; **renewable 1-year license**; commercial use allowed; **excludes HA, caching, backups, LDAP, full connector set**; community support only | High ([Pricing FAQ](https://www.stardog.com/pricing/)) |
| **Stardog Cloud Free** | Shared hosting; **≤1M stored edges**; 3 databases; 3 Voicebox DBs (table); SSO/Google auth; community support | High ([Stardog Cloud](https://www.stardog.com/stardog-cloud/)) |
| **Stardog Cloud "Essentials"** | Referenced in text ("Everything in Essentials, plus…") and in Voicebox docs ("Essentials and Enterprise customers") but **absent from the current plan table** | **Unverified** whether Essentials is currently sold |
| **Stardog Enterprise (Cloud)** | "Contact us"; dedicated hosting; custom edge limit; unlimited DBs; SSO/Google, **Azure AD**, OpenID; **99.9% uptime SLA**; **daily backup / 30-day retention**; Private Link; BI/SQL endpoint; professional services; option to self-manage; Premium support | High (published), **price Unverified** |
| **Stardog Enterprise (on-prem / customer VPC)** | License via Order Form; Base Support included; auto-renewing subscription terms with **90-day non-renewal notice** in the Enterprise Agreement | High ([Pricing](https://www.stardog.com/pricing/); [Enterprise Agreement](https://www.stardog.com/legal/stardog-enterprise-agreement/)) |
| **Price points** | **Not published**. "we will need just a few minutes with you on a call" | High that pricing is unpublished. **No credible public price data found** (Unverified) |
| **Database-only SKU** | "No. … we do not offer that as a separate product" | High ([Pricing FAQ](https://www.stardog.com/pricing/)) |

#### DF-3.2 Hosting and ops burden

| Dimension | Stardog Cloud (managed) | Self-managed (customer VPC / on-prem) | Evidence |
| --- | --- | --- | --- |
| Infra | AWS regions us-west-2, us-east-1, eu-central-1 | Kubernetes (EKS/AKS/GKE), AWS/Azure marketplaces, OpenShift | High ([Deployment](https://www.stardog.com/deployment/); page contains dated "Q1 '24" labels) |
| Backups / DR | Daily, 30-day retention (Enterprise) | Operator-run: Helm runner pod `stardog-admin server backup` to PVC or S3; restore requires scaling down pods and deleting PVCs | High ([backup](https://support.stardog.com/support/solutions/articles/151000211575-how-to-do-server-backup-in-stardog-helm-charts); [restore](https://support.stardog.com/support/solutions/articles/151000211579-how-to-do-server-restore-in-stardog-helm-charts)) |
| Uptime guarantee | 99.9% (Enterprise Cloud only) | "We can only guarantee uptime as part of our Stardog Cloud managed service offering" | High |
| LLM (Voicebox) | Included per license (Voicebox DB count limited by `voicebox.count.limit`) | Requires **Bedrock, Azure AI Foundry or Databricks Mosaic AI** (customer VPC) | High ([Deployment](https://www.stardog.com/deployment/); [Voicebox dev guide](https://docs.stardog.com/voicebox/voicebox-dev-guide/)) |
| Entity resolution / graph analytics | (Cloud column blank on page) | **Requires Databricks, EMR Serverless or Apache Spark** | High (published matrix) |
| Security posture out of the box | Configured by tenant admin | Default install "minimal and insecure" (Architect brief citing Stardog Security Model); named-graph security **off by default** | High ([Named Graph Security](https://docs.stardog.com/operating-stardog/security/named-graph-security)) |
| Identity | SSO/Azure AD/OIDC (Enterprise) | JWT/OAuth with role claims (Architect brief) | High |
| MSP familiarity | **Unverified.** No data found on MSP/SMB familiarity with Stardog, SPARQL or SHACL | — | Unverified |

#### DF-3.3 Skills required (from Stardog's own docs)

- **RDF modeling, SPARQL, OWL/RDFS reasoning, SHACL** (platform basis) ([Features](https://www.stardog.com/features/)).
- **Voicebox-specific modeling discipline:** labels/comments only via `rdfs:label`/`rdfs:comment`; prefer `so:domainIncludes`/`rangeIncludes` over `rdfs:domain`/`range`; unique prefixes; prefer IRIs over literals; **5–10 curated stored example queries to start, typically ≤50 for ">90%" accuracy** (vendor claim); SELECT-only; stored queries must be reloaded if deleted; namespace edits break preprocessed queries ([Voicebox dev guide](https://docs.stardog.com/voicebox/voicebox-dev-guide/)).
- **Security administration:** database-qualified named-graph grants (`named-graph:db\iri`), virtual-graph and data-source resource permissions ([VG security](https://docs.stardog.com/operating-stardog/security/virtual-graph-security)).
- **Virtual-graph mapping** (SMS/R2RML). Known limits: duplicate solutions possible, `MINUS` not translated to SQL, datatype comparison caveats, **named graphs in R2RML not supported** ([Virtual Graphs](https://docs.stardog.com/virtual-graphs/)).

#### DF-3.4 Side-by-side: Stardog vs property-graph / thin-crosswalk first step (descriptive)

Builds on Architect brief §5.1. Only facts with sources are shown. Cells marked "inference" are reasoning from those facts.

| Dimension | Stardog (RDF/OWL, Cloud or self-managed) | Property graph (e.g., managed LPG) | Thin crosswalk (Postgres tables + governed API/MCP) |
| --- | --- | --- | --- |
| License cost visibility | Unpublished Enterprise; Free tiers with limits (≤1M edges Cloud Free; no backups/HA on Free) | Varies by vendor (not researched here) | Postgres is open source; managed Postgres widely available (not priced here) |
| Formal schema + validation | OWL + **SHACL native** | App-level or vendor constraints | SQL constraints + app validation |
| Query-time reasoning | Yes (schema axioms global to reasoning users) | No (not OWL) | No |
| Virtualization over sources | Yes (virtual graphs; caveats above) | Generally no (not researched) | No (ETL) |
| Row/graph-level security | Named graphs + sensitive-property masking; silent-drop semantics | Label/property privileges (Neo4j per Architect brief) | Postgres RLS ([docs](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)) |
| NL/LLM surface | Voicebox (NL→SPARQL), Cloud MCP (3 tools) | Text2Cypher-class (Architect brief cites ~30%+ execution match on benchmarks, older models) | Governed tools only (no NL→SQL in product path per prior QCo decisions) |
| Skills | SPARQL/OWL/SHACL (MSP familiarity Unverified) | Cypher (common per Architect brief) | SQL (common) |
| Reversibility | RDF is a W3C standard, so data is portable. Stardog-specific features (VGs, Voicebox, ICV) are not | Moderate | Can be projected to RDF later (Architect brief) |
| Ops burden on thin IT | Low-moderate on Enterprise Cloud (vendor-run). High self-managed (K8s, Helm backup/restore, optional Spark) | Varies | Low-moderate (inference) |

**Counter-evidence worth noting:** Arguments for RDF at this scale do exist. A 2026 preprint unifies 11 simulated manufacturing sources in an ISA-95-informed RDF graph. It reports that blocking cross-system joins cut signal recall from 1.00 to 0.31, and it exposes the graph through MCP tools. Its manifest was author-constructed ("verification not independent validation"), and the sources were simulated ([arXiv:2608.24918](https://arxiv.org/abs/2608.24918); Low-Medium).

---

### DF-4. Evidence bearing on Sprint 1 v2.0 gates (U1–U4, C1–C2, G8)

Source plan: [`QCo-KG-Sprint-1-Work-Plan.md`](../memos/QCo-KG-Sprint-1-Work-Plan.md) **v2.0 (2026-09-28)**. Evidence only. The Researcher pass/fail on the plan is not issued here.

| Gate (as written in v2.0) | Evidence found | Bearing (descriptive) |
| --- | --- | --- |
| **U1 Crosswalk.** Steward named; no third item master | ISA-95 Part 7 (Alias Service Model) defines services for mapping equivalent identifiers across domains, but "the identification of the organization and owner of each system and the lifecycle management of the equivalent object namespaces is outside the scope" ([ISA](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard)). B2MML IDs "are not intended to act as global object IDs" (Architect brief). PLM part → **multiple ERP materials** is a documented pattern ([Beyond PLM](https://beyondplm.com/2024/01/14/3-benefits-of-using-graph-based-digital-thread-product-model-for-plm-erp-integrations/)) | Standards give **alias semantics but no steward**. Crosswalk cardinality may be **1:N** (FM item/rev → several NetSuite items), not 1:1. If Stardog mints its own product IRIs, it risks becoming the de facto "third item master" U1 forbids, unless IRIs derive from FM/NetSuite keys (inference) |
| **U2 SoR split.** PLM = engineering definition; NetSuite = commercial identity | FM exposes item, revision, EBOM, change and lifecycle via REST v3 with permission-filtered incremental reads (High). NetSuite owns `internalid`, price and cost (prior packs) | Evidence supports the split as technically feasible. Partner-only FM→NetSuite transport (no first-party connector) is an integration dependency outside the KG |
| **U3 Field authority.** Must name EBOM→PLM, MBOM→NetSuite, config rules→product director/CPQ | (a) **FM can host MBOMs** (mBOM Editor; 2023 template stores EBOM+MBOM in one workspace; Medium). (b) **Two revision/effectivity schemes**: FM revision letters with date effectivity vs NetSuite BOM revisions with non-overlapping Effective Start/End dates (High). (c) **ISA-95 has no object for configuration rules or ECOs** (Architect brief; §2). (d) CPQ platform for QCo is **unnamed/Unverified** | U3 as written is consistent with evidence, but the table has to resolve (a): if FM MBOM is used anywhere, there are two MBOM sources. It also has to map (b): FM rev ↔ NetSuite BOM revision. And (c) means config rules need a non-ISA extension class and a named source system, or Stardog would hold rules with no SoR (inference) |
| **U4 Store path.** Option B or Stardog + ontology owner explicit | DF-3: pricing unpublished; Free tiers constrained; managed Cloud reduces ops burden; self-managed adds K8s/backup runbooks; Voicebox API "limited"; MCP remote mode beta; skills = SPARQL/OWL/SHACL; Voicebox needs curated stored queries | Evidence supports U4's framing that **ownership/skills, not software, is the gating factor** (consistent with Architect brief trigger #3). A cost figure for Stardog cannot be supplied from public sources (Unverified) |
| **G8 OBO.** End-user identity; no bot see-all | Stardog: Entra/OIDC JWT with role claims; **Cloud MCP authenticates via Voicebox app API key**, with optional `x-sd-auth-token` SSO override ([stardog-cloud-mcp README](https://raw.githubusercontent.com/stardog-union/stardog-cloud-mcp/main/README.md)). Named-graph security silently drops unauthorized graphs (High). Strety: per-user OAuth tokens (High). FM: `If-Modified-Since` listing filtered to the calling user (High). Purview: Copilot runs "in the security context of the user" ([Purview DLP for Copilot](https://learn.microsoft.com/en-us/purview/dlp-microsoft365-copilot-location-learn-about)) | Stardog MCP's **default is an app-level key**. OBO depends on passing the user's SSO token per request (inference). A **sync job** reading FM or Strety with a service identity to populate a store is a "see-all" read at ingest. That differs from query-time OBO, and the plan does not distinguish the two (observation). Silent-drop semantics mean `acl_deny` cannot be detected from an empty SPARQL result without extra logic (inference) |
| **C1 Company ban.** HR/finance/transactional/product-copy blocked at ingest | Strety API exposes **`/reviews` (performance reviews)** and **`/shoutouts`**; `Role.status=needs_hire`; metrics with `number_format=currency`; the spec's own example responsibility mentions **P&L**. Purview classifiers for HR/Finance exist but are English-only and probabilistic; DLP excludes labeled items from Copilot processing but they "could be available in the citations" (High) | Ingest allowlists at **endpoint and field level** are needed to exclude HR/finance-adjacent Strety content. Label-based exclusion in M365 has documented gaps (§6) |
| **C2 Strety SoR.** Cite, not duplicate; org-graph waits on research pack (first-class owner/role vs free text) | DF-1.2: **Chart, seat (API "Role") and person are UUID-keyed and read-only**; **seat responsibilities are ID-less text**; **external assignees are free text**; **Rock/Issue/Metric/To-Do owners are persons, not seats**; no webhooks, so change detection means polling `updated_after` at 10 req/10 s per token | The C2 condition "first-class owner/role vs free text" now has a factual answer: **mixed**, per the table above. Whether that suffices for a thin org graph is a Council/Architect judgment (not made here) |
| **Park.** No SuiteQL Item sync / sole-spine build | — | No evidence contradicts parking. NetSuite remains the MBOM/commercial source under U2/U3, so a NetSuite read path is still needed later (inference) |

---

## Operating Model and Staffing (evidence)

*Scope of this section (as given by Chris, recorded as a design assumption — not a Researcher recommendation): product knowledge graph is **formal** and requires engineering expertise; company knowledge graph / company KB is **informal** and can be tended by a trained lower-level associate after initial standup. Evidence below is labeled by source type: **vendor marketing**, **analyst / commissioned study**, **consultant / practitioner guide**, **survey benchmark**, or **case-study anecdote**.*

### OM-1. Staffing and hours by option

Numbers below are **not additive claims about QCo**. Ranges come from different industries and company sizes. SMB thin-IT applicability is often **Unverified** when the source is large-enterprise.

#### Option 1 — Stardog / RDF+OWL product KG

| Phase | Published numbers | Source type | Confidence |
| --- | --- | --- | --- |
| **Standup (vendor framework)** | Step 1 "Understand why" **2–4 weeks**; Step 2 data inventory **2–4 weeks**; Step 3 design/build **~1 week per data source**; Step 4 reasoning/quality **1–2 weeks**; Step 5 publish **3 weeks**. Sum for a few sources ≈ **8–16+ weeks** of calendar (not FTE-weeks) | **Vendor marketing** — Stardog "How to Build an Enterprise Knowledge Graph" PDF ([HubSpot host](https://6618383.fs1.hubspotusercontent-na1.net/hubfs/6618383/Stardog_How%20to%20build%20an%20Enterprise%20Knowledge%20Graph.pdf)) | Medium (vendor schedule; no FTE × weeks disclosed) |
| **Standup (vendor claim)** | "Most teams can stand up an initial use case in a few weeks" | **Vendor marketing** — [What is a Knowledge Graph \| Stardog](https://www.stardog.com/knowledge-graph/) | Low–Medium (marketing) |
| **Standup + year-1 cost (third-party synthesis)** | In-house build: **9–14 months** to production; **$600K–$1.5M** year one; team **1 graph DB specialist + 1–2 data engineers + 1 ontology architect + 0.5 FTE data steward**. Managed Cloud path: **4–8 months**; **$300K–$800K** year one; team **1 graph developer + 1 data engineer + 0.5 FTE steward**; consulting for ontology **$80K–$200K**. Broader "EKG" readiness budget **$400K–$1.2M** year one with **2–3 FTE ongoing** | **Analyst-style blog estimates (not a named firm study)** — [Improvado EKG guide, updated Jul 2026](https://improvado.io/blog/enterprise-knowledge-graph). Author is a marketing platform; figures are **unattributed ranges** | Low–Medium (useful as an order-of-magnitude claim only; not verified against Stardog invoices) |
| **Ongoing ops** | Same Improvado post: **2–3 FTE ongoing** (graph specialist, data engineer, ontology steward) for a production EKG. Stardog Free self-host excludes backups/HA/LDAP ([Pricing](https://www.stardog.com/pricing/)); Enterprise Cloud provides daily backup/30-day retention ([Cloud](https://www.stardog.com/stardog-cloud/)). Self-managed Helm backup/restore is operator work ([Stardog support articles](https://support.stardog.com/support/solutions/articles/151000211575-how-to-do-server-backup-in-stardog-helm-charts)) | Mixed: blog estimate + first-party ops docs | Ongoing FTE: Low–Medium. Ops tasks: High |
| **Consulting partner typical?** | Stardog Enterprise includes Base Support, Customer Success, and access to Solutions Architects ([Pricing](https://www.stardog.com/pricing/)). Partner/PS exists ("Professional Services" on Cloud table). Forrester TEI was **commissioned by Stardog** (Dec 2021) and reports benefits (320% ROI, $9.86M over 3 years for a **composite customer**) but the **landing page does not publish implementation FTE or hours** ([TEI page](https://www.stardog.com/resources/forrester-tei/); [press](https://www.stardog.com/news/stardogs-enterprise-knowledge-graph-platform-delivers-up-to-320-percent-return-on-investment-according-to-independent-research-study/)). Full TEI PDF was **not obtained** in this research pass | Commissioned analyst study (marketing-hosted) | High that TEI exists and is commissioned; **Unverified** for project hours inside the PDF |
| **Skills gate** | "67% of abandoned EKG projects cited lack of internal graph expertise as the primary failure cause" (Improvado citing "a 2025 survey of enterprise data leaders" — **survey instrument and sample not linked**) | Third-party blog citing unnamed survey | Low |
| **Voicebox ongoing** | Vendor guidance: start with **5–10** stored NL→SPARQL examples; typically **≤50** for claimed ">90%" accuracy; examples must be public, SELECT-only, and reloaded if deleted ([Voicebox dev guide](https://docs.stardog.com/voicebox/voicebox-dev-guide/)) | First-party docs | High for the guidance; accuracy claim is vendor |

**Consulting vs internal (when sources separate them):** Improvado's managed-Cloud path lists **consulting for ontology design $80K–$200K** as a distinct line item from platform subscription and personnel. Stardog's own framework does not separate partner hours from customer hours. **No SMB-specific Stardog FTE study was found.**

#### Option 2 — Property-graph or thin-crosswalk first step

| Phase | Published numbers | Source type | Confidence |
| --- | --- | --- | --- |
| **Single-domain graph construction** | **6–12 weeks** calendar for one bounded domain (assess 1–2 wk, ontology 2–4, extract 1–3, resolve 2–4, validate 1–2); enterprise-wide ontology **2–3 months**. Prerequisite team: "**project lead at 20–40% FTE, domain experts on call**"; "at minimum a data or ML engineer … and a domain expert" | **Vendor / platform practice guide** — [Atlan, Knowledge Graph Construction for AI (updated Sep 2026)](https://atlan.com/know/ai-agent/knowledge-graph/knowledge-graph-construction-for-ai/) | Medium (vendor guide; Neo4j/Neptune-class, not Postgres-specific) |
| **Thin product evidence graph** | Prior QCo research pack cited Claro: start with **20–100 products / one family**; store optional (tables OK) ([Claro guide, Jul 2026](https://getclaro.ai/resources/guides/build-product-evidence-graph/); prior pack §1.2) | Vendor product guide | Medium |
| **Role pattern (consultant)** | Knowledge graph teams need: Product/Technical leads; **ontologists / information architects / taxonomists** with SMEs; **semantic data engineers** (SQL/Python + SPARQL/RDF/OWL). "A single person may fulfill multiple roles." No FTE counts | **Consulting firm practice note** — [Enterprise Knowledge, Aug 2022](https://enterprise-knowledge.com/what-team-do-you-need-for-successful-knowledge-graph-development/) | Medium for role list; FTE Unverified |
| **Property graph vs RDF TCO (same Improvado post)** | Property-graph / relational style TCO cited lower than RDF EKG ranges in that author's table (e.g. relational **$200K–$600K** 3-year vs EKG **$800K–$2.5M** for 100M entities). **Method and sample undisclosed** | Third-party blog estimates | Low |
| **Thin crosswalk (Postgres + MCP)** | No published FTE study found for "Postgres edge tables + read-only MCP" as a named pattern. Closest: SQL-trained staff sufficiency claims in the same Improvado comparison table; QCo Architect brief notes SQL/Cypher skills are common vs SPARQL/OWL | Inference from Architect brief + absence | **Unverified** for hours. Skills contrast: Medium |
| **Ongoing** | Atlan: entity resolution must be "an ongoing capability, not a one-time ETL step"; continuous population required. No hours/week figure | Vendor guide | Medium for the requirement; hours Unverified |

#### Option 3 — Company knowledge base / informal org-role graph

| Phase | Published numbers | Source type | Confidence |
| --- | --- | --- | --- |
| **KM program FTE (enterprise benchmark)** | Median KM program: **8 FTEs** directly supporting KM; **7.3 FTEs per $1B revenue**; **1 FTE per 444 employees** in the supported entity. Roles include content management specialists. Content processes: ~⅔ have clear business owners and lifecycle review | **Survey benchmark** — APQC 2022 KM Program Benchmarks ([PDF mirror](https://bibliotecas.sebrae.com.br/chronus/ARQUIVOS_CHRONUS/bds/bds.nsf/a09c52126a2de3eb324694529208e2e0/$File/31651.pdf); APQC measure definition [Open Standards](https://www.apqc.org/what-we-do/benchmarking/open-standards-benchmarking/measures/number-ftes-directly-support-business)) | High for the survey numbers; **SMB fit Unverified** (median programs are much larger than a manufacturing SMB thin track) |
| **Information-finding waste (context for KB value)** | Employees spend **8.2 hours/week** locating/recreating/resharing information (APQC claim on Content Management advisory page) | APQC / advisory | Medium |
| **Verification triage hours (vendor case studies)** | Faire (Guru): automated verification rules → **>5 hours/week** of manual verification work reclaimed; trust score 76%→96% ([case study write-up](https://www.casestudies.com/company/guru/case-study/faire-reclaims-5-hours-a-week-with-guru)). Entrata HR Knowledge Agent (Guru help): admin spent **2.5 hours/week** triaging HR questions; after geographic scoping + verification, verification 45%→98% and human responses dropped sharply ([Guru help](https://help.getguru.com/docs/customer-example-hr-knowledge-agent)) | **Vendor case-study anecdotes** | Medium (customer names given; methodology not independently audited). **Note:** Entrata example is **HR content** — directly relevant to QCo's **ban** on HR in the company lane |
| **Verification intervals (product capability)** | Guru: 30/60/90 days, 6 months, 1 year, or "Does not expire"; named verifier; auto-verify/unverify for Knowledge Agents ([Guru verification](https://www.getguru.com/features/verification); [help](https://help.getguru.com/docs/verifying-and-unverifying-cards)). Slite: Verified / Verification expired / Outdated / Verification requested; AI Ask/Agent deprioritizes or excludes outdated ([Slite tribal knowledge guide, May 2026](https://slite.com/learn/tribal-knowledge)) | First-party product docs | High |
| **Org-role graph (Strety cite-only)** | Strety Accountability Chart is **read-only via API**; polling at **10 req/10 s per token** ([OpenAPI](https://2.strety.com/api/docs/v1/openapi.yaml)). No published FTE for "sync Strety seats into a thin org graph" | First-party API | Hours Unverified |
| **Standup for thin company track** | No published "1–2 certified tribal sources + Strety cite" hour estimate found. Closest industry pattern: expert interview + verify + maintain loop (Slite five-step capture; no hours) | Practitioner / vendor content | Unverified |

### OM-2. Design assumption: formal product KG vs informal company KB tended by an associate

**Chris's statement (recorded, not evaluated):** Product KG is formal and needs engineering expertise. Company KB / informal org-role graph can be tended by a trained lower-level associate after initial standup.

#### OM-2.1 Evidence that a split between ontology ownership and content curation is common

| Pattern | What sources say | Source type | Confidence |
| --- | --- | --- | --- |
| **Ontology / semantic model ownership ≠ content curation** | Digetiers: "pairing knowledge engineers … with subject matter experts"; "Ontology ownership should sit with the people who actually understand the business domain, not only the platform engineers"; "A knowledge graph without ongoing ontology stewardship will eventually degrade" ([Digetiers](https://www.digetiers.com/en/insights/library/ontologies-knowledge-graphs-difference)) | Consultant / vendor insight | Medium |
| **Role separation on knowledge platforms** | K-AI platform roles: **Document Steward** = daily user (triage, KPIs, review triggers); **Producer** = SME with **occasional** contribution; Owner vs Authority distinguished so editorial and transverse governance don't collapse ([K-AI roles](https://k-ai.gitbook.io/knowledge-ai/the-k-ai-platform/roles.md)) | Product documentation | Medium (one product's model) |
| **KM Institute "knowledge steward"** | Stewards set taxonomy/metadata policy **and** curate repositories; often supported by departmental "knowledge champions" because KM teams are small ([KM Institute](https://www.kminstitute.org/blog/the-role-of-knowledge-stewards-in-safeguarding-organizational-intelligence)) | Practitioner association blog | Medium — note this **blends** governance and curation in one role |
| **APQC content ownership** | Two-thirds of surveyed KM programs have **clear business owners for content** and lifecycle review processes (2022 survey) | Survey benchmark | High |
| **Verified-card KB model** | Guru: every Card has a named verifier (person or group); Knowledge Agents can auto-verify/unverify; humans retain oversight via Quality Log. Viewers can comment; only Authors/Owners/Admins verify ([Guru help](https://help.getguru.com/docs/verifying-and-unverifying-cards)) | First-party | High — fits "trained associate as steward + SME as verifier" |
| **Product / formal graph side** | Atlan: ontology design needs business knowledge "not just engineering"; project lead 20–40% FTE. Enterprise Knowledge: ontologists + semantic engineers as a distinct team from product leads. OvalEdge (2026): "Most teams pair a subject matter expert with a data architect, then add a steward to maintain definitions after launch" ([OvalEdge](https://www.ovaledge.com/blog/ontology-in-ai)) | Vendor / consultant | Medium |

**Reading (descriptive):** Public sources commonly separate (a) **model/ontology engineering** from (b) **content verification/curation**, and they commonly assign (b) to business owners or dedicated stewards who are not necessarily senior engineers. That is **partially consistent** with Chris's formal/informal split. Sources do **not** generally claim that the company-side steward can be "lower-level" without SME verifiers behind them — the Steward/Producer split keeps SMEs in the loop for accuracy.

#### OM-2.2 Caveats where "associate tends company KB" fails

| Failure mode | Evidence | Relevance to QCo constraints |
| --- | --- | --- |
| **Certification / verification drift** | Undocumented-but-stale knowledge is "worse [than undocumented], because people act on it" (Slite, May 2026). Guru: unverified Cards remain searchable unless the agent is set to "Verified only"; default Card interval can be "Does not expire". Faire case: trust score had fallen to 76% under manual verification load before automation | Associate can run the **queue**, but intervals and "verified only" policy must be set; automation without rules recreates drift |
| **ACL mistakes / oversharing amplification** | Certified-Slices checklist: "Source ACLs reviewed for oversharing before indexing (search will amplify bad ACLs)". Prior semantic KG pack: shared-entity pivots and community summaries leak across ACL boundaries. Purview: Copilot runs as the user, so **overshared** SharePoint still grounds answers | An associate who can publish/ingest without an ACL review gate can enlarge blast radius |
| **HR / finance leakage** | Strety API includes **performance reviews** and shoutouts; seat responsibilities can contain P&L language; metrics can be `currency`. Guru Entrata case study is explicitly an **HR** Knowledge Agent — the pattern that works for HR is the pattern QCo's ban list forbids. Purview DLP: labeled items excluded from processing can **still appear in citations**; files uploaded into prompts are **not** DLP-scanned; Feb 2026 Copilot Chat bug summarized confidential email despite DLP ([The Register](https://www.theregister.com/software/2026/02/18/copilot-chat-bug-bypasses-dlp-on-confidential-email/4238087); [Purview docs](https://learn.microsoft.com/en-us/purview/dlp-microsoft365-copilot-location-learn-about)) | Associate-operated ingest needs **endpoint/field allowlists** and label hygiene; classifiers alone are insufficient |
| **Ontology / schema drift on the "informal" side** | If the company lane grows a **role/seat graph** (Strety seats are UUID-keyed), schema changes (responsibility reordering without IDs; person holding multiple seats; Rock owned by person not seat) require modeling judgment beyond card verification | Informal **document** curation ≠ informal **graph** curation. Evidence for associate-tended **graphs** specifically is weak (Unverified) |
| **Single-steward burnout** | APQC median is 8 KM FTEs (enterprise). Guru Faire: lean team crushed by manual verification until automation. Slite: "tribal knowledge paradox" — experts with most knowledge have least time to document | Thin IT + one associate is a concentration risk; automation and named SME verifiers appear as co-requirements in vendor plays |

### OM-3. Thin-IT evolution (descriptive observations from the evidence)

1. **Managed SaaS** (Stardog Cloud Enterprise; Guru/Slite-class KB; Strety as SoR) shifts burden from infrastructure FTE toward **stewardship and modeling FTE**. Self-managed Stardog adds K8s/backup FTE that thin MSP shops may not carry.
2. **Public cost/FTE figures for Stardog are either unpublished (list price) or large-enterprise composites (Forrester TEI benefits; Improvado ranges).** They do not answer "hours for a 20–100 SKU ISA-95 slice at an SMB."
3. **Company-lane associate model has precedent for content verification queues**, not for ontology authorship or for ACL/DLP engineering. Sources that succeed with associates still keep **SME verifiers** and often **automation rules**.
4. **Product-lane formal model has near-universal agreement that ontology/graph skills are scarce** and that projects without them stall (Improvado's 67% claim is Low confidence on methodology; Enterprise Knowledge / Atlan / Digetiers agree qualitatively at Medium).

---

## Part A — Product KG in Stardog on ISA-95 with PLM as SoR

### 1. Stardog capabilities relevant here

*(Summarizes and extends DF-3 with capability detail. Prefer DF-3 for cost/ops.)*

| Capability | What is documented | Caveats | Confidence |
| --- | --- | --- | --- |
| **RDF / OWL / SPARQL** | Full W3C standards support; SPARQL query engine; OWL reasoning | Reasoning schema graphs are shared across all reasoning users (named-graph permissions do not filter schema axioms) | High ([Features](https://www.stardog.com/features/); [Named Graph Security](https://docs.stardog.com/operating-stardog/security/named-graph-security)) |
| **Virtual graphs vs materialization** | Declarative mapping of RDBMS/CSV/NoSQL into RDF; query in situ via `GRAPH <virtual://…>` or import/copy into named graphs | `MINUS` not translated to SQL; duplicate solutions possible; R2RML named graphs unsupported; datatype comparison caveats | High ([Virtual Graphs](https://docs.stardog.com/virtual-graphs/)) |
| **SHACL** | Constraints for validation / ICV | — | High ([Features](https://www.stardog.com/features/)) |
| **Security** | Named-graph R/W; sensitive-property masking; virtual-graph as named graph; Entra OAuth; Databricks VG credential passthrough under Entra | **Named-graph security off by default**; unauthorized graphs **silently empty** (no error on GRAPH/FROM); SERVICE keyword errors instead | High ([VG security](https://docs.stardog.com/operating-stardog/security/virtual-graph-security); Architect brief §6.2) |
| **Voicebox (LLM)** | NL→SPARQL for Essentials/Enterprise; needs `voicebox.enabled`, stored examples, concise Designer models | API access "**currently limited and not enabled for most users**"; license `voicebox.count.limit` | High ([Voicebox dev guide](https://docs.stardog.com/voicebox/voicebox-dev-guide/)) |
| **MCP** | Official `stardog-cloud-mcp`: `voicebox_settings`, `voicebox_ask`, `voicebox_generate_query`; local stdio or remote HTTP (**beta**); auth via API key + optional SSO token | Free-form NL→query surface — not a governed business-verb tool layer | High ([GitHub README](https://raw.githubusercontent.com/stardog-union/stardog-cloud-mcp/main/README.md)) |
| **REST / BI** | HTTP API; BI/SQL endpoint on Enterprise Cloud | — | High ([Pricing](https://www.stardog.com/pricing/); Cloud plan table) |

### 2. ISA-95 as an ontology: renderings, coverage, commercial-data gaps

#### 2.1 Existing OWL/RDF (and adjacent) renderings

| Artifact | Form | Coverage | Confidence |
| --- | --- | --- | --- |
| **Digital Twin Consortium ManufacturingOntologies / ISA95** | **DTDL** (not OWL), plus ISA95-WoT variants | Equipment hierarchy and common object models aligned to ISA-95 | High ([GitHub](https://github.com/digitaltwinconsortium/ManufacturingOntologies/tree/main/Ontologies/ISA95)) |
| **HSU-AUT ODP DIN EN 62264-2** | OWL Ontology Design Pattern | **Equipment hierarchy only** | High ([GitHub](https://github.com/hsu-aut/IndustrialStandard-ODP-DINEN62264-2)) |
| **OPC UA ISA-95 information model** | OPC UA nodeset (OPC 10030) | Level model / equipment | High ([OPC 10030](https://reference.opcfoundation.org/specs/OPC-10030/4.2.3)) |
| **MESA B2MML** | **XSD** (V0701, 2023); royalty-free; untested JSON Schema variant exists | Machine-readable ISA-95 Part 2/4 shapes for exchange | High ([GitHub MESAInternational/B2MML-BatchML](https://github.com/MESAInternational/B2MML-BatchML); the mesa.org B2MML page returned 404 when fetched 2026-09-28) |
| **Research / survey ontologies** | Various OWL (MASON, ONTO-PDM, RAMI-aligned models, etc.) | Manufacturing resources/processes/products; **not** a standards-body ISA-95 OWL | Medium ([MDPI survey 2021](https://www.mdpi.com/2076-3417/11/11/5110); [arXiv:2608.24918](https://arxiv.org/abs/2608.24918)) |

**Fact:** There is **no canonical, standards-body OWL ontology for the full ISA-95 object model** (consistent with Architect brief §4.3). Implementers use B2MML names as alignment annotations or build thin namespaces.

#### 2.2 What ISA-95 covers well vs not for product marketing / commercial data

ISA-95 (ANSI/ISA-95 / IEC 62264) defines Levels 0–4 and focuses on the **Level 3↔4 interface** — equipment, material, personnel, process segments, operations definition/schedule/performance, quality test specs/results, inventory operations ([ISA overview](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard); Part 1 models/terminology 2025 edition listed).

| In scope (native or B2MML-shaped) | Out of scope / no native object (commercial & marketing) |
| --- | --- |
| Material Definition / Class / Lot | **Cutsheets / datasheets as marketing artifacts** |
| Equipment / Physical Asset hierarchy | **Pricebook / list price / discount rules** (price may appear in ERP Level 4, not as an ISA-95 Part 2 object) |
| Personnel (as MOM resource) | **Product families / sellable variants / packs as merchandising structure** |
| Process / Operations segments | **Photometrics** (IES LM-63-19 R2025; successor-oriented TM-33-23) ([IES LM-63](https://store.ies.org/product/lm-63-19-approved-method-ies-standard-file-format-for-the-electronic-transfer-of-photometric-data-and-related-information/)) |
| Operations Definition / Schedule / Performance | **Engineering Change Orders** (Architect brief: no ISA-95 object) |
| Test Specification / Result | **CPQ / configuration option rules** |

#### 2.3 Extending ISA-95 with product / PLM ontologies (evidence)

- Thin namespace + `rdfs:comment` / custom `b2mmlAlignment` annotations pointing at B2MML element names (Architect brief illustrative TriG).
- schema.org **`ProductGroup` + `hasVariant` + `variesBy`** for family/variant structure ([schema.org ProductGroup](https://www.schema.org/ProductGroup); [Google product-variant structured data](https://developers.google.com/search/docs/appearance/structured-data/product-variants)).
- GoodRelations / `gr:BusinessEntity` referenced from W3C ORG for commercial entities ([W3C ORG](https://www.w3.org/TR/vocab-org/)).
- ONTO-PDM and Product–Process–Resource (PPR) patterns for PLM/product lifecycle (MDPI survey; prior semantic pack §2.1).
- Cross-system identity via `owl:sameAs` / alias services; ISA-95 Part 7 Alias Service Model exists but **does not define who stewards aliases** ([ISA](https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard)).

### 3. PLM as SoR: integration patterns, identity keys, ERP crosswalk, affected prior items

#### 3.1 Common PLM→KG patterns (industry)

1. **Materialize certified extracts** into named graphs / edge tables by run timestamp (Architect brief TriG pattern: `g:fm-ebom-run-…`, `g:crosswalk`).
2. **Virtualize** PLM relational/API projections (Stardog VG) — only if a connector/SQL projection exists; FM is REST, so a landing store or custom connector is typical (**Unverified** whether a native Stardog FM connector exists).
3. **Digital-thread graph bridge** between revision-based PLM and effectivity-based ERP ([Beyond PLM](https://beyondplm.com/2024/01/14/3-benefits-of-using-graph-based-digital-thread-product-model-for-plm-erp-integrations/); OpenBOM-biased).
4. **Partner middleware** FM→ERP (vdR, KETIV, coolOrange) then KG reads results (Architect brief).

#### 3.2 Identity keys

| System | Typical hard key | Notes |
| --- | --- | --- |
| Fusion Manage | Item URN + revision/lifecycle; `dmsId` within workspace | High |
| NetSuite | Item `internalid` (immutable); SKU/`itemid` as display evidence | High (prior packs) |
| Crosswalk | Certified `(fmUrn, rev) ↔ netsuiteInternalId` with steward approval | Required by U1; cardinality may be **1:N** |

#### 3.3 Prior plan items affected by PLM-as-engineering-SoR (descriptive; no recommend)

From locked v1.3 → parked/changed in v2.0 ([Sprint 1 Work Plan](../memos/QCo-KG-Sprint-1-Work-Plan.md)):

| Prior item | Status under split | Why affected |
| --- | --- | --- |
| NetSuite `internalid` as **sole** product spine | Parked as sole engineering spine; retained as **commercial** identity | Engineering definition moves to PLM |
| SuiteQL Item extract as Sprint 1/2 build gate | Parked behind U1/U2 | Wrong first extract until field authority + crosswalk |
| Items View–only token | Parked behind U1/U2 | Avoids back-door sync before crosswalk |
| v0 store = LPG/Postgres; defer RDF | Contested: U4 now requires explicit Option B vs Stardog+ontology owner | Stardog path re-opened by Chris's ISA-95/Stardog decision |
| MCP `resolve_item` keyed only on NetSuite id | Needs two-lane / dual-key resolve (FM eng id + NS commercial id) | Identity model changed |
| BOM/`part_of` from NetSuite or certified BOM | Split: EBOM from PLM; MBOM from NetSuite (U3); never from drawings | Still valid ban on drawing inference |
| Dept landing zone "join on `internalid`" | Still valid for **commercial** answers; engineering artifacts may join on FM URN/rev first | Messaging change, not deletion |
| Architecture POV anti-pattern "RDF/OWL day one" | Tension with Stardog/ISA-95 path; Architect brief already framed Option B baseline with A as upgrade | Documented tension, not resolved here |

---

## Part B — Company knowledge base (tribal / unstructured, internal)

### 4. Architecture patterns for certified, AI-queryable tribal knowledge

Patterns observed (descriptive):

1. **Expert-capture loop:** identify holders → structured interviews / recorded walkthroughs → draft → **human verify** → maintain → retrieve (Slite five-step, May 2026).
2. **Verified buffer between raw sources and agents:** curated cards/pages with owner + review cadence; agents consume only verified (or verified+unlabeled) content (Guru agentic KB framing; [Guru blog](https://www.getguru.com/blog/agentic-knowledge-base-vs-enterprise-search-vs-rag)).
3. **AI ingestion from approved systems** with human certification gate (aligns to QCo Certified-Slices checklist: criticality, ownership, freshness, query-time ACL, citations, refuse).
4. **ACL-aware retrieval:** end-user / OBO identity at query time; no see-all service account for retrieve (checklist; Purview note that Copilot runs as the user).
5. **Org/accountability overlay:** link knowledge items to seats/owners (W3C ORG `Post`/`Role`/`Membership`; Strety seat UUIDs as cite keys — DF-1).

### 5. Tool landscape (verified-current only, with caveats)

*Only tools with 2025–2026 first-party or credible secondary confirmation. Not a ranking.*

| Category | Tool | Verified-current capability (2026) | Caveat |
| --- | --- | --- | --- |
| **M365-native** | SharePoint / Microsoft 365 Copilot | Permission-trimmed grounding; Purview DLP location for Copilot | Label/DLP gaps (§6); needs Copilot license |
| | SharePoint Knowledge Agent (preview Sep 2025) | Autofill metadata columns, organize library, summaries; PowerShell `KnowledgeAgentScope` | Preview; metadata-focused more than certified KB; audit gaps noted by practitioners ([Office 365 IT Pros](https://office365itpros.com/2025/09/25/sharepoint-knowledge-agent/)) |
| | SharePoint Premium / Syntex autofill | NLP prompts → library columns | Pay-as-you-go; not a verification workflow by itself ([Learn autofill](https://learn.microsoft.com/en-us/microsoft-365/documentprocessing/autofill-setup?view=o365-worldwide)) |
| **Verified KM** | **Guru** | Named verifier, intervals, auto-verify/unverify, Quality Log, MCP mentioned in vendor materials | Seat minimums / pricing in secondary blogs — treat as Medium |
| | **Slite** | Verification states; Ask / Agent retrieval respects outdated status | Vendor content |
| | **Confluence + Rovo** | Wiki + AI search; **no built-in forced re-verification cadence** comparable to Guru (secondary comparisons) | Medium |
| | **Tettra, Bloomfire** | Appear in 2026 KB roundups (Tettra Slack-first Q&A; Bloomfire multimedia search) | Medium — features not re-verified page-by-page in this pass |
| | **Notion AI** | Workspace AI; weaker formal verification vs Guru (secondary) | Medium |
| **Enterprise search** | **Glean** | Permissions-aware connectors across apps | Public list price unpublished; third parties cite ~100-seat minimums (Low–Medium) |
| **Build-on RAG** | Custom vector index over certified SharePoint/Teams slices | Fits certified-slices discipline | Engineering cost; ACL must be query-time |
| **EOS / accountability SoR** | **Strety** | Rocks/Issues/Scorecard/Chart via API + official MCP (DF-1) | Cite-only; HR-adjacent endpoints exist |

### 6. Exclusion controls for HR and financial data

| Control | What it does | Failure modes | Confidence |
| --- | --- | --- | --- |
| **Purview sensitivity labels + DLP for Copilot** | Block Copilot from **processing** labeled files/emails; block SITs in prompts; block external email grounding (preview); block web search when SITs present | Labeled items "**could be available in the citations**"; policy lag **up to 4 hours**; **cannot scan files uploaded into prompts**; SITs and labels cannot share one rule; mid-session label changes apply on next open; **Feb 2026 bug** summarized confidential Draft/Sent mail despite DLP (Microsoft: authorized user only, but violated intended Copilot exclusion) | High ([Purview](https://learn.microsoft.com/en-us/purview/dlp-microsoft365-copilot-location-learn-about); [Register](https://www.theregister.com/software/2026/02/18/copilot-chat-bug-bypasses-dlp-on-confidential-email/4238087)) |
| **Trainable classifiers** | Built-in **HR**, **Employee disciplinary action**, **Finance**, **Financial statement**, **Financial audit** (English) | Probabilistic; language/format limits; not a hard guarantee | High that they exist ([classifier definitions](https://learn.microsoft.com/en-us/purview/trainable-classifiers-definitions)) |
| **Connector / ingest allowlists** | Scope Graph connectors, Guru sources, custom RAG crawlers to approved libraries | Mis-scoped connector = silent inclusion | Medium (pattern) |
| **Strety endpoint deny** | Do not call `/reviews`; filter currency metrics and finance-worded responsibilities | Easy to miss fields inside allowed endpoints | High that risky endpoints exist (spec) |
| **Audit** | DLP alerts / simulation supported for Copilot location | SharePoint Knowledge Agent activity may lack rich audit (practitioner report) | Medium |

**Reliability summary:** Controls rely on **correct labeling, correct allowlists, and product-bug-free enforcement**. They reduce risk; they do not eliminate HR/finance leakage into AI answers.

### 7. Modeling organization roles and accountability

| Model | Core ideas | Fit notes |
| --- | --- | --- |
| **EOS Accountability Chart** | Seats (functions), 5–7 roles/outcomes per seat, single owner, GWC; **not** a reporting org chart ([EOS Worldwide](https://www.eosworldwide.com/accountability-chart)) | Strety implements seats as API `Role` with UUID; internal "roles" as ID-less `responsibilities[]` (DF-1) |
| **W3C ORG ontology** | `Organization`, `OrganizationalUnit`, `Post` (position independent of holder), `Role`, `Membership`, `reportsTo`, `headOf` ([W3C REC](https://www.w3.org/TR/vocab-org/)) | Explicitly notes it does **not** cover all accountability nuances; encourages extensions. `org:Post` ≈ EOS seat; `org:Role` ≈ abstract role type |
| **Tying knowledge to seats** | Proven pattern: `DocSlice` / Card `owner` → Person; optional `accountableSeat` → seat UUID | Rock ownership in Strety is **Person**, so seat link is derived or manual |

### 8. Strety integration (detail)

See **[DF-1](#df-1-strety-what-it-exposes-and-are-owner--seat--role-first-class)** for the exposure matrix and first-class object analysis. Additional URLs:

- Integrations overview: <https://strety.com/integrations/>
- Official MCP: <https://strety.com/integrations/strety-mcp/>
- OpenAPI YAML: <https://2.strety.com/api/docs/v1/openapi.yaml>
- OpenAPI HTML (login wall): <https://2.strety.com/api/docs/v1>
- Help Center integrations collection: <https://help.strety.com/en/collections/3600950-integrations>
- Rocks help: <https://help.strety.com/en/articles/8887868-strety-rocks-complete-guide>
- Community n8n node: <https://github.com/ajoshuasmith/n8n-nodes-strety>
- Community MCP (To-Dos only): <https://github.com/brentwpeterson/mcp-strety>

**Not verifiable from this pass:** official Zapier/Make apps; first-party bulk CSV export; webhooks outside the public OpenAPI; exact equality of `Person.seats[].id` and `Role.id`.

---

## Part C — Cross-cutting

### 9. Boundaries between the two systems

| Concern | Product lane | Company lane | Shared without merging |
| --- | --- | --- | --- |
| **Entities** | Product / MaterialDefinition, EBOM, MBOM, rev, ECO, DocSlices (cutsheet/pricebook) | SOP/process notes, Rocks, Issues, Scorecard cites, seats/roles | **People** (Entra UPN ↔ Strety Person UUID); **products referenced in SOPs/Rocks** by NetSuite `internalid` or FM URN **as citation**, not copied graph |
| **Identity keys** | FM URN+rev; NS `internalid`; certified crosswalk | Strety goal/issue/metric/role UUIDs; DocSlice IDs | Entra object id / UPN |
| **Cross-links** | Company answers that mention SKUs **cite** product MCP `resolve_item` | Product answers that mention process owners **cite** Strety seat/person | Hyperlink / ID reference only |
| **ACL** | Named-graph / RLS + OBO on product tools | Source ACL + OBO on company retrieve; Strety per-user OAuth | No shared "see-all" bot; sync jobs are a separate ingest identity problem (DF-4 G8) |
| **Ban list** | N/A (product data) | No HR, finance, NetSuite transactional fields, no product-graph copy | — |

### 10. Industry-observed phased rollout patterns (descriptive)

**Product / formal graph**

1. Bound one domain / one question (6–12 weeks single-domain guides; Claro 20–100 SKUs).
2. Identity/crosswalk before edges (ISA-95 Part 7 alias problem; U1).
3. Field-authority table before sync (U3).
4. Governed read API/MCP before NL→query (Architect brief; Text2Cypher error rates).
5. Expand families only after precision/ACL hold (prior Council POV D30/D90 pattern).

**Company / tribal KB**

1. Ban-list and ACL hygiene before crawl.
2. Certify critical few (checklist), not whole SharePoint.
3. Named verifiers + cadence (Guru/Slite).
4. Optional automation of verify/unverify with Quality Log oversight.
5. Org/accountability as **citation index** to Strety seats before any local org graph (C2).

### 11. Risks / failure modes

| Risk | Lane | Evidence hook |
| --- | --- | --- |
| Stardog without ontology owner / SPARQL skills | Product | Improvado abandonment claim; Architect trigger #3; U4 |
| Third item master via KG IRIs | Product | U1; Part 7 leaves steward out of scope |
| Dual MBOM / dual effectivity confusion | Product | FM mBOM Editor vs NetSuite Advanced BOM |
| Voicebox/MCP as see-all or ungoverned NL→SPARQL | Product | API-key MCP; limited API access; silent ACL drops |
| Company lane copies product facts | Cross | Ban list; cite-by-id |
| HR/finance via Strety reviews or currency metrics | Company | OpenAPI `/reviews`, `currency` format |
| Associate steward without SME verifiers / ACL gate | Company | OM-2.2 |
| Verification theater ("Does not expire" everywhere) | Company | Guru defaults; Slite stale-worse-than-missing |
| GraphRAG / uncertified SharePoint as SoT | Both | Prior research pack; checklist OUT list |
| Thin IT underestimates ongoing FTE | Both | OM-1 ranges vs Free-tier limits |

### 12. Sources

**Strety / EOS**
- <https://2.strety.com/api/docs/v1/openapi.yaml>
- <https://strety.com/integrations/>
- <https://strety.com/integrations/strety-mcp/>
- <https://help.strety.com/en/collections/3600950-integrations>
- <https://help.strety.com/en/articles/8887868-strety-rocks-complete-guide>
- <https://help.strety.com/en/articles/9579761-strety-adminland>
- <https://www.eosworldwide.com/accountability-chart>
- <https://github.com/ajoshuasmith/n8n-nodes-strety>
- <https://github.com/brentwpeterson/mcp-strety>

**Stardog**
- <https://www.stardog.com/pricing/>
- <https://www.stardog.com/stardog-cloud/>
- <https://www.stardog.com/deployment/>
- <https://www.stardog.com/features/>
- <https://www.stardog.com/resources/forrester-tei/>
- <https://docs.stardog.com/voicebox/voicebox-dev-guide/>
- <https://docs.stardog.com/virtual-graphs/>
- <https://docs.stardog.com/operating-stardog/security/named-graph-security>
- <https://docs.stardog.com/operating-stardog/security/virtual-graph-security>
- <https://github.com/stardog-union/stardog-cloud-mcp>
- <https://6618383.fs1.hubspotusercontent-na1.net/hubfs/6618383/Stardog_How%20to%20build%20an%20Enterprise%20Knowledge%20Graph.pdf>

**ISA-95 / ontologies / PLM**
- <https://www.isa.org/standards-and-publications/isa-standards/isa-95-standard>
- <https://github.com/digitaltwinconsortium/ManufacturingOntologies/tree/main/Ontologies/ISA95>
- <https://github.com/hsu-aut/IndustrialStandard-ODP-DINEN62264-2>
- <https://github.com/MESAInternational/B2MML-BatchML>
- <https://help.autodesk.com/cloudhelp/ENU/FLC-RestAPI/files/FLC_RestAPI_Advanced_Functionalities_item_details_endpoints_html.htm>
- <https://help.autodesk.com/cloudhelp/ENU/PLM-360-User/files/UG-BOMTAB-BOMCOMPARE.htm>
- <https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-INTERFACE.htm>
- <https://docs.oracle.com/en/cloud/saas/netsuite/ns-online-help/section_1517399803.html>
- <https://beyondplm.com/2024/01/14/3-benefits-of-using-graph-based-digital-thread-product-model-for-plm-erp-integrations/>
- <https://www.schema.org/ProductGroup>
- <https://www.w3.org/TR/vocab-org/>
- <https://arxiv.org/abs/2608.24918>
- <https://www.mdpi.com/2076-3417/11/11/5110>

**Company KB / exclusion / staffing**
- <https://learn.microsoft.com/en-us/purview/dlp-microsoft365-copilot-location-learn-about>
- <https://learn.microsoft.com/en-us/purview/trainable-classifiers-definitions>
- <https://www.theregister.com/software/2026/02/18/copilot-chat-bug-bypasses-dlp-on-confidential-email/4238087>
- <https://www.getguru.com/features/verification>
- <https://help.getguru.com/docs/verifying-and-unverifying-cards>
- <https://slite.com/learn/tribal-knowledge>
- <https://office365itpros.com/2025/09/25/sharepoint-knowledge-agent/>
- <https://atlan.com/know/ai-agent/knowledge-graph/knowledge-graph-construction-for-ai/>
- <https://enterprise-knowledge.com/what-team-do-you-need-for-successful-knowledge-graph-development/>
- <https://improvado.io/blog/enterprise-knowledge-graph>
- <https://www.digetiers.com/en/insights/library/ontologies-knowledge-graphs-difference>
- <https://www.apqc.org/what-we-do/benchmarking/open-standards-benchmarking/measures/number-ftes-directly-support-business>
- APQC 2022 KM Benchmarks PDF (mirror linked in OM-1)

**QCo prior art**
- Paths listed in the header of this pack.

---

## Appendix: Open verification items

1. Strety: Zapier/Make official apps; webhooks; bulk export; `Person.seats[].id` ≡ `Role.id`.
2. Stardog: Enterprise USD list price; whether "Essentials" Cloud tier is still sold; full Forrester TEI PDF FTE tables; native Fusion Manage connector.
3. Fusion Manage: whether QCo tenant has mBOM Editor; serial/lot effectivity; Relationships/Affected-Items REST coverage.
4. QCo CPQ system name and API (configuration-rules SoR under U3).
5. SMB-specific FTE hours for a 20–100 SKU ISA-95 slice (no public study found).
6. Glean seat minimum / price (third-party only).

---

*End of research pack. Evidence only — no QCo recommendations.*
