# QCo KG — Sprint 1 Work Plan (two weeks)
**Rock:** Product knowledge first — then company knowledge in a separate lane  
**Horizon:** Weeks 1–2 of the 90-day Rock slice (D30 foundation). **Product-lane anchor: Fusion Manage go-live for initial product families at D90 (~late Dec 2026, from 2026-09-28 greenlight)**  
**Dates:** Sprint 1 = next 10 working days (lock calendar with Chris)  
**Baseline:** `QCo-KG-Council-POV.md` · `QCo-KG-Architecture-POV.md` · `QCo-FusionManage-ISA95-Stardog-Architecture-Brief.md` · Certified-Slices checklist · `QCo-KB-Split-Council-Options-Memo.md` (v1.1) · `QCo-KG-Field-Authority-Table-Draft.md` (v0.2) · `QCo-KG-Two-Lane-MCP-Stub.md` · research pack `QCo-KB-Split-Stardog-ISA95-and-Company-KB-Research-Pack.md`  
**Owner (plan):** Agile Project Manager  
**Status:** **LOCKED v2.1** (2026-09-28) — v2.0 PASS from Analyst · Architect · Researcher; Chris rulings folded; **field-authority table v0.3 + MCP stub v0.2 SIGNED** (Analyst amend: C1 ban applies to live Strety cites, not only ingest)

---

## Re-baseline (2026-09-28) — Chris / room lock

**Two lanes, two stores, two SoRs — do not merge.**

| Lane | SoR | Store / pattern | Job |
| --- | --- | --- | --- |
| **Product** | PLM = engineering definition (EBOM, rev, change, Material Definition); **NetSuite = commercial identity** (`internalid`, price). **MBOM host = OPEN DISCOVERY** (S1-32) | Read-only projection: **Option B** (thin crosswalk + governed MCP) baseline; Stardog/ISA-95 only if ontology owner named + funded | Same part across certified engineering + commercial evidence |
| **Company** | Strety = SoR for Rocks / Issues / Scorecard; humans + approved systems for tribal → certified slices | Separate system: permissioned RAG over certified tribal slices + **thin org/role/Rock graph** (cites Strety seats/people by UUID) | NLP Q&A for internal use; roles & accountability. **Runs independently of Product KG.** Later: read-only, cited, OBO query into Product KG via `ask_product` — pointers only, never copies |

**Ban list (company lane):** no HR, no finance, no NetSuite transactional fields, no product-data copy (pointers `internalid` / FM item id at most), **plus Strety `/reviews`, shoutouts, currency-format metrics, finance-mentioning seat responsibilities**. Applies at **ingest and at live Strety cite time** (`get_rock` / `list_rocks` drop banned fields before return; `who_owns` must not treat finance-worded `responsibilities[]` as binding authority).

### Operating-model design assumption (Chris, 2026-09-28)

| Lane | Nature | Who tends it | Evidence / condition |
| --- | --- | --- | --- |
| **Product KG** | Formal; requires engineering expertise | Engineering + ontology stewardship. Research gives rough sizing only: formal Stardog graph ≈ multi-week standup, **~2–3 FTE ongoing or consulting partner** | Supports **Option B baseline** until a product data-model owner is named. Thin-IT model may need to evolve — staffing ask goes to Council |
| **Company KB** | Informal | **Trained lower-level associate after initial standup** | Holds **only with** SME verifiers + ingest allowlist + ACL gates (C1 exit). Fails on drift / ACL / HR-finance leakage without them |

### PARKED until unlock (product lane — do not build)

| Parked | Why |
| --- | --- |
| NetSuite Item-spine as **sole** hard identity for engineering | PLM owns engineering definition; NetSuite remains commercial identity |
| SuiteQL Item extract as Sprint 1/2 **build** gate | Wrong first extract until field-authority + crosswalk exist |
| NetSuite Items View–only token (even "commercial sync later") | Behind **U1/U2** — not a back door before crosswalk |
| Postgres/LPG as assumed product store default | Store path is a staffed decision (U4) |
| Sprint 2 sync/linker unlock on old Gates A–D alone | Needs named steward (person + hours) + signed field-authority table |
| Any certified MBOM `part_of` edges | MBOM host is open discovery (S1-32) |

### Still valid from v1.3 (carry forward)

