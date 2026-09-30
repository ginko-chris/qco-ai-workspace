# QCo Two-Lane MCP Contract — Stub (v0.2)

**From:** Knowledge Graph Architect · **For:** Chris; sign-off: QCo Analyst · **Date:** 2026-09-28
**Builds on:** MCP v0 contract (Architecture POV / Council POV), `QCo-KG-Field-Authority-Table-Draft.md` v0.2, `QCo-KB-Split-Council-Options-Memo.md` v1.1
**Status:** v0.2 — Analyst-signed 2026-09-28 (one amend: C1 ban list applies to live Strety cites, not only ingest). Conceptual contract only; no build unlocked by this stub.

## 1. Shape

Two separate MCP servers, two stores, two systems of record. No shared store and no shared write path.

| | Product lane | Company lane |
|---|---|---|
| Server | `qco-product` | `qco-company` |
| Store | Option B projection (thin crosswalk), Stardog-ready | Certified tribal slices + thin org/role index |
| SoR | FM (engineering definition), NetSuite (commercial identity); MBOM = discovery | Strety (Rocks, Issues, Scorecard, seats/people); certified docs |
| Tools | Read-only | Read-only |
| Identity | End-user on-behalf-of (G8); no see-all service identity for retrieval | Same |

## 2. Product lane tools (`qco-product`)

| Tool | Returns | Notes |
|---|---|---|
| `resolve_item(query)` | Candidate `internalid`s with evidence; FM item/rev via certified crosswalk only | `unknown_item` if nothing certified |
| `get_product_context(internalid, as_of?, view?)` | Engineering definition (FM), commercial attributes (NetSuite), linked certified DocSlices; per-attribute provenance | `view` is filtered by ACL, not chosen freely by the caller |
| `get_bom(internalid, kind=EBOM|MBOM, as_of?)` | EBOM from FM. MBOM is returned **only with `non_authoritative` and a citation** until discovery closes | Never a blended BOM |
| `change_impact(eco_id)` | Affected items and where-used (FM); links to NetSuite objects when crosswalked | Engineering-side only until MBOM host assigned |
| `list_doc_slices(internalid, type?)` | Certified slices with owner, version and cert date | Uncertified slices are excluded |

## 3. Company lane tools (`qco-company`)

| Tool | Returns | Notes |
|---|---|---|
| `search_knowledge(query)` | Certified tribal slices with citations, owner and verifier | Ingest allowlist enforced (C1 ban list) |
| `who_owns(topic | rock | process)` | Seat/Person (Strety UUIDs) with a cite to Strety | C2 = citation index only until the seat/Person join is designed. Do not return finance-worded `responsibilities[]` text as binding authority |
| `get_rock(rock_id)` / `list_rocks(owner?)` | Rock / Issue / Scorecard summaries **cited to Strety** | Read-only; no Strety MCP write on the agent path in year one. **Live cite must still apply C1 filters:** drop `/reviews`, shoutouts, currency-format Scorecard metrics, and finance-worded seat responsibilities — ban is not ingest-only |
| `ask_product(query)` | Pass-through to `qco-product` under the **same user identity**; returns cited results | Company store keeps pointers only (`internalid`, FM item id), never specs, BOMs or prices |

## 4. Shared refuse / flag codes

`uncertified` · `stale` · `acl_deny` · `unknown_item` · `source_conflict` (non-winner disagrees with winner) · `non_authoritative` (winner still discovery/unassigned; cited, never presented as certified) · `banned_content` (company lane: HR/finance or C1-banned source requested)

## 5. Hard rules

1. No write tools on either server. No standing write into FM, NetSuite, WMS, Tulip or Strety.
2. Every response carries citations and an as-of timestamp. Audit log per call: user, agent, tool, query, sources touched, freshness.
3. The company lane never caches product answers as company knowledge.
4. Cross-lane calls do not escalate privilege: `ask_product` runs as the user, not as the company server.
5. Stardog, if adopted (U4), sits behind `qco-product` unchanged. Vendor NL-to-SPARQL MCP is for experts only and is not on the agent path (its API-key model conflicts with G8).

6. **C1 ban list applies to live Strety cites**, not only to company-store ingest. Tools that read Strety at call time must filter the same banned fields before returning. Ingest allowlist alone is not enough.

## 6. Open

MBOM host (discovery) · Crosswalk Steward individual · U4 store path · C2 seat/Person join design · AI framework choice against this contract

## 7. Sign-off

| Role | Action | Status |
|---|---|---|
| Knowledge Graph Architect | Draft | Done 2026-09-28 |
| QCo Analyst | Sign (agree / amend) | **Signed 2026-09-28** with amend: live Strety cite filters (§3 notes + rule 6) |

## 8. Amendments
- v0.2 (2026-09-28): Analyst amend — C1 ban list on live Strety cites; `who_owns` must not treat finance-worded responsibilities as binding. Signed.
