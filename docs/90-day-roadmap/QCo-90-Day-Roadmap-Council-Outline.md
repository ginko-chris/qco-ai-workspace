# QCo First 90 Days — Council Roadmap Brief Outline
**Owner:** QCo Analyst · **Audience:** AI Council · **Day one:** 2026-09-08 (Chris, Director of AI and Technology)  
**Method language:** EOS (Rocks / Issues / Scorecard / Level 10)  
**Companion threads:** layered SoT + certified slices; tier-by-job model mix; Knowledge Systems memos

---

## BLUF (for Council cover)

Chris’s first 90 days establish **control and shared vision**, not a finished AI program. Four pillars run in parallel with staggered “done” bars: capability landscape → scored Opportunity Map in EOS language → AI tooling baseline/threats → Business/Ops IT (MSP) control. AI adoption only rides **certified slices** and **right-sized models** — not tribal knowledge or max-capability everywhere.

---

## How EOS maps to the pillars

| EOS object | Role in 90 days | Primary pillars |
| --- | --- | --- |
| **Vision (V/TO niche / strategy)** | Capability map = org-neutral “what we must be able to do to win/operate”; Opportunity Map = shared picture of where AI/tech helps | 1, 2 |
| **Rocks** | ≤3–7 company/dept 90-day priorities pulled from top-scored opportunities + IT control needs — not a laundry list of pilots | 2 (select), 4 (if MSP Rock) |
| **Issues list** | Parking lot for pains, shadow AI, ACL debt, MSP gaps; IDS in L10 | All |
| **Scorecard** | Weekly numbers: AI spend/seats, SLA/ticket aging, certified-slice count, pilot override/citation rates; shadow-AI only if MSP already surfaces it | 3, 4 (+2 metrics) |
| **Level 10 / cadence** | Weekly traction; MSP ops review attached or sibling | 4 (+ Council monthly) |

**Rule:** Opportunity Map feeds the **Issues list** and **Rock candidates**. Only Rock-backed work gets budget/timeline commitments in the day-90 brief.

---

## Brief structure (Council packet)

1. Cover BLUF + 90-day success criteria  
2. Pillar 1 — Capability map (≤2 levels, org-neutral) + growth limits called out  
3. Pillar 2 — Opportunity Map (EOS-scored) + proposed Rocks  
4. Pillar 3 — Tooling/usage baseline + security/privacy + shadow-AI threats  
5. Pillar 4 — IT/MSP control posture (cadence, SLA/spend, change risk) — *IT/OT Expert input*  
6. Cross-cuts: SoT/certified slices; model tier-by-job; BotOps vs frontier lanes  
7. Day 30/60/90 done-definition + asks of Council  
8. Appendix: inventories, scorecard stub, open Issues

---

## Sequencing & dependencies

```
Week 1–2   Kickoff cadence · pull vendor dashboards · MSP baseline meeting · draft L1 capabilities
Week 3–4   L2 capabilities · pain→opportunity backlog · agreement security skim · Scorecard v0
Week 5–8   Score Opportunity Map · certify first slices if pilots named · MSP scorecard draft
Week 9–12  Lock proposed Rocks · Council brief · budget/timeline · defer MSP swap unless Rock
```

**Dependencies**
- Capability map (1) unlocks clean Opportunity Map (2) — don’t score AI toys that don’t map to a capability.  
- Tooling baseline (3) informs risk column on Opportunity Map and shadow-AI Issues.  
- IT/MSP (4) is **portfolio control**; do not block (1)–(3), but any production/OT touch waits on IT/OT Expert risk gate.  
- Certified-slice checklist gates any agent path from (2)/(3).

**Locked:** Company-wide EOS; AI Council in week one (listening + plan). Ongoing cadence TBD with Council; Chris runs weekly traction on pillars.

---

## What “done” looks like

### By day 30
- [ ] EOS artifact harvest: what % of L1 capabilities already exist in Vision/Accountability? Gap list + L1 draft from harvest (not blank-page map)  
- [ ] Vendor dashboard baseline (seats, spend, top products) + glaring contract flags listed  
- [ ] MSP meeting cadence set; SLA/ticket/spend snapshot #1 on Scorecard  
- [ ] MSP review agenda includes: existing shadow-AI / web-filter / CASB visibility (no greenfield monitor in roadmap)  
- [ ] Issues list seeded; Scorecard stub live  

### By day 60
- [ ] Capability map L1+L2 good enough to socialize with Council (harvest-first)  
- [ ] 2–3 in-flight Rocks selected for AI-assist candidates; each scored impact-on-Rock vs risk/cost + certified-slice plan  
- [ ] Security/privacy agreement review notes for in-use AI vendors  
- [ ] MSP vendor scorecard v0 (with IT/OT Expert) — **no MSP swap decision required**  
- [ ] If pilots named: certified-slice gate applied to first initiative corpus  

### By day 90
- [ ] Council-ready brief delivered  
- [ ] Council brief: AI assist on named current Rocks + certified-slice plans; any *new* Rock proposals only if clearly additive  
- [ ] Shared vision slide: capability limits to growth + opportunity portfolio  
- [ ] Explicit non-Rocks parked on Issues with revisit date  
- [ ] IT control narrative: how Chris’s portfolio sees MSP health; change path only if Rock-backed  

