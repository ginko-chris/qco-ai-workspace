# CFO Scorecard — Grok as Third Option (Pilot Only)
**From:** QCo Analyst · **For:** CFO (then AI Council) · **Date:** 2026-09-22  
**Ask:** Approve a **bounded pilot**, not a companywide flip.  
**Evidence:** Grok Bot lived use (Chris); Grok 4.7 Bot-native harness (https://x.ai/news/grok-4-7); AA cost-per-task caveat (https://artificialanalysis.ai/articles/benchmarking-grok-4-7); Claude ~80 min outage (multi-model failover); Floor / staged-order / Claude-Desktop channel policy.

---

## One-liner

**Add Grok as a third horse for routine BotOps + coding assist — pilot 1–2 surfaces, measure $/finished Rock task, keep Claude/Astra for consequential work, keep NetSuite off every agent UI.**

---

## Why a third option (local weather)

| Signal | What it means for QCo |
| --- | --- |
| Chris already clears the bar on Grok Bot for Director knowledge work | Lived evidence for **right-sized routine assist** — not a theory |
| Grok 4.7 trained on **Grok Bot harness**; live in Cursor / Build / API | Real product path for BotOps + coding lane (not Galaxy theatre) |
| AA: same list $/M can hide ~**2× tokens/task** vs 4.6 | Pilot must diary **$/finished task**, not trust sticker pricing |
| Claude elevated-errors outage (~80 min) | Single-home on one lab is an ops risk — failover is prudence |
| Marketing Claude Desktop already in flight | Multi-vendor reality; governance = **channel + Floor**, not one brand |

**Not the case:** “Grok won AI” / replace Claude/Astra everywhere / unattended NetSuite.

---

## Pilot shape (phase 1)

| Dimension | Lock |
| --- | --- |
| **Surfaces (max 2)** | (A) **Grok Bot** for routine Director/ops knowledge assist · (B) **Cursor + `grok-4.7`** for bounded coding/script assist |
| **People** | Chris + ≤5 named volunteers (mix Ops/Corporate knowledge work — not a department mandate) |
| **Duration** | **30 days** or 20–50 finished tasks (whichever first), then go/kill review |
| **Models** | `grok-4.7` (and Bot default) only for in-scope tasks; no silent upgrade of all seats |
| **Floor (non-negotiable)** | No NetSuite UI / no NS SSO on agent profiles; no standing prod secrets; human gate before any SoR write; attended or short Bot routines only — same as Claude channel policy |
| **DPA / identity** | Managed enterprise path + IdP clear **before** seat growth beyond pilot |

---

## Success metrics (must beat baseline)

| Metric | Target | Kill if |
| --- | --- | --- |
| **$/finished Rock-relevant task** (tokens × price × rework time) | ≤ baseline Claude/Astra routine lane **or** clearly better quality at ≤1.25× cost | Cost **≥2×** without quality lift (AA warning) |
| **Task success / rework rate** | ≥ baseline on same golden set (20–50 tasks) | Systematic wrong-valid answers or language-drift on long jobs |
| **Time-to-useful answer** | Measurable win on routine assists Chris already does | Friction > value (IdP/deny/tool gaps block real work) |
| **Floor breaches** | **0** NS writes / shared SSO / unattended SoR | Any breach → immediate pause |
| **User willingness** | Pilot cohort would keep using for in-scope work | Cohort abandons for Claude/Astra on the same tasks |

---

## What stays on Claude / Astra (phase 1)

- Consequential **agentic** / long-horizon work where Fable/Astra-class still leads internal judgment  
- Any workflow touching **NetSuite draft/commit** (staging API + human only — vendor-agnostic)  
- Marketing’s existing Claude Desktop pilot (**non-NS only**) until channel policy + staging yes-path mature  
- Legal/compliance-sensitive generation until a named eval set exists  

---

## What we explicitly will **not** put on Grok in phase 1

1. Companywide default model / mandatory seat swap  
2. NetSuite UI automation or any agent with NS credentials  
3. Unattended multi-hour agents (Mollick drift risk)  
4. “AI headcount” messaging — Rock-assist = **better work**, never Corporate attrition wins  
5. Galaxy / logo theatre as success criteria  

---

## Decision asks

1. CFO: Approve **30-day / 20–50 task** third-option pilot on Bot + Cursor only?  
2. Confirm Floor + DPA gates before any seat beyond pilot cohort?  
3. Name who owns the **cost diary** (Chris / Analyst facilitation)?  
4. Council: Accept “third horse / tier-by-job” as standing design — not a bake-off winner?

---

## Messaging Themes

Acknowledge Grok 4.7 ship → facts (Bot-native + API/Cursor + AA cost caveat) → pivot to **pilot seats we control**, multi-model failover, Floor unchanged.

*Companion briefs: `/workspace/qco/digest/2026-09-22-brief.md` · Claude Desktop CFO brief · staged-order options.*
