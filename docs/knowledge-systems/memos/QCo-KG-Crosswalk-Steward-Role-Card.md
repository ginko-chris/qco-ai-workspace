# QCo Crosswalk Steward: Role Card (Draft v0.1)

**Owner:** Knowledge Graph Architect (data design) with Agile PM (hours and sequencing)
**Status:** v0.2 (2026-09-28). Adopted in plan as S1-21, with hours in S1-35 and measures in S1-37. Draft for the room to review. The individual is still TBD, and no crosswalk row can be approved until a named person with set hours holds the role (U1).
**Related documents:** `QCo-KG-Field-Authority-Table-Draft.md` (v0.3), `QCo-KG-Two-Lane-MCP-Stub.md` (v0.2), `QCo-KG-Sprint-1-Work-Plan.md` (v2.1), `QCo-KB-Split-Council-Options-Memo.md` (v1.2)

## 1. Purpose

The steward is the one human who confirms that a given Fusion Manage item at a given revision is the same product as a given NetSuite item, and who signs off on that link. Every product answer an agent gives depends on that link. The approved crosswalk is the only place in the product lane where a person writes directly. Agents never write to it.

## 2. What a crosswalk row is (the thing the steward approves)

| Field | Meaning |
|---|---|
| `fm_item_id` + `fm_revision` (or an effectivity range) | Engineering side of the link (Fusion Manage). |
| `ns_internalid` | Commercial side of the link (NetSuite). This is the sole hard identity key. |
| `link_type` | `exact` (one to one), `variant` / `cut_length` / `packaging` (one FM item to many NetSuite items), or `superseded_by`. |
| `status` | `candidate`, then `approved`, `rejected` or `retired`. Rows are never deleted. A retired row gets an end date. |
| `proposed_by` | The script, the AI extraction pass, the migration load, or a person. |
| `approved_by`, `approved_at`, `evidence` | Who signed off, when, and what they checked (drawing number, cutsheet, NetSuite item record). |

## 3. Decision rights

**The steward decides:**
- Whether to approve, reject, split or retire a proposed link.
- How to handle one-to-many cases (variants, cut lengths, packaging).
- Whether a link rolls forward to a new revision, or is retired, when an ECO releases.
- Where each `source_conflict` goes. The steward routes it to the field owner named in the field-authority table.

**The steward does not:**
- Edit Fusion Manage or NetSuite records. The steward has no write access to either system of record.
- Design the data model or ontology. That belongs to the Architect or an ontology owner.
- Own configuration rules. Those stay with the product director.
- Decide where the MBOM is hosted.
- Resolve conflicts by picking a winner. The field owner fixes the source data.

## 4. What agents do with each state (from the MCP stub)

| Crosswalk state | What the agent returns |
|---|---|
| `approved`, and the revision is currently effective | A normal answer with a citation. |
| `candidate` | `non_authoritative` with low confidence. |
| `retired`, or the revision has been superseded | `stale`, with a pointer to the successor if one exists. |
| Fusion Manage and NetSuite disagree on an authority field | `source_conflict`, and the item goes into the steward's queue. |
| No row at all | `unknown_item`. |

## 5. Triggers and standing rule

The work is triggered by an FM family go-live (a bulk batch), an ECO release or revision roll, a new SKU set up in NetSuite, an item retirement, or a new entry in the `source_conflict` queue.

**Standing rule:** the link for a new revision must be approved before that revision's effective date. If it isn't, agents keep answering from the prior approved revision and flag the answer `stale`. They never answer from an unapproved link.

## 6. Keeping the go-live peak manageable (architect recommendation)

Most of the steward's effort lands at each family's Fusion Manage go-live, because that's when FM item IDs are first created. To shrink that peak:
- **Carry the NetSuite `internalid` onto FM items as an attribute during the FM data migration.** Links the migration load already carries come in as pre-filled `candidate` rows that the steward confirms in bulk. The steward then spends real time only on exceptions and one-to-many cases.
- Before go-live, draft crosswalk rows keyed on NetSuite `internalid` plus the drawing or part number. Bind them to FM IDs at load time.
- Have the product director spot-check a sample of each bulk go-live batch. This is a light second check, not a review of every row.

This is a request to the FM implementation team. No one owns the migration mapping yet (plan item S1-36). Chris's list now includes who owns the FM migration load, and this has to be settled before migration design begins. Until it is, candidate rows are drafted from the NetSuite side, so the steward is not blocked.

## 7. Hours and backup

- **Hours:** set these from the timed single-family pilot in sprint 2, as the Analyst recommended. Multiply the pilot's review rate by the item counts of the first families. External FTE estimates are not a budget basis.
- **Profile:** someone in Product Development who knows the engineering item structure and NetSuite item setup, for example a product data or NPI coordinator, or the R&D engineer who does first-pass rule extraction.
- **Not a fit:** the MSP, the Company KB associate, or IT generalists.
- **Backup:** a named backup can approve links when the primary is away. The product director is the escalation point for urgent cases only.
- **Date risk:** a named person with set hours by about day 30. Without that, rows can't be approved before the day-90 FM go-live, and agents would launch on `non_authoritative` links.

## 8. Health measures (Scorecard candidates)

- Age of the oldest `candidate` row.
- Count of open `candidate` rows per family.
- Count of open `source_conflict` items.
- Number of revisions that became effective without an approved link (target: zero).

## Amendments log
- v0.2 (2026-09-28): Linked to plan items S1-21, S1-35, S1-36 and S1-37. The steward's name is due around Oct 28 (day 30).
- v0.1 (2026-09-28): Initial draft following Chris's request to expand on the steward role. Built on the Researcher's evidence and the Analyst's recommendation.