---

## Pillar notes (Analyst)

**1 Capability map** — Max 2 levels; verbs/outcomes (“fulfill order to promise,” “maintain safe OT”), not NetSuite modules or dept names. Call out bottlenecks to profitable growth.

**2 Opportunity Map** — Each card: capability link, greenfield vs in-flight, impact, risk/cost, SoT readiness (certified slice?), model tier (routine vs frontier), EOS status (Issue / Rock candidate). Prefer few Rocks.

**3 Tooling** — Dashboards > interviews. Flag personal-account / IP-leak paths as Issues with severity.

**4 IT** — Two vendors: corporate MSP + separate NetSuite host. Cadence on both; no swap unless Rock. Production risk is paper+NetSuite, not shopfloor OT. Shadow-AI ask sits on MSP agenda only.

---



---

## IT vendor cadence (Manufacturing IT/OT Expert)

**Facts (Chris, 2026-09-06):** Shopfloor OT is **minimal**. Production is largely **paper-based** and driven by **NetSuite**. NetSuite is hosted by a **third party distinct from the corporate MSP**. Chris meets both vendors.

Treat as **two control planes**, not one MSP review:

### A. Corporate MSP (standing ops review)
Monthly deep-dive; weekly ticket/SLA glance. First baseline week 1–2.
1. **SLA / tickets / spend** — aging P1–P3, reopen rate, after-hours, contract vs actual  
2. **Shadow AI / network visibility ask** *(not a greenfield program)* — proxy/DNS/CASB/web-filter for gen-AI destinations + personal accounts; if nothing → Issue only  
3. **SPOFs** — admin, backups, escalations; handoffs **to/from NetSuite host**  
4. **Scorecard inputs** — feed weekly Scorecard (v0 needs no new tooling)

### B. NetSuite host (separate firm — parallel cadence)
First meeting in Q1 alongside MSP baseline.
1. **Uptime / ticket / change** — who approves NetSuite changes that hit production workflows  
2. **Access & environments** — sandbox, integrations, who holds admin vs QCo  
3. **Paper → NetSuite gaps** — where floor still runs on paper; data latency / rekey risk (capability + Rock-assist fuel)  
4. **Spend & contract** — hosting, support tiers, customization backlog  
5. **Boundary with MSP** — who owns workstation/identity vs ERP app; no orphaned tickets between vendors

**OT note:** Keep “run production / recover from downtime” as capability outcomes, but **do not staff OT inventory or heavy OT change-freeze theater** — shopfloor OT is minimal; the real production system risk is **NetSuite + paper process**, not PLCs.

**D90 bar:** dual-vendor control narrative + scorecard v0 (MSP + NetSuite host health). No RFP / swap on either unless Rock-backed.

---

## Revisions from Manufacturing IT/OT Expert (2026-09-06)

Adopted:
1. **Sequencing:** Do not *score* Opportunity Map until tooling baseline (pillar 3 dashboards) exists — otherwise we score fiction. Opportunity *backlog* may start from capability map earlier.
2. **Shadow AI D30:** Define method + request MSP/proxy/DNS/CASB access; live network monitoring is not a day-one assumption.
3. **Capability L2 by D60:** Soften to “good enough to socialize with Council.” Production capabilities skew **paper + NetSuite**, not OT systems. Map “fulfill/run production” outcomes — no PLC/SCADA inventory.
4. **Vendor boundary Issue:** Seed early — MSP vs NetSuite host ownership (tickets, identity, change); OT change-freeze is low priority given minimal shopfloor OT.
5. **D90 IT done:** Control narrative + MSP scorecard — **not** an RFP. No MSP swap unless Rock-backed.



---

## Revisions from Chris / CoS (2026-09-06)

1. **Shadow AI:** Live network monitoring **out** of the 90-day roadmap. During MSP review, ask what proxy/DNS/CASB visibility already exists; Chris owns adding it to MSP review agenda. Do not staff a greenfield watch program in D30–90.
2. **Opportunity Map = EOS-native:** Do not introduce a parallel opportunity framework. First 90 days will have **established Rocks in flight**. Approach: identify **2–3 existing Rocks** as candidates for AI assistance; for each, layer **certified-slice KM** (critical few docs/metrics) so progress is in-stride, not disruptive.
3. **Capability map harvest first:** D30 discovery question — does existing EOS Vision / Accountability Chart / related artifacts already approximate ≥50% of a lightweight capability map? Prefer extract-and-gap-fill over greenfield mapping. Still keep max 2 levels and org-neutral language when filling gaps.
4. **Implication for scoring:** “Opportunity scoring” in this window means ranking AI-assist options **against current Rocks** (impact on Rock success criteria vs risk/cost/SoT readiness), after tooling baseline — not a free-floating AI idea portfolio.



---

## Ops reality (Chris, 2026-09-06)

