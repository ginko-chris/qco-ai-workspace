# QCo Evidence Pack — RLCD / TypeSafe / Cua (research only)

**For:** QCo Analyst (advise-memo draft)  
**From:** Research executor  
**Date:** 2026-09-21 (EDT) · **Status:** Evidence only — no go/no-go  
**Binding context:** Prior options memo `/workspace/qco/docs/ops/QCo-Staged-Order-Entry-Agent-Options.md` — hybrid extract→stage→human commit; **no browser write path into NetSuite**; no standing NetSuite write/admin for agents.

**Companion constraint reminder:** Outlook may be intake; messy email/PDF → agent extract; EDI/portal → iPaaS; thin IT; dual MSP + NetSuite host.

---

## BLUF of claims

| Actor | What they actually claim | One-line framing |
| --- | --- | --- |
| **TypeSafe AI** (`typesafe.ai`) | **RLCD = Reinforcement Learning for Calibrated Decisions** (not “Controlled”). Trains **Jev**, a “System One” model that returns **typed decisions + probabilities/confidence**, not chat text. Vendor headline: **~193.6× faster / ~444.6× cheaper** vs frontier LLMs on *their* System One workflow evals; latency **70–500 ms**; input **$0.042 / MTok**, output free; “can’t hallucinate” = **no type/schema errors**. | Decision layer for code (classify / route / score / gate), not a GUI form-filler. |
| **Cua** (`cua.ai` / `trycua/cua`) | Open-source **computer-use** stack (Driver, Sandbox, Fleets, Bench): agents **click / type / read AX tree** on real desktops. Separate research artifact **`cua-s1-forms`**: small **Jev-like** one-pass option scorer for **GUI form fill**, explicitly **not** TypeSafe RLCD. HF card reports very high top-1 on synthetic + tiny real demo; GitHub `libs/cua-s1` is more conservative (source-first / research profile). | Infrastructure + specialist form-fill decision head for **desktop RPA**, not an RLCD product. |

**Naming note:** Task brief said “Controlled Decision.” Primary TypeSafe spelling is **Calibrated Decisions**. TechCrunch (2026-09-18) quotes Almeida as “reinforcement learning **from** calibrated decisions.” Same acronym, different preposition — treat as one vendor method name, unpublished recipe.

---

## 1. TypeSafe findings + primary links

### 1.1 Product & method (vendor primary)

