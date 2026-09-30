# QCo Product Lane — Field Authority Table (DRAFT)

**From:** Knowledge Graph Architect  
**For:** Chris (Director of AI & Technology); sign-off: QCo Analyst  
**Date:** 2026-09-28  
**Status:** v0.3 (2026-09-28) — Analyst-signed (v0.2 and v0.3); v0.3 folds in Chris's MBOM discovery answers. U2 SoR split CONFIRMED (PLM = engineering definition; NetSuite = commercial identity); **MBOM = open discovery**; crosswalk steward = named role, individual TBD. Confirmed crosswalk rows and any sync wait for a named steward with hours.
**Supersedes:** Informal field-authority references in Architecture POV / Fusion Manage ISA-95 brief  
**Evidence:** `research/QCo-KB-Split-Stardog-ISA95-and-Company-KB-Research-Pack.md` §DF-2, §DF-4; room lock 2026-09-28 (two lanes)

---

## 1. Purpose

One product entity in the knowledge layer. No third item master. Each attribute has exactly one **winner** system of record; the graph stores the value with **provenance** (source system, source key, as-of timestamp) and never silently merges conflicting values.

This table is the U3 unlock artifact. Analyst signs. U2 confirmed by Chris 2026-09-28. **Crosswalk Steward** is a named role (individual TBD); drafting proceeds, but no crosswalk row moves from `candidate` to `approved` until a person with weekly hours holds the role.

---

## 2. Identity keys (not attributes)

| Key | Role | Winner | Notes |
|---|---|---|---|
| NetSuite `internalid` | **Commercial product spine** — the single product entity key in the knowledge layer | NetSuite | Never invent a Stardog-native or FM-native product ID as the spine |
| Fusion Manage item / `dmsID` + revision | Engineering identity | Fusion Manage | Linked by **certified crosswalk** only; no fuzzy match, no auto `same_as` |
| Tulip record IDs (target state) | Execution identity | Tulip | Linked via work-order / item crosswalk when Tulip is in target state |
| SKU / itemid / display name | Evidence toward spine | — | Candidates only; never the hard key |

**Crosswalk cardinality:** Industry pattern allows **1 FM part → N NetSuite materials** (revision / effectivity / plant splits). Default assumption for QCo v0 is **1:1 within a named product family**; any 1:N must be an explicit certified row, not inferred. *(Evidence: Beyond PLM framing in research pack §DF-2.3; Medium confidence.)*

---

## 3. Field authority matrix

**Legend:** Winner = authoritative write source. Consumers may read; knowledge layer is read-only projection. Conflict rule = what the agent / MCP does when a non-winner differs.

### 3.1 Engineering definition (ISA-95 Material Definition / Product Definition)

| Attribute / concept | Winner | Consumers | Conflict rule |
|---|---|---|---|
| Item master (engineering description, classification, lifecycle state) | **Fusion Manage** | NetSuite (via release/integration), Stardog projection, agents | Prefer FM; flag NetSuite drift as `source_conflict` |
| Revision ID / revision letter | **Fusion Manage** | NetSuite BOM revision mapping, Stardog | Prefer FM; NetSuite BOM revision is a separate object (see §3.3) |
| **EBOM** (engineering bill of materials, structure, qty, units, find-number) | **Fusion Manage** | NetSuite (MBOM derivation), Stardog, agents | Prefer FM; never invent BOM edges from drawings or RAG |
| Engineering change order / affected items / effectivity intent | **Fusion Manage** | NetSuite (ECO release workflow), Stardog change-impact | Prefer FM; open ECO ≠ released MBOM |
| Attachments / certified cutsheets / drawings linked to item+rev | **Fusion Manage** (or certified DocSlice vault — TBD) | RAG, agents | DocSlice must cite FM item+rev; uncertified docs refuse |
| Where-used (engineering) | **Fusion Manage** | Stardog change-impact | Prefer FM |

*FM coverage for Material Definition fields is strong (REST v3 items/BOM/rev/change/effectivity). Research pack DF-2: High confidence on API; FM is fit as sole **engineering** SoR for these rows — not as sole product SoR overall.*

### 3.2 Commercial identity and ERP

| Attribute / concept | Winner | Consumers | Conflict rule |
|---|---|---|---|
| NetSuite `internalid` | **NetSuite** | All lanes | Immutable spine |
| SKU / itemid, display name (selling) | **NetSuite** | Sales, agents | FM names are evidence only |
| Price, cost, currency | **NetSuite** | Finance views only (ACL) | Never from FM or company KB; never into company lane |
| Active / inactive, tax, subsidiary, inventory-flag | **NetSuite** | Ops, agents | Prefer NetSuite |
| Purchasing / vendor item links | **NetSuite** | Planning, WMS | Prefer NetSuite |

### 3.3 Manufacturing BOM and effectivity — OPEN DISCOVERY (per Chris, 2026-09-28)

