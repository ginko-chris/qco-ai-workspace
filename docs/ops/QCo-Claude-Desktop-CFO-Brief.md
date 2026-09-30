# CFO Brief — Marketing Claude Desktop Pilot (Channel, Don’t Freeze)
**From:** Chris / QCo Analyst · **For:** CFO (then AI Council)  
**Date:** 2026-09-21 · **Time-box ask:** interim rules **this week**; staging yes-path in **weeks**, not a 90-day veto  
**Detail:** `QCo-Claude-Desktop-Attended-Pilot-CFO-Advise.md` · IT/OT controls folded

---

## One-liner (slide)

**“Channel, don’t freeze — agent profiles can’t reach NetSuite this week; Marketing keeps attended assist off ERP write; we deliver Pending Approval staging in weeks as the yes-path.”**

---

## Situation

SVP Marketing is already piloting **Claude Desktop attended** form-filling. A hard companywide stop without a yes-path loses political capital and drives shadow use. Waiting three months for a perfect API path is also a non-answer.

---

## Ask of CFO (today)

1. **Endorse interim policy** below and name **Director of AI & Technology** as owner of the **agent ↔ NetSuite / SoR boundary**.  
2. **Authorize MSP/IdP tickets this week** so agent-capable profiles **cannot reach NetSuite** (evidence: blocked access logs).  
3. **Prioritize NetSuite host** work for **create–Pending Approval–only** staging (API/middleware) so Marketing has a governed on-ramp in weeks.

---

## What we bless (time-boxed pilot)

| Allowed now | Condition |
| --- | --- |
| Claude Desktop **attended** assist | Human at the keyboard; **no** unattended/scheduled runs |
| **Non-NetSuite / non-SoR** forms & Marketing tools | Drafts the human still owns in those systems |
| Extract assist into a **designated staging queue** (sheet/app Chris names) | Agent has **no** NetSuite credentials; human moves data |

**Time-box:** revisit in **30 days** with Scorecard (below) — extend, tighten, or fold into staging path.

---

## Hard red lines (CFO-backed)

1. **No unattended** agent runs for form-fill / order entry.  
2. **No NetSuite UI writes** — including “just create a draft” in the browser. UI draft = write = banned.  
3. **No NetSuite SSO** on the Claude / computer-use profile — split profile; block NS URLs; no shared cookies/password autofill.  
4. **No standing prod NetSuite write/admin** credentials for any agent identity.  
5. **Human confirms** before any System-of-Record commit (NetSuite or other SoR).  
6. **Cua / Driver / similar:** same rules — no path that can drive a browser with NS SSO.

---

## “Do it here, not there” (landing zone)

Dept pilots keep moving, but product/order truth attaches to the **shared path** as it appears:

- **Here:** staging schema → NetSuite **Pending Approval** via scoped API → named human approve  
- **Not there:** shadow Claude/NS clicks, rival SKU lists, agent with inherited SSO  

Same landing-zone logic as the product KG: absorb evidence, don’t invent a second spine.

---

## This-week MSP/IdP teeth (evidence required)

1. Split OS/browser profile (agent vs daily NS)  
2. Block NetSuite host + NS SSO apps from agent profile  
3. Kill cookie / password-manager inheritance into agent profile  
4. Conditional Access / app assignment if IdP allows  
5. Logging: gen-AI destinations; blocked NS from agent profile; NS sessions from Marketing users  
6. Written interim rule email + MSP ticket  
7. Computer-use tools under same containment  

If MSP cannot block URLs in ~5 days → escalate as **IT control gap**, not Marketing’s fault.

---

## Scorecard (interim)

| Metric | Target |
| --- | --- |
| NS write / draft-create from agent-capable profile | **0** |
| Attended-only compliance (no scheduled agents) | **100%** of known pilot devices |
| Staging yes-path: draft-only token + Pending Approval | Named date within **weeks** |
| Wrong-valid field incidents attributed to pilot | Track; fail closed into exception queue |

---

## Council follow-on (after CFO)

Same brief; add Rock language: interim contain Marketing energy; Rock-assist = staging path that improves Ops/Marketing work — not “AI cuts seats.”

---

## Decision

- [ ] CFO endorses channel policy + Chris boundary ownership  
- [ ] MSP/IdP tickets opened this week  
- [ ] NetSuite host date for Pending Approval staging  
- [ ] SVP Marketing confirms pilot stays off NS write given dated yes-path  