| Item | Detail | Source |
| --- | --- | --- |
| Launch | 2026-09-15; early access; founder Diogo Almeida (ex-OpenAI; TypeSafe credits him with RLHF co-invention) | [Launch post](https://typesafe.ai/blog/introducing-system-one-models-and-jev) |
| Funding | $40M seed (Business Wire / press) | [Business Wire](https://www.businesswire.com/news/home/20260915525333/en/TypeSafe-AI-Emerges-From-Stealth-With-%2440M-in-Funding-With-New-Model-for-Composable-AI) |
| Model | **Jev** (`jev-1.13.0` / aliases `jev-latest`, `jev-preview`); “System One” class | [Docs models](https://docs.typesafe.ai/models.md), [System One](https://docs.typesafe.ai/concepts/system-one.md) |
| I/O contract | State + typed questions → **Choice / Score / Noul** with probabilities; Choice/Score also return **confidence**; parallel eval of many questions in one call | [Introduction](https://docs.typesafe.ai/introduction.md), [Primitives](https://docs.typesafe.ai/primitives.md) |
| RLCD (vendor def) | Post-training that “trains TypeSafe to return decisions and calibrated probabilities instead of generated text”; contrasted with RLHF (human preference) and RLVR (verifiable rewards) | [AI primer](https://docs.typesafe.ai/introduction/machine-learning-primer.md), [Launch post](https://typesafe.ai/blog/introducing-system-one-models-and-jev) |
| Speed (vendor) | **70–500 ms** E2E; homepage **193.6× faster**; launch post range **40×–200×** for System One–shaped queries | Homepage [typesafe.ai](https://typesafe.ai/), launch post |
| Cost (vendor) | **$0.042 / MTok** input ($42 / BTok); **output free**; homepage **444.6× cheaper** (workflow-eval peak); input sticker **238×** below Claude Fable 5.1 (vendor arithmetic) | [Models](https://docs.typesafe.ai/models.md), homepage |
| Type-safety / “no hallucination” | Schema/type errors claimed **impossible by construction** (0% type error); “cannot hallucinate” in marketing = out-of-schema text, **not** “cannot pick wrong allowed option” | Launch post Evidence section; FAQ “Can Jev still get things wrong?” (yes) |
| Workflow evals | Custom “workflow” evals: agreement with average of large external models (Astra / Fable) as reference — **not** ground-truth labels; company notes workflows built by their capabilities team | Launch post; [evals.typesafe.ai](https://evals.typesafe.ai/) |
| Data | Almeida to TechCrunch: trained **exclusively on synthetic data** | [TechCrunch 2026-09-18](https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/) |
| Customization | **No** per-customer fine-tune / LoRA; same weights for all; domain via `state` + question criteria | [Models](https://docs.typesafe.ai/models.md) |
| Known limits (vendor) | Weak math/counting; dates as text; degrades with large irrelevant state; no guaranteed logical consistency across related questions (e.g. Noul + negation need not sum to 1); text-only input; English strongest; Choice cardinality up to 255 | [Jaggedness jev-1.13](https://docs.typesafe.ai/model-jaggedness/jev-1.13.md) |
| Pricing sustainability | Launch post: cannot prove pricing unsubsidized; expects prices to fall | Launch post |

### 1.2 Independent / secondary (not vendor marketing)

| Source | What was measured | Caveat |
| --- | --- | --- |
| [systemonemodels.org RLCD explained](https://systemonemodels.org/guides/rlcd-explained/) (updated 2026-09-20) | Glossary-style independent writeup: **no arXiv paper**, no reward function, no dataset, no reproducible training; Hugging Face models named “RLCD” post-launch are mostly **naming**, not reproductions | Secondary aggregator; useful as “what is unpublished” checklist |
| [TechCrunch](https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/) | Anecdotal developer quotes (Vercel classifier latency 5–18×; email class cost 10–20× cheaper vs Gemini with mixed accuracy); synthetic-data + RLCD quote | Journalism, not a controlled benchmark |
| [Capital & Compute](https://capitalandcompute.net/blog/typesafe-jev-system-one-models-cost-use-cases/) (2026-09-18…20) | Compiles **five outside tests** (Every, Near Here, gemanor, paddo.dev, Arize) + own 483-call run: measured cost advantage **~8.6×–580×** depending on comparator; advertised **444.6×** ≈ Opus-5 row; **abstain rate** can collapse effective saving (e.g. 30% abstain → ~3.2× blended vs Terra); latency band **reproduced** (~314 ms median) | Strong secondary synthesis; some figures rely on early-access onboarding pages (not public URLs) |
| [TrueStandard 108-claim test](https://truestandard.ai/blog/jev-accuracy-tested) | Jev 96.3% vs Flash Lite 94.4% / Haiku 4.5 93.5% on grounding claims; ECE separation tiny at n=108 | Small, custom corpus |
| [Progressive Robot / Every vibe-check summaries](https://www.progressiverobot.com/2026/09/16/jev-model-typesafe-programmatic-logic/) | Vendor dashboard agreement ~67.8% vs top comparator ~74.1%; Every: ~25× faster, large cost win, 6/7 planted defects vs Fable 7/7 | Task-specific |
| [Register](https://www.theregister.com/ai-and-ml/2026/09/16/typesafe-ai-debuts-model-for-machines-that-plays-doom/5296711) | Launch coverage (Doom demo angle) | Press |
| Arize / Laurie Voss (via secondary) | Spam 18,514 emails: Jev **98.3%** vs trained TF-IDF **98.4%** — rare **ground-truth** label test | Cited via Capital & Compute / Arize blog; treat as independent but verify primary before board use |

### 1.3 Unverified / unpublished (TypeSafe)

- Full **RLCD algorithm**, reward / proper scoring rule, training steps, architecture topology, parameter count, weights.
- Independent audit of **workflow evals**; public SWE-bench / GPQA / MMLU (vendor says wrong task class).
- Long-run proof that **$0.042** is sustainable / unsubsidized.
- Calibration **curves** (ECE) published by TypeSafe for Jev on named task families — calibration remains a **stated objective**; outside parties must measure on own data.
- Named **production NetSuite / ERP order-entry** customers (none found in sources reviewed).

---

## 2. Cua findings + primary links

### 2.1 Core product (computer-use infra)

| Item | Detail | Source |
| --- | --- | --- |
| Positioning | Scale **computer fleets** for computer-use agents: train / eval / data-gen on Linux, Windows, macOS, Android | [cua.ai](https://cua.ai/) |
| Open source | MIT; monorepo `trycua/cua` — Driver, Sandbox, Bench, Lume, agent libs | [GitHub trycua/cua](https://github.com/trycua/cua), [Docs](https://cua.ai/docs) |
| Cua Driver | Background clicks/type/scroll; accessibility tree + screenshots; MCP stdio / CLI / daemon; no stealing system cursor | [cua-driver page](https://cua.ai/cua-driver) |
| Form recipe | Official how-to: open local PDF → fill browser form → **submit** via Driver | [Fill a form from a local file](https://cua.ai/docs/how-to-guides/recipes/fill-a-form-from-a-local-file) |
| Cloud / compliance claims | Usage-based Fleet pricing; **SOC 2 Type I**, BYOC, on-prem stated on homepage | [cua.ai](https://cua.ai/) |
| RL / benchmarks | Cua-Bench: OSWorld, ScreenSpot, Windows Arena, custom tasks; trajectory export for training | GitHub README / Docs |

**RLCD:** Cua’s main site and docs **do not claim TypeSafe RLCD**. Their overlap with “System One / Jev” is the separate **cua-s1** research line.

### 2.2 `cua-s1-forms` / Cua-S1 (Jev-like form decision head)

| Item | Detail | Source |
| --- | --- | --- |
| Role | Small (~706k params, ~2.8 MB) one-pass **option scorer** for GUI form filling behind cua-driver; same *I/O shape* as Jev (probabilities over predefined options), **not** a chat LLM | [HF cua-ai/cua-s1-forms](https://huggingface.co/cua-ai/cua-s1-forms) |
| Explicit non-claim | **“Not calibrated with TypeSafe's RLCD method — independent research checkpoint, not a reproduction of Jev.”** | HF model card |
| Training (HF) | 10k synthetic episodes; CE loss; form-signature-disjoint splits; AdamW 6 epochs | HF card |
| Reported results (HF) | Synthetic test top-1 **99.95%**; real demo **100%** on 196 decisions (3 forms + 3 PDFs); shuffled-context control **37%**; vs hosted `jev-latest` zero-shot on same task: **99.7%** vs **83.6%** overall (Jev weaker on already-filled no-op convention it wasn’t trained for) | HF card |
| Safety design (repo) | Plan vs execute separated; dry-run default; submit opt-in; submit only high-confidence Submit/Submit Form button; cannot invent values — only choose among extractor `Label: value` entities | [libs/cua-s1 README](https://github.com/trycua/cua/tree/main/libs/cua-s1) |
| GitHub vs HF tension | `libs/cua-s1` **MODEL_CARD / README** (main, fetched 2026-09-21): research profile; **“does not distribute weights”**; **“No checkpoint performance claim is established by this source-only release.”** Meanwhile HF hosts weights + published ladders. Treat **HF numbers as project-claimed demo metrics**, and **GitHub prose as the more cautious official stance** until reconciled. | GitHub MODEL_CARD vs HF |

### 2.3 Unverified / limited (Cua)

- Independent third-party replication of **cua-s1** 99.7% / 100% figures outside demo set.
- Transfer to **NetSuite UI** (SSO, custom forms, SuiteScript-driven pages, 2FA) — not claimed.
- Whether HF checkpoint license/status matches commercial production terms (repo notes future weights may need separate commercial agreement).
- Any Cua claim to own or implement **RLCD** — **none found**.

---

## 3. Claim vs evidence table

| Claim | Who | Evidence class | Status |
| --- | --- | --- | --- |
| RLCD = training for **calibrated** decision probabilities | TypeSafe | Vendor docs + launch | **Defined as objective**; method **unpublished** |
| RLCD reproduced outside TypeSafe | — | arXiv / open training code | **Not found** (as of sources dated ~2026-09-18…20) |
| Jev ~70–500 ms latency | TypeSafe | Vendor + multiple outside runs | **Supported** (outside tests broadly match band) |
| 193.6× / 444.6× vs LLMs | TypeSafe homepage | Vendor workflow evals | **Vendor peak / home-field**; outside range ~**8.6×–580×** cost depending on comparator; speed often **~20–95×** on published tables |
| $0.042 / MTok, output free | TypeSafe | Public models page | **Sticker verified**; subsidy **unproven** |
| “Can’t hallucinate” | TypeSafe marketing | Schema guarantee | **Type-error** claim strong; **wrong-but-typed** answer still possible (vendor acknowledges) |
| Similar intelligence to frontier LLMs on System One tasks | TypeSafe | Agreement with other LLMs as reference | **Partially supported / contested**; often within a few points; sometimes below top comparator |
| Calibration usable for confidence gating | TypeSafe pattern docs | Objective + outside anecdotal | **Plausible architecture**; **must measure ECE / abstain on own data** |
| Cua is an RLCD / Jev competitor product | — | cua.ai | **False** — Cua is computer-use infra; s1 is Jev-*like* research |
| cua-s1 uses RLCD | Cua HF | Explicit denial | **False** (by their statement) |
| cua-s1 near-perfect form fill | Cua HF | Self-reported synthetic + n=196 real | **Unverified externally**; **narrow demo**; GitHub cautions against treating as established |
| GUI form-fill → NetSuite commit | Cua recipes / s1 executor | Product design | **Technically aligned with RPA**; **conflicts with QCo hard rule** |

---

## 4. What’s not said (gaps that matter for QCo)

1. **Neither vendor publishes a NetSuite- / ERP-order-entry case study** suitable for QCo’s staged approval architecture.
2. TypeSafe **does not generate SKUs, free-text line items, or PDF OCR** — extraction still needs regex / OCR / generative LLM; Jev is a **picker / scorer / gate** over bounded options (mirrors QCo “certified slices”).
3. TypeSafe **does not drive browsers**; “automation” means **software-native decisions**, not RPA clicks.
4. Cua **does** drive browsers/desktops and has recipes that **submit forms** — that is the **form-fill RPA** class QCo already flagged as lab-only for NetSuite.
5. RLCD **reward function, paper, and weights** are closed — buy API / early access, not self-host today.
6. **Abstain / low-confidence rate** is the economic and ops variable nobody can guess for QCo email/PDF orders without a pilot corpus.
7. Cua-S1’s high scores assume a **prior document extractor** that already emitted `Label: value` pairs — the hard QCo problem (messy email/PDF → correct entities) is **upstream**, not solved by the option scorer.
8. Thin IT / MSP + NetSuite host: introducing **desktop Driver fleets** or **new SaaS decision API** are different RACI / egress / secrets problems; neither replaces **Pending Approval API role** work with the host.

---

## 5. Hype level

| Topic | Hype level | Comment |
| --- | --- | --- |
| TypeSafe **speed & token economics** | **Medium–High marketing, Medium evidence** | Latency and cheap input look real; homepage multiples are **best-case peaks**. |
| TypeSafe **RLCD as novel science** | **High marketing, Low evidence** | Name + objective only; no paper / recipe. |
| TypeSafe **“can’t hallucinate”** | **High marketing, Medium-narrow truth** | True for **schema**; false if read as **semantic infallibility**. |
| TypeSafe **frontier intelligence parity** | **Medium marketing, Mixed evidence** | Often close on decision tasks; not universally ahead; evals often LLM-agreement. |
| Cua **computer-use platform** | **Medium** | Real OSS + cloud product category; stars/community large; not magic. |
| Cua-S1 **99%+ form accuracy** | **High for a research demo, Low external corroboration** | Tiny real n; GitHub itself downplays established claims; **not RLCD**. |

---

## 6. Analyst-ready bullets — implications for form-fill RPA vs QCo preferred architecture

*Prefer architecture (binding):* **extract → canonical staging schema → middleware/API → NetSuite Pending Approval → human commit.** Hard rule: **no browser write path into NetSuite.**

1. **RLCD/Jev is a decision API, not an order-entry RPA bot.** Best conceptual fit for QCo is **confidence-gated field validation / customer-SKU matching / route-to-exception**, sitting *after* extract and *before* stage — not clicking NetSuite.
2. **“Calibrated confidence” maps cleanly to QCo fail-closed rules** (auto-stage only above threshold; else human queue) — **if** QCo measures calibration and abstain rate on **its** email/PDF corpus; vendor calibration is not yet independently standardized.
3. **Do not equate “can’t hallucinate” with safe unattended commit.** Wrong Choice among allowed SKUs is still a bad order; certified item/customer lists + human approve remain mandatory.
4. **Cua Driver + form-fill recipes are the opposite architectural pole** from the preferred path: they optimize **GUI mutate/submit**. Useful for lab demos or non-NetSuite desktops; **in conflict with the no-browser-write hard rule** for NetSuite draft/commit.
5. **`cua-s1-forms` is interesting as a pattern** (bounded options from extractor; code owns order/submit) but **wired to cua-driver execution**. For QCo, keep the pattern (**bounded Choice over extracted entities**) and **swap the sink** to staging API — do not adopt the Driver submit path for NetSuite.
6. **Cua does not claim RLCD**; treating cua.ai and typesafe.ai as one “RLCD vendor shortlist” is a category error. Shortlist split: **(A) decision model API** vs **(B) computer-use RPA infra**.
7. **EDI/portal stays iPaaS** — neither Jev nor Cua changes the prior memo’s premise that structured intake should not use agents.
8. **Pilot metric if Analyst explores TypeSafe:** % auto-staged at confidence ≥ T, SKU/customer error vs human baseline, abstain/escalation rate, $/order, latency — **not** “RPA percent of NetSuite UI automated.”
9. **Host/MSP implication:** TypeSafe = egress to decision API + secrets for API key; Cua fleets/Driver = desktop identity, session, and (if misused) NetSuite UI credentials risk — latter is **higher trust-boundary smell** under current constraints.
10. **Go/no-go deferred to Analyst** — this pack does not recommend buy/build; it only separates marketing from measured evidence and maps both to the staged-order architecture.

---

## Primary URL index (quick)

**TypeSafe:**  
https://typesafe.ai/ · https://typesafe.ai/blog/introducing-system-one-models-and-jev · https://docs.typesafe.ai/ · https://docs.typesafe.ai/introduction/machine-learning-primer.md · https://docs.typesafe.ai/models.md · https://docs.typesafe.ai/model-jaggedness/jev-1.13.md · https://evals.typesafe.ai/ · https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/

**Independent TypeSafe/RLCD notes:**  
https://systemonemodels.org/guides/rlcd-explained/ · https://systemonemodels.org/glossary/rlcd/ · https://capitalandcompute.net/blog/typesafe-jev-system-one-models-cost-use-cases/

**Cua:**  
https://cua.ai/ · https://cua.ai/docs · https://cua.ai/cua-driver · https://cua.ai/docs/how-to-guides/recipes/fill-a-form-from-a-local-file · https://github.com/trycua/cua · https://github.com/trycua/cua/tree/main/libs/cua-s1 · https://huggingface.co/cua-ai/cua-s1-forms

---

*End of evidence pack. Research only — no QCo go/no-go.*