- Shopfloor OT: **minimal**  
- Production: largely **paper-based**, driven by **NetSuite**  
- NetSuite hosting: **third party ≠ corporate MSP**; Chris meets NetSuite firm as well  
- Implication: dual-vendor IT control; capability map + Rock-assist should treat NetSuite/paper workflows as the production spine

## Timing lock (Chris / CoS, 2026-09-06)

- **EOS:** Company-wide, CEO + leadership.
- **AI Council:** First week after day one (week of 2026-09-08).
- **Expectation:** No finished roadmap day one. Chris wants a **plan in pocket**.
- **Week-one Council posture:** Listening + directional plan — four pillars, Rock-assist thesis, what he’ll learn by D30. **Not** a finished Opportunity Map or scored portfolio.
- **Speaking aid:** `/workspace/research/QCo-90-Day-Plan-In-Pocket.md` (one-pager).

## Clarifying questions (only if blocked)
*Resolved:* company EOS; Council week one. Remaining (optional): exact Council hour/agenda length; NetSuite firm contact / contract owner.

---

## Next drafting steps
1. ~~IT/OT Expert: pillar 4 + dual-vendor cadence (MSP + NetSuite host)~~ **done**  
2. Analyst: Scorecard stub + Rock AI-assist card template; fold paper→NetSuite into capability harvest prompts  
3. CoS: Chris review of plan-in-pocket before week-one Council

---

## Production / systems reality (Chris + IT/OT, 2026-09-06)

- **Shopfloor OT is minimal.** Do not center the 90-day plan on PLC/SCADA inventory or heavy OT change-freeze theater.
- **Production spine:** largely **paper-based** and **NetSuite-driven**.
- **NetSuite hosting** = third party **distinct from** corporate MSP → **two control planes** (MSP cadence + NetSuite host cadence).
- **Capability / Rock harvest bias:** prefer NetSuite + paper workflows (order-to-cash, inventory, scheduling, handoffs from paper→ERP) when nominating Rock-assist candidates and certified slices — not shopfloor control systems.

---

## Bifurcation (Chris / CoS, 2026-09-06): Roadmap framing vs landing criteria

**Week-one posture (invert):** Lead with *“What is QCo’s vision and which Rocks matter?”* — humility and curiosity. The pocket plan is Chris’s **private operating model**, not a splashy assertion of “Chris’s vision.”

### (A) Roadmap framing — socialize carefully
What Council/leadership can hear without overclaiming day one:

| Element | Notes |
| --- | --- |
| Four pillars as a **learning agenda** | Capability harvest, Rock-assist, tooling baseline, dual vendor control — framed as how Chris will learn and contribute, not finished recommendations |
| EOS-native Rock-assist (2–3) + certified slices | In-stride progress thesis |
| Tier-by-job models; anti-surrender coaching norms | Design rules, not vendor picks |
| Explicit non-goals | No company-wide KM program, no MSP RFP, no AI-everywhere |
| D30/60/90 “what you’ll see” | Learning milestones + one visible supervised assist if bandwidth allows |

**Challenges tagged (A):** SoT/certified-slice gates on Rock assists (#5 partial); acceptable-use/IP as a Council conversation (#6); Rock-assist Scorecard metrics (#7); coaching norms (#8); NetSuite field trustworthiness as discovery (#9); D30 visible assist vs pure diagnostic (#10); change-capacity / what we defer if snapped (#3 as shared non-goals).

### (B) Chris landing criteria — ruthless personal checklist
Not all for the speak sheet. These make Chris *effective* in the seat:

| # | Criterion | Why |
| --- | --- | --- |
| B1 | **Relationship map** (Rock owners, NetSuite power users, MSP + NetSuite-host escalation humans) | Rock-assist and dual cadence stall without people |
| B2 | **Week-1 access** — read/admin as needed to MSP tickets/spend + NetSuite host tickets/change queue | Scorecard is fiction without sightlines |
| B3 | **CFO Scorecard contract** — which weekly numbers Chris shows CFO vs Council | Reports to CFO; avoid “busy but unreadable” |
| B4 | **MSP ↔ NetSuite RACI** (identity/workstation vs ERP app vs integrations) | Orphaned tickets between two firms |
| B5 | **NetSuite-down playbook ask** (RTO/RPO, who declares, how paper floor runs) | Outage = production rekey chaos |
| B6 | **Spend split** — MSP + NetSuite host $ visible on dual Scorecard | Half-portfolio control otherwise |
| B7 | **Change collision calendar** — MSP patches vs NetSuite customizations | Self-inflicted outages |
| B8 | **Certified-slice owners named** per Rock-assist | Invalidation/review doesn’t staff itself |
| B9 | **Personal defer list** if bandwidth snaps | Protect landing; don’t silently drop B-items |

**Challenges tagged (B):** relationship map (#1); CFO Scorecard (#2); day-one access (#11); RACI (#12); outage playbook (#13); spend split (#14); change collision (#15); slice owners (#5); personal capacity defer (#3).

**Speak sheet rule:** Week-one room voice = mostly **(A)** + questions. **(B)** stays Chris’s private landing checklist (and CoS accountability).

