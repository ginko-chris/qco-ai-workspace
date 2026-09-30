# Advise — Marketing Claude Desktop Attended Form-Fill (CFO / Council)
**From:** QCo Analyst · **For:** Chris (Director of AI & Technology)  
**Date:** 2026-09-21 · **Status:** Political + control advise — not a ban memo  
**Ground truth:** SVP Marketing already piloting Claude Desktop **attended** form-filling. Chris lacks day-one clout/time for a hard stop without CFO backing. Position cannot be full-stop NO or “wait 3 months for API.”

**CFO one-pager:** `QCo-Claude-Desktop-CFO-Brief.md`  
**Companions:** `QCo-Staged-Order-Entry-Agent-Options.md` · `QCo-RLCD-CUA-Staged-Order-Advise.md` · eval-hygiene memo

---

## BLUF (what to argue)

**Channel, don’t freeze.** Ask the CFO to back an **interim attended policy with hard boundaries this week**, plus a **fast “yes path”** (Pending Approval / staging API) — not a companywide ban on Claude Desktop.

| Frame that fails | Frame that wins with CFO |
| --- | --- |
| “Marketing must stop until we finish architecture” | “Marketing can keep **attended** assist **outside NetSuite write** while we put teeth on IdP/MSP and stand up staging” |
| “AI form-fill is unsafe forever” | “**Unattended ERP write** and **shared SSO with agents** are the safety problem — not a human watching a draft in a marketing tool” |
| “Wait for perfect API (90 days)” | “**Two clocks:** (A) interim contain Marketing’s blast radius **this week**; (B) deliver draft-only NetSuite path in weeks, not quarters” |

**Ask of CFO:** Endorse Chris as owner of **agent ↔ NetSuite boundary**; Marketing pilot continues under written interim rules; MSP/IdP execute containment; Ops/NetSuite host prioritize draft-only token so Marketing (and others) have a governed on-ramp.

---

## Safety rationale (honest, not maximal)

**What is actually risky (lead with these):**
1. **Wrong-but-valid data** — schema-safe picks wrong SKU/customer/qty; attended human still rubber-stamps under time pressure (cognitive surrender).  
2. **Credential inheritance** — Claude Desktop / computer-use on a profile that already has NetSuite SSO can act as that user (writes look like the human).  
3. **Audit gap** — UI clicks don’t give clean Message-ID ↔ SO externalId trail the way staging API does.  
4. **Precedent** — if Marketing can “just click NetSuite,” every dept will; landing-zone governance dies.

**What is weaker as a full-stop argument (don’t overclaim):**
- “Attended form-fill in **non-ERP** marketing tools” is lower blast radius.  
- Vendor brand (Claude vs Jev vs Cua) is secondary to **where** the agent can write.

**IT/OT interim teeth — ask MSP/IdP to execute this week (written ticket + evidence):**

1. **Split profile** — Claude Desktop (and any computer-use) under a **dedicated** OS/browser profile; daily NetSuite on a separate profile. No “same Chrome, agent + NS tabs.”
2. **Block NS from the agent profile** — URL/proxy/DNS deny for NetSuite host domains + IdP apps that mint NS SSO from that profile (Conditional Access / app assignment if Azure/Okta; else proxy category). Goal: agent profile **cannot open NS**.
3. **Kill cookie inheritance** — clear/block NS and IdP cookies on the agent profile; disable password-manager autofill of NS into that profile; no shared browser sync of that profile to the NS identity.
4. **IdP sign-in policy** — if available: Conditional Access block NS app from device/group tagged “AI-assist / Marketing Claude pilot”; phish-resistant MFA on NS from the human’s normal profile only.
5. **Logging** — gen-AI destinations from Marketing endpoints; failed/blocked NS access from agent profile; new NS sessions from Marketing users (Scorecard: **NS write from agent path = 0**).
6. **Written interim rule** (MSP ticket + email) — attended only; no scheduled/unattended; **no NS create/edit/approve via UI or API** for any agent identity; “draft in NS UI” = write = banned.
7. **Cua / Driver / similar** — same containment; if it can drive a browser with NS SSO, out of policy until sandbox has **no route to NS host**.

**CFO teeth line:** “We’re not banning Claude Desktop — we’re making **agent-capable profiles unable to reach NetSuite** this week, and delivering a **Pending Approval API** yes-path in weeks so Marketing isn’t stuck.” If MSP can’t block URLs in ~5 days, escalate that as the **IT control gap** — not Marketing’s pilot.

---

## Recommended interim policy (one page for Council)

**Allowed now (attended):**
- Claude Desktop (or similar) assisting **Marketing systems** and non-SoR drafts the human still owns  
- Assist on **email/PDF extract** into a **staging sheet/queue Chris designates** — human pastes/imports; agent does not hold NS credentials  

**Not allowed now:**
- Any agent/computer-use path that can **create/edit/approve** NetSuite transactions (including “just a draft” in the UI)  
- Unattended runs; agent identity with NS write; shared SSO between daily NS work and agent desktop  

**Yes-path (next 2–6 weeks, not “day 90”):**
- NetSuite host: create-Pending-Approval-only integration role + SuiteFlow notify (existing staged-order options)  
- Marketing / Ops messy orders: extract → stage → human approve in NS — Claude can help extract; **middleware** creates the draft  

---

## CFO briefing arc (5 minutes)

1. **Situation:** Marketing already piloting attended Claude Desktop — stopping cold without a yes-path creates shadow use and political loss.  
2. **Risk:** Not “AI bad” — **ERP write + inherited SSO + wrong-valid fields** without staging/audit.  
3. **Ask:** CFO backs interim boundary + names Chris owner of agent↔ERP plane; MSP executes profile/URL blocks this week.  
4. **Offer:** Parallel Rock — draft-only staging so Marketing gets a governed speed path within weeks.  
5. **Metric:** Wrong-valid field rate and “NS write from agent-capable profile = 0” as Scorecard items.

---

## Messaging Themes (local weather)

Acknowledge: “Marketing is moving — good energy.”  
Facts: Attended assist ≠ unattended ERP write; UI draft is still a write.  
Pivot: We channel speed into **staging + human commit**; we don’t wait three months doing nothing and we don’t bless NetSuite clicks.

---

## Decision asks

1. CFO: Endorse interim policy + Chris ownership of agent↔NetSuite boundary?  
2. MSP/IdP: Can NetSuite be blocked from Claude Desktop profile **this week**?  
3. NetSuite host: Timeline for draft-only Pending Approval role (weeks, not quarter)?  
4. SVP Marketing: Willing to keep pilot **off NS write** if we deliver a staging on-ramp on a named date?

---

*IT/OT this-week list folded 2026-09-21. CFO teeth line ready for slide.*