- North Star: product knowledge first, build outwards — not companywide brain day one  
- OBO / no bot see-all for product answers (G8)  
- Candidates ≠ production edges; no embedding auto-`same_as`  
- No agent write to any SoR (incl. **Strety MCP write off agent path, year one**)  
- Refuse codes: `uncertified` · `stale` · `acl_deny` · `unknown_item` · **`source_conflict`** (new, v2.1); flag **`non_authoritative`** on MBOM answers until S1-32 closes  
- Certified-slices discipline; landing zone = "do it here, not there"  
- One family / ~20–100 SKUs for first product slice (quality gate, not ambition ceiling)  
- BOM/`part_of` only from SoR or certified BOM — never inferred from drawings; config rules never extracted from people's heads into the graph  

---

## Unlock decisions (Chris)

| # | Decision | Status (2026-09-28) |
| --- | --- | --- |
| 1 | Crosswalk steward (PLM eng id ↔ NetSuite `internalid`) + hours/week | **Role named, person TBD.** Drafting proceeds. **No crosswalk row approved and no sync** until a person with set hours holds the role |
| 2 | SoR split | **CONFIRMED:** PLM = engineering definition; NetSuite = commercial identity. **MBOM excluded** → discovery (#5) |
| 3 | Company track | **GREENLIT** — conceptual design, independent of Product KG |
| 4 | Store path: Stardog vs Option B | **Open.** Option B baseline, Stardog-ready (no rewrite); Stardog only with named + funded ontology owner |
| 5 | MBOM discovery owner + target date | **Owner: Product Development (likely).** Host open pending Q1 mBOM Editor + Q2 Advanced BOM (Chris to advise); not blocking drafting |

---

## Sprint 1 goal v2.1 (definition of done)

**Exit = decision-ready two-lane foundation — not a live consumer on either lane.**

### Product lane (must)

1. Field-authority table **SIGNED v0.2** — EBOM→PLM, config rules→product director until certified in FM, MBOM/effectivity/routing = discovery; eng vs mfg "as of" kept separate; `source_conflict` on disagreement  
2. Crosswalk steward role in RACI; person + hours requested (not a Sprint 1 blocker for drafting)  
3. **MBOM discovery owner + date named**; four discovery questions assigned (S1-32)  
4. First product family shortlist (~20–100) with commercial + engineering SME input  
5. Store path recorded: **Option B baseline** *or* Stardog with named ontology owner  
6. Two-lane MCP stub **SIGNED v0.2** (live Strety C1 filters on `get_rock` / `list_rocks` / `who_owns`)  
7. Operating-model staffing ask drafted (FTE vs partner hours) for Council  

### Company lane (must — greenlit)

8. Ban-list ingest policy written (expanded C1 list) **with exit criteria:** SME verifiers named, ingest allowlist, ACL gate  
9. 1–2 tribal source candidates named for certification (SOP / process notes). **Paper travelers:** capture only the know-how around them (tips, workarounds, who to ask) + pointer to product record; **never the routing steps** (C3)  
10. Strety cite sketch: Rocks/Issues/Scorecard + seats/people cited by UUID (Strety remains SoR); **citation-index only** until seat↔Person join is designed  
11. Thin org/role model sketch — roles & accountability, not people-as-content  
12. Associate tending runbook outline (queue, verify, publish, drift check)  

### Explicit out of Sprint 1

Live MCP consumer, Stardog production load, SuiteQL Item sync job, crosswalk row approval, certified MBOM edges, full catalog, HR/finance, agent write (incl. Strety MCP), GraphRAG as product/price truth, companywide brain.

---

## Sequencing v2.1 — timeline anchored to Fusion Manage D90

| Window | Product lane | Company lane |
| --- | --- | --- |
| **Sprint 1 (D0–D14)** | Field-authority v0.3 + MCP stub v0.2 signed ✔; S1-32 owner = Product Development (likely); family shortlist; store path note; staffing ask (S1-33) | C1 policy + exits; tribal/traveler-know-how cert candidates; Strety UUID cite sketch; associate runbook outline |
| **Sprint 2 (D15–D28)** | Crosswalk drafts for first family (steward role); Q1 mBOM Editor + Q2 Advanced BOM answers → MBOM host ruling | First certified tribal slice; one conversational consumer under OBO ACL |
| **Sprints 3–4 (D29–D56)** | **Steward person + hours needed here** to approve crosswalk rows; certified PLM engineering slices from FM as families load | Associate takes queues under SME verify |
| **Sprints 5–6 (D57–D90)** | Crosswalk rows approved + field authority live **before FM go-live**; knowledge layer reads FM from day one (no re-point) | Optional read-only `ask_product` link (C3) |

```
Gate 0 two-lane split (done) · U2 SoR split (done, MBOM excluded) · U3 field authority v0.3 signed
    → S1-32: Product Development owns discovery → Q1 + Q2 close → MBOM host ruling
    → Steward person + hours (by ~D30 to hold D90) → crosswalk rows approved
    → Store path (Option B baseline vs Stardog w/ ontology owner) + staffing ask
    → FM go-live D90: crosswalk + certified engineering slices ready
```

**Hard:** No crosswalk approval / sync until steward person + hours. No certified MBOM edges until host ruling. Routing answers low-confidence (`non_authoritative`) until traveler steps are captured as candidates and certified.  
**Parallel:** Company track runs independently; `ask_product` link is later and read-only.

---

## Backlog v2.1

| ID | Item | Size | Priority | Owner role | Depends on | Status |
| --- | --- | --- | --- | --- | --- | --- |
| S1-20 | Confirm SoR split in writing | S | Must | Chris | — | **Done** (MBOM excluded) |
| S1-21 | Crosswalk steward — role card drafted (`QCo-KG-Crosswalk-Steward-Role-Card.md`); person from Product Development + named backup; **name by ~2026-10-28 (D30)** | S | Must | Chris | S1-20 | Role card done; **person TBD** |
| S1-22 | Field-authority table — EBOM (PLM), config rules (product director), MBOM/effectivity/routing = discovery, `source_conflict`; no silent merge | M | Must | KG Architect (+ Analyst signs) | S1-20 | **v0.2 SIGNED** |
| S1-23 | First family shortlist (1–3) for Chris pick | S | Must | Commercial + engineering SME | — | Open |
| S1-24 | Record store path: Option B baseline vs Stardog if ontology owner named | S | Must | Chris + Architect | S1-33 | Open |
| S1-25 | Two-lane MCP stub (`ask_product` OBO, pointers only, no write tools; live Strety C1 filters) | M | Must | Architect + Analyst | S1-22 | **v0.2 SIGNED** (live-cite amend) |
| S1-26 | Landing-zone one-liner for Chris ("here not there" — both lanes) | S | Must | Analyst / Chris | S1-25 | Open |
| S1-27 | Re-baselined RACI | S | Must | Agile PM | S1-21 | v2.1 below; names TBD |
| S1-28 | Company ban-list ingest policy (expanded: Strety `/reviews`, shoutouts, currency metrics, finance seat text) + **exit criteria: SME verifiers, allowlist, ACL** | S | Must | Analyst + Researcher | Chris #3 ✔ | Open |
| S1-29 | Name 1–2 tribal cert candidates + SME verifiers | S | Must | Chris / process owners | Chris #3 ✔ | Open |
| S1-30 | Strety cite sketch — seats/people by UUID; seat roles are ID-less text, Rock owner = Person; citation-index until seat↔Person join designed | M | Must | Architect + Researcher | Research pack ✔ | Open |
| S1-31 | Draft Sprint 2 product + company backlogs | S | Must | Agile PM | S1-21–33 | Open |
| **S1-32** | **MBOM discovery** — owner vs host split. Answers (Chris 2026-09-28): Q1 FM greenlit, 90-day target; **mBOM Editor decision TBD**. Q2 NetSuite Advanced BOM / dated revs **UNKNOWN (Chris to advise)**. Q3 routing/ops = NetSuite + tribal; paper travelers on floor. Q4 **Product Development edits mfg structure after ECO** → likely MBOM **owner** + discovery lead. **Host stays open** until Q1 + Q2 close (neither blocks drafting). Traveler routing steps = product-lane `candidate` records, `non_authoritative` until certified | M | Must | **Product Development (likely owner/lead)** + PLM admin + NetSuite admin | Q1 mBOM Editor decision; Q2 Chris | Owner likely; **host open (Q1, Q2)** |
| **S1-33** | Operating-model staffing ask — product KG FTE vs partner project + ongoing hours; company KB associate + SME verifier hours | S | Must | Analyst + Agile PM | Research pack ✔ | Open |
| **S1-34** | Associate tending runbook outline (company KB) | S | Should | Agile PM + Analyst | S1-28 | Open |
| **S1-35** | Steward hours — timed review of one family in Sprint 2 × item count of first families = defensible hrs/week (no outside estimate) | S | Must (Sprint 2) | Steward (or PD delegate) + Analyst | S1-23 family pick | Planned |
| **S1-36** | **FM migration mapping owner** — confirm who owns the Fusion Manage migration load; requirement: load each item's NetSuite `internalid` as an FM item attribute so links arrive as pre-filled candidates | S | Must | **Owner UNKNOWN — Chris / FM implementation lead** | FM greenlight ✔ | **Open — needed before migration load design** |
| **S1-37** | Steward Scorecard measures: age of oldest candidate row, open candidates per family, open `source_conflict`, revisions effective without approved link (target 0) | S | Should | Agile PM + Analyst | Role card | Planned |
| ~~S1-05~~ | ~~SuiteQL Item schema / sole NetSuite spine extract~~ | — | **PARKED** | — | U1 person + U2 | — |
| ~~S1-03~~ | ~~Items View–only token~~ — commercial sync later still behind U1/U2 | — | **PARKED** | — | U1 person + U2 | — |

---

## Quality gates v2.1

| Gate | Pass criteria | Evidence |
| --- | --- | --- |
| **U1 Crosswalk** | Role named (drafting OK); **person + hours named before any crosswalk row approved or sync**; no third item master | RACI + Chris confirm |
| **U2 SoR split** | PLM = eng definition; NetSuite = commercial identity — **PASS 2026-09-28**; MBOM excluded | Chris ruling via CoS |
| **U3 Field authority** | Signed table: EBOM→PLM; config rules→product director until certified in FM; MBOM/effectivity/routing = discovery with `non_authoritative` flag + no certified MBOM `part_of`; eng vs mfg "as of" separate; `source_conflict` refuse | Signed table + S1-32 ruling |
| **U4 Store path** | Option B baseline or Stardog + named, funded ontology owner; staffing ask attached | Decision note + S1-33 |
| **G8 OBO** | End-user identity; no bot see-all; `ask_product` runs as user | Contract + freeze |
| **C1 Company ban** | Expanded ban list blocked at **ingest and live Strety cites**; **exit = SME verifiers named + allowlist + ACL gate** before associate tends | Policy + MCP stub v0.2 |
| **C2 Strety SoR** | Cited by UUID, not duplicated; **citation-index only** until seat↔Person join designed; no Strety MCP write (year one) | Sketch |
| **C3 Product link** | Company KB reaches product data only via read-only, cited, OBO `ask_product`; pointers only | MCP stub |
| **Park** | No SuiteQL Item sync / sole-spine build / crosswalk approval started | Plan status |

---

## Staffing / RACI v2.1 (fill names)

| Role | Who (fill) | R | A | C | I | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Director / Rock owner | Chris | | A | | | Decisions 1–5 |
| **Crosswalk steward** | _Product Development, person TBD_ (+ backup; product director = urgent-only escalation) | R | | | | Approves crosswalk rows only; no SoR write; name by ~D30 |
| **FM migration mapping owner** | _UNKNOWN_ | R | | | | Loads NetSuite `internalid` onto FM items (S1-36) |
| **MBOM discovery owner** | **Product Development (likely)** | R | | | | Edits mfg structure after ECO; host still open (Q1, Q2) |
| PLM / engineering SoR admin | _TBD_ | R | | | | Fusion Manage access; S1-32 Q1 |
| NetSuite commercial identity admin | _TBD_ | R | | | | `internalid`, price; S1-32 Q2 (MBOM not assigned) |
| Product director (config rules) | _TBD_ | | A | | | Config-rule authority until certified in FM |
| Ontology owner (if Stardog) | _TBD_ or none | R | | | | If blank → Option B; ~2–3 FTE or partner (rough) |
| Product family SMEs | _TBD_ | R | | | | Commercial + engineering |
| Company tribal cert owners / SME verifiers | _TBD_ | R | | | | C1 exit criterion |
| Company KB associate (post-standup) | _TBD_ | R | | | | Tends queues under SME verify + allowlist |
| Strety / EOS admin | _TBD_ | | | C | | Read path only |
| KG Architect | Knowledge Graph Architect | | | C | | Field authority, MCP stub, store path |
| QCo Analyst | QCo Analyst | | | C | | Council options memo v1.1; signs table + stub |
| Knowledge Systems Researcher | Knowledge Systems Researcher | | | C | | Evidence pack |
| Agile PM | Agile Project Manager | R | | | | Plan, deps, staffing chase |

---

## Risks / blockers

| Risk | Mitigation |
| --- | --- |
| Build on old NetSuite-only spine | Parked; U1 person + U2 required |
| Steward unnamed → crosswalk becomes third item master | No row approval until person + hours |
| Dual MBOM (FM mBOM Editor vs NetSuite) + two effectivity schemes | S1-32: owner (Product Development) separated from host; `non_authoritative` flag; no certified MBOM edges |
| Tribal routing + paper travelers → confident wrong agent answers | Steps = product-lane candidates only; routing answers low-confidence until certified; Company KB keeps know-how + pointer |
| Steward person not named by ~D30 → crosswalk not ready at FM go-live D90 | Timeline flags it; CoS tracks with Chris |
| FM migration loads without NetSuite `internalid` → steward hand-matches every item | S1-36: name mapping owner; make `internalid` attribute a migration requirement |
| New revision effective without approved link | Agents answer from last approved rev + `stale`; Scorecard target 0 |
| Stardog without ontology owner / thin IT can't run K8s | Option B baseline; staffing ask (S1-33) |
| Stardog named-graph security off by default (silent empty results) | Architect checklist item if Stardog chosen |
| Company lane copies product data | Ban + pointers only + `ask_product` read-only |
| Associate-tended KB drifts or leaks HR/finance | C1 exit: SME verifiers + allowlist + ACL |
| Strety MCP is read/write | Write off agent path year one |
| Ambition → one mega store | Two lanes locked; rejected in Council memo |

---

## Validation log

| Who | When | Result |
| --- | --- | --- |
| Knowledge Graph Architect | 2026-09-18 | PASS on v1.3 |
| QCo Analyst | 2026-09-18 | PASS on v1.3 |
| Knowledge Systems Researcher | 2026-09-18 | PASS on v1.3 |
| Room (Chris + experts + CoS) | 2026-09-28 | Two-lane split locked; PM re-baseline — v2.0 |
| QCo Analyst | 2026-09-28 | PASS on v2.0 — token behind U1/U2; C2 cite-only |
| Knowledge Graph Architect | 2026-09-28 | PASS on v2.0 — U3 named winners |
| Knowledge Systems Researcher | 2026-09-28 | PASS on v2.0 — research pack delivered |
| Chris (via CoS) | 2026-09-28 | Rulings: SoR split confirmed (MBOM → discovery); steward = role, person TBD; company KB greenlit; operating-model assumption |
| Agile PM | 2026-09-28 | **v2.1 LOCKED** — rulings + S1-32/33/34 + C1 exits + C3 folded |
| QCo Analyst | 2026-09-28 | **SIGNED** field-authority v0.2 (no amend) + MCP stub v0.2 (amend: C1 on live Strety cites) |
| Knowledge Graph Architect | 2026-09-28 | Amend accepted; MCP stub v0.2; field-authority v0.2 unchanged |
| Chief of Staff | 2026-09-28 | Taking MBOM discovery owner + date to Chris; steward hours + store path stay open |
| Chris (via CoS) | 2026-09-28 | MBOM Q1–Q4 answered; FM greenlit, 90-day target |
| Agile PM | 2026-09-28 | S1-32 + timeline updated to FM D90 anchor; traveler boundary folded (field authority v0.3, memo v1.2) |
| QCo Analyst | 2026-09-28 | Re-signed field authority v0.3 (no amend); concurs steward-person-by-~D30 is the constraint |
| Agile PM | 2026-09-28 | Steward role card folded (S1-21); added S1-35 pilot sizing, S1-36 FM migration mapping owner (unknown), S1-37 steward Scorecard |

---

*Living plan — update in place; do not fork. v1.3 and v2.0 preserved in the validation log; build path follows v2.1.*
