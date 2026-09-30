# Research Note — RLCD (TypeSafe) & CUA (cua.ai) for Form Input / Desktop RPA

**From:** QCo Analyst · **For:** Chris (Director of AI)  
**Date:** 2026-09-21 (EDT) · **Status:** Skeptical research — not a buy recommendation  
**Companion:** `QCo-Staged-Order-Entry-Agent-Options.md` (hard rule: extract → stage; humans commit; no NetSuite browser write)

---

## Naming correction (read first)

| Term people say | What it actually is |
| --- | --- |
| **“RLCD / Controlled Decision”** | **Wrong expansion.** TypeSafe’s term is **Reinforcement Learning for Calibrated Decisions** (sometimes quoted as “from calibrated decisions”). Same acronym, different preposition; goal is **honest probabilities**, not “controlled” UI clicks. |
| **Older arXiv “RLCD”** | Unrelated 2023 method: *Reinforcement Learning from Contrastive Distillation* (LM alignment). Do not conflate. |
| **CUA / cua.ai** | **Open-source computer-use infrastructure** (Driver, Sandbox, Fleets, Bench) — *not* an RLCD-trained model. |
| **jev-use** | Dev-preview **stacking**: TypeSafe **Jev** (decide) + **Cua Driver** (see / click / verify). |
| **ScaleCUA** | Separate OpenGVLab research CUA (cross-OS VLM agent). Not TypeSafe; not trycua’s product model. |

---

## 1. Claims table (vendor vs what can be checked)

### TypeSafe — Jev + RLCD

| Claim | Source | Skeptical read |
| --- | --- | --- |
| New model class: **System One** — typed decisions (Choice / Score / Noul), not chat | Launch post 2026-09-15 | Plausible product shape; early access / waitlist |
| Trained with **RLCD** for calibrated probabilities | Launch + docs | **Name + objective only.** No paper, reward function, dataset, or recipe published (as of ~2026-09-20) |
| **~70–500 ms** E2E; **$0.042 / MTok** input; **output free** | Launch / pricing | Latency/pricing are the easiest claims to spot-check on your traffic; sustainability of price not proven (vendor notes possible subsidy) |
| **~40–200× faster**, **~100–400×+ cheaper** vs frontier LLMs on “System One” tasks | Homepage / launch (“193.6× / 444.6×” from workflow evals) | **Self-run Pareto charts.** Gains are real *if* you already have a decomposed workflow; headline multipliers are upper-end marketing |
| Intelligence **similar** to frontier LLMs on System One tasks; workflow score **~67.8%** | evals.typesafe.ai | **Not ground-truth accuracy.** Labels = average of GPT-6 Astra + Claude Fable 5.1. Measures agreement with those models |
| **0% type / schema errors**; “can’t hallucinate” | Launch | **Guaranteed by construction** (output constrained to schema). Model can still pick the **wrong valid** option. 0% is asserted, not measured vs open generation |
| Confidence is **calibrated** (higher conf ⇒ higher accuracy) | Docs / marketing | Central to the pitch; **no published ECE curves / reliability diagrams** from TypeSafe. Must measure on QCo data |
| Text-only state; no native vision | Docs / third-party guides | Screenshots need OCR / accessibility tree / another model first |
| Synthetic training data only | TechCrunch 2026-09-18 (Almeida) | Interesting; unreproducible from outside |

### Cua (cua.ai / trycua/cua)

| Claim | Source | Skeptical read |
| --- | --- | --- |
| **MIT open-source** computer-use platform: Driver, Sandbox, Bench, Lume | GitHub + docs | **Strongest evidence trail.** Large public repo (~25k★ as of research date); real artifacts |
| **Background** desktop control (macOS / Windows / Linux): click, type, AX tree, without stealing focus | cua.ai / docs | Differentiator vs classic RPA that grabs the cursor; still GUI-fragile |
| Cloud Fleets / warm pools for eval, RL loops, trajectory data | Product site | Infrastructure sell; SOC 2 Type I / BYOC claimed — verify under NDA if relevant |
| **jev-use** = Fast Computer Use solved (Jev + Driver) | trycua announcements / PR draft ~#3943 | Vendor demo / internal task sets (e.g. 71/80 @ ~0.1s median vs Astra). **Not a public OSWorld SOTA claim for “Jev alone”** |
| Cua-Bench runs OSWorld / custom gyms | Docs + notebooks | Harness ≠ proven agent. Independent KiCad-style stress tests on Cua-Bench show **planning/perception still fail** even with strong LLMs |

### Composite “RLCD desktop RPA” narrative

| Claim | Reality |
| --- | --- |
| “RLCD models from typesafe.ai and cua.ai” as one product line | **Two companies.** TypeSafe = proprietary decision model. Cua = OSS control plane. They **compose** via jev-use; Cua does not ship an RLCD-trained desktop foundation model as its core open release. |
| Drop-in replacement for UiPath on NetSuite | **No.** Wrong trust model for QCo (see §4). |