| Attribute / concept | Winner | Consumers | Conflict rule |
|---|---|---|---|
| **MBOM** (manufacturing / production BOM) | **DISCOVERY — not assigned.** Candidates: Fusion Manage (mBOM Editor) or NetSuite (Advanced BOM) | Planning/MRP, Tulip (target), WMS, Stardog | Until assigned: agents answer only **with citation + `non_authoritative` flag**; never pick whichever system is indexed; no `part_of` edges marked certified from either host |
| MBOM revision effectivity | **DISCOVERY** (follows MBOM host) | Planning, Stardog as-of queries | Same as above |
| MBOM owner (accountable editor) | **Product Development (likely)** — per Q4 | All | Owner ≠ host; confirm when host is chosen |
| Routing / operations | **DISCOVERY** — today NetSuite + tribal + paper travelers | Planning, execution | Traveler steps = product-lane `candidate` only; answers `non_authoritative`, low confidence |
| EBOM date-based revision effectivity ("as of" released child) | **Fusion Manage** | Engineering views, Stardog engineering as-of | Prefer FM dates for engineering queries |
| Serial / lot / unit effectivity | **Unassigned** | — | Open — FM serial/lot effectivity Unverified in Autodesk docs |

**Why discovery, not a default:** evidence shows two plausible MBOM hosts (FM mBOM Editor — research pack DF-2, Medium; NetSuite Advanced BOM with its own Effective Start/End dates — High). Vendor docs cannot settle which one QCo actually uses. Assigning NetSuite by default would bake in a guess.

**Discovery status (Chris answers, 2026-09-28):**

| # | Question | Answer | Effect |
|---|---|---|---|
| Q1 | FM mBOM Editor in use? | **FM greenlit; 90-day target for initial product families. mBOM Editor decision TBD.** | MBOM host stays open |
| Q2 | NetSuite Advanced BOM / dated revisions? | **Unknown** — Chris to advise | MBOM host stays open |
| Q3 | Where do routing / operations live? | **NetSuite + tribal knowledge; paper travelers on the floor** | Routing is undocumented in part; high confidence-risk for agents |
| Q4 | Who edits the manufacturing structure when an ECO lands? | **Product Development** | Under the decision rule, **Product Development is the likely MBOM owner and discovery lead** |

**Owner vs host:** Product Development is the likely MBOM *owner* (the accountable editor). The MBOM *host* (FM mBOM Editor vs NetSuite Advanced BOM) stays **open until Q1 and Q2 close**. Owner and host are separate rows so a system choice doesn't silently move accountability.

**Routing (Q3) rule:** Routing steps and operations are **product-lane** objects (ISA-95 Process Segments / Operations Definitions). Steps captured from paper travelers enter the product lane as `candidate` records only, pending the routing host decision. The Company KB captures only the **know-how around a traveler** (tips, workarounds, who to ask) and cites the product record by pointer; it never copies routing steps (C3). Until routing is captured and certified, routing answers return `non_authoritative` with low confidence.

**Timeline anchor:** crosswalk and field-authority rows for the initial families should be ready **before FM goes live for those families** (90-day target), so the knowledge layer reads from FM from day one.

**Decision rule once answered:** the MBOM host is the system where the manufacturing structure is actually maintained after an ECO, provided it can carry revision effectivity. If the answer is split (for example FM drafts it and NetSuite re-keys it), the MBOM SoR is the system Planning/MRP runs against, and the other becomes a working copy. **No dual-write, no silent preference for the newer-looking BOM.**

**As-of query rule (unchanged):** engineering "what was designed on date D?" goes to FM effectivity. Manufacturing "what do we build or plan on date D?" goes to the MBOM host once assigned. Questions spanning both return both answers with provenance, never a blended BOM.

### 3.4 Configuration rules (QCo-specific)

| Attribute / concept | Winner | Consumers | Conflict rule |
|---|---|---|---|
| Product configuration / CPQ rules (allowed options, constraints, cut rules) | **Product director (CPQ ownership)** — target SoR intended as Fusion Manage when config rules live there | Ops fulfillment, sales CPQ, agents | Prefer certified rule pack under product-director sign-off; tribal/head-rules are **uncertified** and refuse |
| Precision / shop-floor fulfillability of a configured SKU | **Product director** (accountable) | Ops | Not inventable by agents |

*Context (shared fact, 2026-09-27): 50+ CPQs across ~dozen families; Fusion Manage intended as primary SoR for config rules but still conceptual; product director between engineering and Ops is the natural cert owner. Until FM holds certified config, treat CPQ packs + director sign-off as the certified source — do not scrape heads into the graph.*

### 3.5 Planning, execution, quality (target-state stack; not Sprint 1)

