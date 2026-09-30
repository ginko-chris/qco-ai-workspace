# QCo Options Memo — Staged Sales Order Entry (Agent / RPA)
**From:** QCo Analyst · **For:** Chris (Director of AI & Technology)  
**Date:** 2026-09-18 · **Status:** Options only — not a build plan  
**Constraints:** NetSuite SoR (3rd-party host); MSP co-managed IT; no standing NetSuite write/admin in agent path; human approve before commit; certified-slices SoT; thin IT; EOS / Rock-assist framing  

**Companion:** `/workspace/qco/docs/security/QCo-Agent-Eval-Hygiene-and-Network-Safeguards.md`

---

## BLUF

**Hard rule:** agents may **draft** sales orders unattended; **only humans** (or a governed NetSuite workflow with named approvers) **commit** to NetSuite.

**Best near-term pattern for QCo:** **hybrid** —  
- **Structured intake** (EDI / portal / clean CSV) → iPaaS or SuiteScript into a **NetSuite staging custom record / Sales Order in Pending Approval** (no agent required).  
- **Messy intake** (email + PDF / free-text) → agent (Grok Bot or other) extracts to the **same staging store** → human reviews in NetSuite or a review queue → commit.  

Do **not** give Grok Bot (or any agent) standing NetSuite write credentials that can create/fulfill orders. Prefer: agent writes only to a staging table/queue you control; NetSuite integration uses a **scoped service role** that can create *Pending Approval* orders only (or SuiteFlow that blocks release without role).

**90-day realism:** pick **one** intake channel (likely email+PDF) as a Rock-assist pilot; measure duplicate rate, SKU/qty error rate, time-to-approve — not “full unattended RPA.”

---

## Premise challenge (when *not* to use agents)

| Intake type | Prefer | Why |
| --- | --- | --- |
| EDI / customer portal / structured CSV | **iPaaS / SuiteScript / Celigo-class** | Deterministic mapping; agents add hallucination risk with little upside |
| Clean email with fixed template | Light RPA or scripted parse → staging | Agent optional |
| Messy email, scanned PDF, multi-line exceptions | **Agent extract → stage** | High variance; human review already required |
| Phone / verbal | Human entry | Don’t automate |

**Vibecheck:** “unattended RPA with agents” is often a nerd trap when the real win is **staging + approval discipline** — agents are only the extract layer.

---

## Architecture options

### Option A — Grok Bot–centric (browser / Outlook → staging)
**Flow:** Outlook/PDF → Bot routine extracts fields → writes draft to staging (Sheets/DB/custom app or NetSuite *Pending Approval* via tightly scoped API called by a **middleware**, not by Bot with admin SSO) → human approves in NetSuite.

**Pros:** Fits existing BotOps interest; 1Password approve-fills for any break-glass; good for email mess.  
**Cons:** No first-party NetSuite connector today → brittle browser automation **or** need middleware; Galaxy theatre ≠ production controls.  
**Fit:** Pilot if email is the pain and Bot is already the daily driver.

### Option B — Dedicated RPA / iPaaS (UiPath/Power Automate/Celigo/Boomi-class)
**Flow:** Connector watches mailbox/SFTP → map → NetSuite staging/Pending Approval → SuiteFlow notify approver.

**Pros:** Proven order-entry pattern; audit; retries; MSP/NetSuite host more likely to support.  
**Cons:** License + integration project; weaker on weird PDFs unless +OCR/LLM step.  
**Fit:** Default for EDI/portal; hybrid with LLM extract for PDFs.

### Option C — NetSuite-native (SuiteScript / SuiteFlow / CSV import + Approval)
**Flow:** Files drop to File Cabinet / email capture SuiteApp → script parses or queues → SO in Pending Approval → role-based approve.

**Pros:** Stays inside SoR trust boundary; host can RACI it; strongest audit.  
**Cons:** Dev capacity on NetSuite firm; PDF NLP still needs external LLM unless SuiteApp.  
**Fit:** Always own **commit + approval** here even if extract is elsewhere.

### Option D — Hybrid (recommended)
**Extract:** Agent or LLM service for messy docs only → **canonical staging schema** (customer ID, ship-to, SKU, qty, UOM, promise date, source message ID).  
**Stage:** NetSuite custom record **or** SO status = Pending Approval (cannot fulfill/invoice).  
**Commit:** Human (or dual-control for high $) in NetSuite.  
**Identity:** Agent = read Outlook + write staging only; NetSuite integration identity = create Pending Approval only; approvers = named Ops roles.

---

## Intake paths — unattended vs human