---

## 2. Independent evidence vs marketing

### What exists (usable)

| Evidence | Independence | Weight |
| --- | --- | --- |
| TypeSafe launch post + docs + [evals.typesafe.ai](https://evals.typesafe.ai/) | First-party | High transparency on *methodology caveats*; low independence |
| TechCrunch (2026-09-18) — quotes Almeida; anecdotal developer tests (Vercel classifier latency; email classify vs Gemini) | Journalism + anecdotes | Mild positive signal for **speed/cost on classify/route**; not a controlled benchmark |
| Latent Space / ThursdAI / practitioner blogs (DEV, Substack) | Analyst / community | Useful architecture critique; still launch-week echo chamber |
| Launch-week demos: Browser Use + Jev, OCR+Jev Mac agent, etc. | Author self-reports | Show stack patterns; **not** production reliability |
| GitHub `trycua/cua` + docs + Cua-Bench | Open artifacts | Strong for **infra**; weak for “accuracy of automated form fill” |
| systemonemodels.org RLCD explainers | Secondary curation (not TypeSafe) | Good skeptical synthesis: **no arXiv RLCD paper**, HF “RLCD” names mostly branding |

### What does **not** exist (as of 2026-09-21)

- Peer-reviewed **RLCD** paper or open training recipe  
- Third-party reproduction of TypeSafe workflow evals with **human / business ground truth**  
- Published reliability diagrams / ECE for Jev on held-out domains  
- Independent OSWorld (or NetSuite-like ERP) score proving jev-use beats classic RPA + API for order entry  
- Proof that “calibrated” holds under **adversarial / messy vendor PDFs** and customer email threads  

**Bottom line on evidence:** Speed and schema-constraint claims are the most credible. Intelligence-parity and calibration claims are **plausible but self-attested**. Desktop RPA “solved” claims are **architecture demos**, not Ops-ready proof.

---

## 3. Architecture — how this differs from classic RPA / LLM browser agents

### Classic RPA (UiPath / Power Automate / Selenium)

```
Deterministic selectors / recorded macros → click/type → brittle on UI change
```

- Strength: audit, retries, enterprise connectors  
- Weakness: messy documents, ambiguous fields, UI churn  

### LLM browser / computer-use agents (Operator-class, CDP, screenshot→VLM)

```
Screenshot / DOM → big VLM reasons → emit actions → loop
```

- Strength: flexible on novel UIs  
- Weakness: slow, expensive, can invent actions, hard to gate; CDP often detectable  

### TypeSafe Jev (decision substrate — not a GUI driver)

```
Program state (text/JSON) + typed questions → parallel Choice/Score/Noul + probabilities
```

- **No free-text generation.** Surrounding code owns retrieval, arithmetic, dates, and side effects.  
- “Calibrated decision” **in practice** = you set **confidence thresholds** (auto / confirm / human) *if* you validate calibration on your data.  
- Fits **classify, route, score, pick-from-enum, guardrail** — not “fill NetSuite and commit.”

### Cua Driver / Sandbox (GUI substrate — not a decision model)

```
AX tree / screenshot / window state → agent or client builds candidates → clicks/types in background → optional verify
```

- Desktop-native; MCP/CLI; sandboxes for eval/RL data gen  
- Model-agnostic: Astra, Claude, Jev, etc. are swappable  

### jev-use pattern (fast computer-use stack)

```
Observe (Cua) → optional vision parse → client builds bounded action IDs
    → Jev Choice picks one ID → Driver executes + verifies
```

- Converts open-ended “what should I do?” into **bounded selection** (System 1 pick among candidates).  
- Reasoning / free-text typing still need code or a generative model.  
- Desktop vs web: Cua emphasizes **native apps + AX**; web still possible via browser-in-sandbox but is a different failure mode than CDP-only agents.

**Practical meaning of “controlled / calibrated decision” for Ops:**  
Not magic UI control. It means **narrow judgments with scores you can threshold**, wrapped in deterministic code — closer to a smart classifier API than to an autonomous clerk.

---

## 4. Fit for QCo staged sales-order entry

**QCo hard rule (unchanged):** agent extracts → **stage**; humans commit; **API/middleware only** for NetSuite Pending Approval; **NO browser write path** to NetSuite draft/commit UI.

### Where RLCD/Jev **could** help

| Use | Why it fits |
| --- | --- |
| Messy **email + PDF** field extraction *after* OCR / LLM candidate generation | Jev Choice over certified SKU/customer lists + Noul “is this a PO?” + confidence → exception queue |
| **Routing / triage** (which queue, urgency, incomplete vs complete) | Cheap parallel questions; cascade before calling expensive models |
| **Confidence gating** into staging vs human exception | Aligns with staged architecture *if* calibrated on QCo labels |
| Guardrails on LLM extract JSON (schema already fixed; score inconsistency) | “Wrong but valid” still possible — treat as second opinion, not oracle |
| Non-NetSuite **internal** forms / spreadsheet cleanup / mailbox tagging | Lower blast radius than ERP writes |

### Where Cua **could** help (narrowly)

| Use | Why / caveat |
| --- | --- |
| Eval harness for extract agents in disposable VMs | Cua Sandbox / Bench matches existing “never test writes on prod NS” hygiene |
| Automating **non-NetSuite** desktop tools (PDF rename, folder drop, legacy Win apps with no API) | Only if Ops owns the machine and RACI is clear |
| Background AX exploration in a **lab** to map fields | Research only — not production commit path |

### Where they should **NOT** be used

| Anti-pattern | Why |
| --- | --- |
| **NetSuite UI clicks** to create/edit/approve/fulfill SOs | Violates QCo rule; UI brittle; MSP/host RACI nightmare; audit weaker than API |
| Unattended “computer use” that can reach Commit / Approve | Same as above; confidence ≠ authorization |
| Replacing iPaaS / SuiteScript for EDI / clean CSV | Deterministic mapping wins; agent adds risk |
| Trusting Jev for **date math, qty arithmetic, price calc** | Vendor “jaggedness” admits weakness; do in code + master data |
| Treating “can’t hallucinate” as “can’t be wrong” | Schema-safe ≠ business-correct |
| Standing agent identity with NetSuite write beyond Pending Approval staging | Explicitly forbidden |

### Recommended stance (aligned with Option D hybrid)

1. **Commit path:** NetSuite Pending Approval / SuiteFlow via **scoped middleware API only** — unchanged.  
2. **Extract path (pilot):** OCR/LLM → structured candidates → **optional Jev** for enum pick + confidence gate → staging schema.  
3. **Cua:** optional **lab/eval** tooling; not the production NetSuite actuator.  
4. **Do not** pitch “desktop RPA into NetSuite” as the Rock; pitch “faster accurate staging.”

---

## 5. BLUF for Chris

**BLUF:** TypeSafe’s **RLCD** is **Calibrated Decisions** (proprietary Jev classifier/scorer), not a desktop RPA engine. **cua.ai** is **open-source GUI/computer-use infra** that can *call* Jev (jev-use), not an RLCD model release. Speed/cost claims for structured classify/route look directionally real; accuracy/calibration claims are **self-eval / consensus-with-frontier-LLMs**, not independent ERP proof. For QCo, Jev is a **maybe** for messy extract + confidence gating into staging; Cua is a **maybe** for sandboxed eval / non-ERP desktop chores. **Neither should click NetSuite.** Keep humans on commit; keep NetSuite writes on API/middleware.

---

## 6. Open questions

1. Can we get early-access Jev and run a **blind** eval on 100–200 historical QCo email/PDF orders with **Ops ground truth** (SKU, qty, ship-to) — not Astra/Fable consensus?  
2. What’s the **calibration curve** on our labels at the thresholds we’d actually automate (e.g. auto-stage ≥0.9)?  
3. Early-access / data residency / DPA: is vendor PDF/email text leaving our boundary acceptable under MSP + NetSuite host policy?  
4. Does jev-use (or any Cua Driver path) ever need to touch a machine that can reach **prod NetSuite UI**? (Default answer should be no.)  
5. Cost at our volume vs current LLM extract — including OCR and candidate-generation steps Jev doesn’t replace?  
6. If Jev price rises or waitlist stalls, is the architecture still valuable with an open “System One–style” constrained decoder (community ports reproduce *interface*, not RLCD training)?  
7. For non-NetSuite forms: which legacy apps have **no API** and enough volume to justify desktop automation RACI with MSP?

---

## Sources / URLs

**TypeSafe / RLCD / Jev**  
- https://typesafe.ai/blog/introducing-system-one-models-and-jev (2026-09-15)  
- https://evals.typesafe.ai/  
- https://docs.typesafe.ai/ (and llms-full export)  
- https://systemonemodels.org/guides/rlcd-explained/ (skeptical secondary; updated ~2026-09-20)  
- https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/  
- https://dev.to/valyuai/how-to-use-jev-a-practical-guide-to-typesafes-system-one-model-g5e  

**Cua**  
- https://cua.ai/  
- https://cua.ai/docs  
- https://github.com/trycua/cua  
- ThursdAI / community coverage of jev-use (e.g. https://thursdai.news/releases/2026-09 )  

**Unrelated / do-not-confuse**  
- arXiv RLCD (contrastive distillation): https://arxiv.org/html/2307.12950  
- OpenGVLab ScaleCUA: https://github.com/OpenGVLab/ScaleCUA  

**QCo internal**  
- `/workspace/qco/docs/ops/QCo-Staged-Order-Entry-Agent-Options.md`

---

*End of note. Claims above are vendor-heavy; treat production adoption as gated on QCo-labeled eval, not launch Pareto charts.*