| Attribute / concept | Winner | Consumers | Conflict rule |
|---|---|---|---|
| Planned orders / MRP suggestions | **NetSuite Planning** (Demand Planning / MRP) | Stardog as **run-scoped snapshot or live read** — not a certified slice | Volatile; regenerate each MRP run |
| Firmed work orders / purchase orders | **NetSuite** | Tulip, Stardog | Prefer NetSuite |
| Production execution / as-built / line quality checks | **Tulip** (if in target state) | NetSuite performance posts, Stardog | Prefer Tulip for execution facts |
| Quality **specification** (what to test) | **Open — Engineering (FM) vs Ops (Tulip)** | Agents | **Director decision #9** from ISA-95 brief; default lean: FM owns spec definition, Tulip owns results |
| Inventory on-hand / WMS moves | **NetSuite / NetSuite WMS** | Planning, agents (ACL) | Prefer NetSuite; intra-platform |
| Maintenance | **Unassigned** | — | Open (Ops + Finance) |
| Equipment master | **Split risk** — Tulip stations vs NetSuite fixed assets | — | Open; do not invent a third |

---

## 4. Operating rules (apply to every row)

1. **No silent merge.** If a non-winner differs from the winner, store both with provenance and surface `source_conflict`; agents must not average or pick arbitrarily.
2. **Certified crosswalk only.** FM ↔ NetSuite links are steward-approved rows (U1). Candidate links stay `candidate` until approved/rejected.
3. **Knowledge layer is read-only.** No standing write from graph or agents into FM, NetSuite, Tulip, or WMS.
4. **Company lane cites, never copies.** The Company KB runs independently of the Product KG (greenlit 2026-09-28). Later it may query the Product KG through the read-only AI query interface under the end user's identity, with citations. Its store holds pointers (`internalid`, FM item id) at most — never specs, BOMs or prices — and no HR/finance fields (Strety `/reviews`, shoutouts, currency metrics stay off the ingest allowlist — research pack DF-1 / C1).
5. **MCP refuse/flag codes:** `uncertified`, `stale`, `acl_deny`, `unknown_item`, plus `source_conflict` (non-winner differs from winner) and `non_authoritative` (answer from a row whose winner is still discovery/unassigned — returned with citation, never presented as certified).

---

## 5. U3 pressure from research (what this draft resolves vs leaves open)

| Pressure (DF-2 / DF-4) | How this draft handles it |
|---|---|
| FM can host MBOM (mBOM Editor) | MBOM marked **open discovery** (Chris ruling); four discovery questions + decision rule in §3.3; interim answers cited and flagged `non_authoritative` |
| Two effectivity schemes (FM date-rev vs NetSuite BOM Effective Start/End) | Engineering as-of stays FM; manufacturing as-of follows the MBOM host once assigned; never blend |
| 1 FM part → N NetSuite materials possible | Crosswalk allows certified 1:N; v0 default 1:1 per family |
| Serial/lot effectivity | Left **Unassigned** — Unverified in FM docs |
| Quality spec master FM vs Tulip | Left **Open** — Director decision |
| Config rules still conceptual in FM | Winner = product director / certified CPQ pack until FM holds them |

---

## 6. DF-3 note (store path — feeds U4, not U3)

Field authority is store-agnostic. The same winners apply whether the projection is Option B (thin crosswalk / property graph / Postgres) or Stardog/RDF. **U4 (store path)** remains: Option B baseline unless an ontology owner is named and funded. Stardog Cloud MCP's API-key model adds OBO friction for G8 — another reason not to unlock Stardog on U3 alone.

---

## 7. Sign-off

| Role | Action | Status |
|---|---|---|
| Knowledge Graph Architect | Draft | Done 2026-09-28 |
| QCo Analyst | Sign (agree / amend) | **v0.2 signed 2026-09-28; v0.3 re-signed 2026-09-28** — agree, no amendments. Owner/host split matches Council options memo v1.2 and Sprint 1 v2.1. |
| Chris | U2 SoR split | **Confirmed 2026-09-28** (MBOM carved out to discovery) |
| Chris | Name Crosswalk Steward (role defined, individual TBD) | Pending — blocks approved crosswalk rows, not drafting |
| MBOM discovery lead | Close Q1 (mBOM Editor) + Q2 (Advanced BOM); propose host | **Product Development (likely)**; within FM 90-day window |
| Product director | Confirm §3.4 config-rules winner | Pending |
| Ops / Finance | Maintenance + equipment master (later) | Not Sprint 1 |

**Amendments log:**
- v0.2 (2026-09-28): MBOM, MBOM effectivity and routing moved from NetSuite to open discovery per Chris; steward = role with individual TBD; company-lane query-through rule added; `non_authoritative` flag added.
- Analyst sign (2026-09-28): agreed as drafted; no amendments. One related amend sits on the MCP stub (live Strety cite filters), not on this table.
- v0.3 (2026-09-28): Chris discovery answers (Q1 FM greenlit, mBOM Editor TBD; Q2 unknown; Q3 NetSuite + tribal + paper travelers; Q4 Product Development). MBOM owner row added (Product Development, likely); host stays open; routing/traveler rule; FM 90-day anchor.
