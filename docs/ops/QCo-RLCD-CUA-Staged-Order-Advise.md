# Advise — TypeSafe RLCD / Jev & Cua for Staged Order-Entry RPA
**From:** QCo Analyst · **For:** Chris (Director of AI & Technology)  
**Date:** 2026-09-21 · **Status:** Advise — not a buy / build plan  
**Evidence:** `QCo-RLCD-CUA-Typesafe-RPA-Research.md` · **Architecture lock:** `QCo-Staged-Order-Entry-Agent-Options.md`

---

## BLUF

**Do not** change the staged order-entry architecture for these products.

**Sequencing (bold):** (1) Stand up NetSuite **draft-only / Pending Approval** staging via host API (boring Rock first). (2) Only then run a **100–200 order** ground-truth Jev/extract eval. (3) Jev earns a seat **only if** it beats baseline on **wrong-but-valid SKU** (and friends) at the auto-stage confidence threshold — not on type-error rate. Do not let a shiny Jev pilot delay the staging path.

| Piece | What it is | QCo use |
| --- | --- | --- |
| **RLCD** | TypeSafe **Reinforcement Learning for Calibrated Decisions** (not “Controlled”) — proprietary training claim behind **Jev** | Maybe later for messy extract + confidence gate into **staging** |
| **Jev** | Fast typed decision API (Choice/Score/Noul + probs); no free-text; schema-safe ≠ business-correct | Optional second layer after OCR/LLM candidates — **after** staging exists |
| **cua.ai** | MIT OSS **computer-use infra** (Driver / Sandbox / Bench) — **not** an RLCD model | Lab/eval VMs only; **no route to NS host / no shared NS SSO** |
| **jev-use** | Compose Jev decide + Cua click | Demo/lab only — **never** NetSuite UI (UI draft create still counts as a write) |

**Hard rule unchanged:** extract → stage → human commit; NetSuite write only via scoped API/middleware to **Pending Approval**; **no browser write** to NetSuite — including “just create draft” via UI.

---

## Claims vs evidence (one screen)

| Claim cluster | Credibility | Implication |
| --- | --- | --- |
| Speed (~70–500 ms) / cheap input tokens / free outputs | **High** (spot-checkable) | Real for classify/route loops — not proof of order accuracy |
| “Can’t hallucinate” / 0% type errors | **Schema guarantee** | Still picks wrong valid SKU/customer |
| Intelligence ≈ frontier on System One; 40–200× faster | **Self-eval** (agree with Astra/Fable consensus) | Not Ops ground-truth accuracy |
| Calibration (conf ↔ hit rate) | **Unverified** publicly (no ECE curves; no RLCD paper/recipe) | Must measure on QCo labels before auto-stage |
| Cua “open-source desktop model” | **Misread** — OSS is the **driver/sandbox**, not an RLCD foundation model | Useful harness; not a NetSuite RPA engine |

---

## Recommendation (Option D hybrid — unchanged)

1. **First:** host checklist for draft-only token + Pending Approval staging (API/middleware) — this is the Rock.  
2. **Keep** EDI/portal/clean CSV on iPaaS/SuiteScript → Pending Approval.  
3. **Only after staging exists:** OCR/LLM extract → optional **Jev** enum pick + confidence → stage → human approve.  
4. **Jev seat gate:** **100–200** historical orders with Ops ground truth; primary failure mode = **wrong-valid SKU** (and qty/ship-to) at high confidence — not schema/type errors. Fail closed if it doesn’t beat baseline extract.  
5. **Cua:** disposable eval sandboxes only; no NS host route, no shared NS SSO cookies; MSP treats computer-use like untrusted RPA on allowlist talks.  
6. **Do not** socialize “desktop RPA into NetSuite” as the Rock — socialize “faster accurate staging.”

---

## Decision asks

1. Authorize a **time-boxed Jev eval** on historical messy orders (or defer until waitlist/DPA clear)?  
2. Confirm **no** Cua Driver / jev-use path on any host with NetSuite UI/SSO?  
3. Who owns Ops ground-truth labels for the eval (commercial / order desk)?

---

*Detail and URLs in the research note. IT/OT: please pressure-test NetSuite/browser conclusions.*