| Step | Unattended OK? | Notes |
| --- | --- | --- |
| Ingest email/PDF/EDI | Yes | Idempotent on Message-ID / file hash |
| OCR / extract fields | Yes (agent/LLM) | Bound to certified customer/SKU lists |
| Validate vs NetSuite master (customer, item, price book) | Yes (API **read**) | Fail closed → exception queue |
| Resolve ambiguous customer/SKU | **No** | Human |
| Create staged draft | Yes | Staging or Pending Approval only |
| Change price / credit / new customer | **No** | Human / credit role |
| Approve & commit | **No** (human) | Named approver; $ threshold dual-control |
| Fulfill / invoice | **No** agent | Existing NetSuite process |

**Certified slices:** customer master, item master, price book, ship methods — must be queryable SoT; agent may not invent SKUs.

---

## Trust boundary checklist

1. **Staging store** — NetSuite Pending Approval SO *or* custom “Order Draft” record (prefer NetSuite-native for one SoR).  
2. **Audit** — source Message-ID, extract JSON, model/version, validator results, approver, timestamps.  
3. **Idempotency** — unique external ID; reprocess ≠ duplicate SO.  
4. **Approvers** — Ops order-entry role; optional second approver above $X.  
5. **Rollback** — cancel/reject draft; never auto-delete committed SO without NetSuite process.  
6. **Secrets** — Bot: no NetSuite admin; 1Password approve-fill only for break-glass; integration: scoped token, default-deny egress.  
7. **Eval** — disposable VM for extract harness; never test writes against prod NetSuite.

---

## Risks (hallucination / ops)

- Wrong SKU / qty / UOM / ship-to → bad promise dates (on-time measured from **firm delivery commitment**, not early interest).  
- Duplicate orders from email threads.  
- Agent “helpfully” creates new customers/items.  
- Browser RPA breaks on NetSuite UI changes.  
- MSP/NetSuite host RACI gaps on who owns the integration identity.

**Mitigations:** fail closed to exception queue; fuzzy match only with confidence threshold; block unknown SKUs; human must pick from NetSuite search results for low confidence.

---

## 90-day recommendation (SMB)

| Phase | Do |
| --- | --- |
| **Now–D30** | Map volume by intake channel; confirm NetSuite Pending Approval / SuiteFlow capability with host; pick **email+PDF** as pilot slice if that’s the pain. |
| **D30–D60** | Build **staging + approval** in NetSuite (Option C core) + LLM extract to staging (Bot or lightweight API) — **not** Bot clicking Commit. |
| **D60–D90** | Pilot with one customer segment; Scorecard: % auto-staged, % rejected, error rate vs human baseline, time-to-approve. |

**Build vs buy:** Buy/reuse NetSuite approval + iPaaS if EDI exists; **buy LLM extract** (or Bot) only for messy channel; **don’t build** a custom agent framework.

**Council / Rock framing:** Rock-assist = “faster accurate order staging for Ops,” not fewer Corporate seats.

---

## Decision asks for Chris / Council

1. Which intake channel is the real pain (email PDF vs EDI vs portal)?  
2. NetSuite host day-one: native approval vs Order Draft; **create-Pending-Approval-only** API role timeline; **who holds the secrets** (host vs MSP)?  
3. Confirm **no browser write path** to NetSuite (draft or commit) — API/middleware only?  
4. Who is the named Ops approver role?  
5. Accept hybrid (agent extract + NetSuite stage) vs insist Bot-only?  
6. $ threshold for dual approval?

---

*IT/OT: please pressure-test SuiteFlow/API staging, host willingness, and failure modes for unattended extract.*

---

## IT/OT pressure-test (2026-09-18) — adopted

**Agree:** hybrid extract; **no Bot standing NetSuite write**; browser-RPA **not** on draft-create or commit.

**Host / SuiteFlow (ask day one):**
1. Native SO approval vs custom “Order Draft” record  
2. Timeline for **create-Pending-Approval-only** integration role (TBA/OAuth)  
3. Who owns that role’s secrets (**NetSuite host ≠ MSP**)  

Red flag: “we’ll just do it in the UI” / soft status convention — require **API/script-forced** Pending Approval status the token cannot promote.

**Draft-only role:** Create SO/draft **without** Approve / Fulfill / Invoice / Item Receipt; SuiteScript forces status; `externalId` = Message-ID/hash for idempotency.

**Browser-RPA:** Lab demo only — SSO/2FA, custom forms, UI churn, session death. Production path: **agent → staging schema → middleware/host API → Pending Approval**.

**Unattended extract:** fail closed on unknown SKU/customer; ambiguity stays human.
