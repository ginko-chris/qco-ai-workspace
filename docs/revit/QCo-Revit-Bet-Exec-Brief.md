# The Revit Bet: Putting QCo Products in Specifiers' Models
**For:** AI Council (CEO, CFO, SVP Marketing, CIO, Chief Power Engineer) · **From:** Chris, Director of AI & Technology · **Date:** Sep 27, 2026
**Status:** Advisory. This is a workable shape for discussion, not a locked plan. Every $ figure is sourced (see footnote), an **assumption**, or a **ROM** (rough order of magnitude).

## 1. The bet
Specifiers design in Revit. A competitor's plugin already puts their products in the model and builds the BOM from it, so switching to us means redrawing. **We bet that making our configured products correct and easy to place in Revit (right geometry, right IES photometry, right part number) will win more large spec projects and cut quote and rework errors.** Rules come first, then the Revit surface: we certify the rules for polyurethane rigid (and flexible after it), move them into Fusion Manage, then publish through a platform that builds parametric families from that data. We don't build a custom add-in until evidence shows the workflow itself is what wins deals.

## 2. Foundation vs. the Revit bet (honest split)
- **(a) Shared foundation: on the roadmap, *not funded*.** Rules certification, Fusion Manage, the knowledge graph and MES are all still concepts or evaluations. None has a budget or a timeline. **Certifying rules pays off with or without Revit** (fewer CPQ errors, cleaner quoting, MES-ready BOMs, trustworthy AI inputs). The Revit bet needs only the *rules + Fusion Manage slice*. The KG (unscoped) and MES (RFQ ROM **$134k–$353k over 5 yrs**, unfunded) are **not** Revit prerequisites.
- **(b) Revit-specific: the true incremental bet.** Platform, families, IES, integration, marketing. A C# add-in is an option, deferred.

**Table 1: Cost (low–high, mid in brackets)**
| Line | Year 1 (startup) | Ongoing / yr |
|---|---|---|
| **(a) Foundation:** key roles (Table 2) | $24–81k [$49k] | $21–50k [$32k] |
| Fusion Manage, 5–20 users @ **$1,115/user/yr list** | $6–22k [$11k] | $6–22k [$11k] |
| Fusion Manage rules-slice setup (partner) | $25–100k [$50k] *ROM, custom quote* | — |
| AI-assisted extraction tooling | $2–10k [$5k] *assumption* | — |
| **Foundation subtotal** | **$57–213k [$115k]** | **$27–72k [$43k]** |
| **(b) Revit:** key roles (Table 2) | $81–161k [$123k] | $48–108k [$76k] |
| Platform: BIMStreamer Standard→Advanced **€11–16k/yr** + BIMobject listing *placeholder $10–30k (custom quote, discovery call)* | $13–49k [$29k] | $13–49k [$29k] |
| Platform onboarding / data-model setup | $10–40k [$20k] *ROM, custom quote* | — |
| Parametric families, LOD 300 (6–12 masters × **$500–1,500**) | $3–18k [$8k] | in content role |
| IES photometry (15–40 lab measurements @ **~€520–775**, scaled by length) | $9–36k [$19k] | $3–12k [$6k] |
| Launch / specifier outreach | $5–20k [$10k] *assumption* | — |
| **Revit subtotal** | **$121–324k [$209k]** | **$64–169k [$111k]** |
| *Option, deferred:* C# add-in | *build **$35–100k+*** | *maint. **$3k–$25k/yr**; plus .NET 8 (2025) → **.NET 10 (Revit 2027)** churn* |

Of Revit Year 1, about **$52–193k [$106k] is new cash**. The rest is time from people we already have.

**Table 2: Key roles (loaded rate = base ÷ 0.685; BLS says wages are 68.5% of full-time private compensation)**
| Role | Year 1 | Ongoing | Loaded rate | Yr 1 $ / Ongoing $ | Bucket |
|---|---|---|---|---|---|
| **Product director:** rule owner, signs off every rule | 2–5 hrs/wk in scheduled review blocks, 4–8 mo | 1–2 hrs/wk (~0.03–0.05 FTE) | ~$250k (~$120/hr)¹ | $4–20k / $6–12k | (a) |
| **R&D engineer:** first-pass AI-assisted extraction, back-testing | 0.4–0.6 FTE for 4–8 mo | 0.10–0.25 FTE (new SKUs, next family) | ~$152k (~$73/hr)¹ | $20–61k / $15–38k | (a) |
| **Content/configurator owner** | 0.25–0.5 FTE | 0.15–0.35 FTE | $145k *assumption* | $36–73k / $22–51k | (b) |
| **Platform/integration owner** (MSP/vendor) | 80–200 hrs | 30–80 hrs/yr | $150/hr *assumption* | $12–30k / $5–12k | (b) |
| **Marketing owner:** listing, leads, specifier follow-up | 0.15–0.25 FTE | 0.10–0.20 FTE | $150k *assumption* | $23–38k / $15–30k | (b) |
| **IT oversight (Chris):** guardrails, vendor terms | 0.05–0.10 FTE | 0.03–0.075 FTE | $200k *assumption* | $10–20k / $6–15k | (b) |

The binding constraint is the **product director's calendar, not dollars**. That team is near capacity. We protect it with short fixed review blocks, the R&D engineer does the heavy lifting, and **nothing is released without sign-off**. The point is to give the director's team capacity back, not to cut headcount.

## 3. ROI: break-even framing (3-year view)
Spec comes months before purchase, so assume **no benefit in Year 1**. Payback has to come in Years 2–3.
**Formula:** Annual benefit = (extra projects won × avg project $ × GM%) + (quote/rework errors avoided × $ per error) + faster specifier response (shows up as wins).
**Break-even revenue / yr (Yrs 2–3)** = (3-yr Revit cost ÷ 2 − error savings) ÷ GM%.

**Table 3: Assumptions QCo must fill in: GM = 40%, avg large project = $100k**
| Scenario | 3-yr Revit cost | Break-even revenue / yr | ≈ Extra projects / yr |
|---|---|---|---|
| Low | ~$248k | ~$310k | ~3 |
| **Mid** | **~$431k** | **~$539k** | **~5–6** |
| High | ~$662k | ~$828k | ~8 |

**Plain read:** at mid cost, **the bet pays back if it moves ~5–6 additional large projects (~$540k revenue) per year** starting in Year 2. That's less if error savings are real. If Revit also had to carry the full foundation, the mid hurdle rises to ~$790k (~8 projects). Sales should sanity-check this against our annual large-spec opportunity count, win rate, and deals we know we lost to the competitor's plugin.

## 4. Decision asks and stage gates
- **Ask now:** (1) Make it a Rock to certify polyurethane-rigid rules: R&D engineer at ~0.5 FTE, product director in fixed review blocks. (2) Authorize quote calls with BIMStreamer, CADENAS and BIMobject. (3) Sales supplies the Table 3 inputs.
- **Gate 1 (~90 days): rules certified.** Certified rules reproduce a sample of past orders with **zero unfulfillable configs**, and the director signs off. *Kill/pause* if director hours run more than 50% over plan or the back-test match stalls.
- **Gate 2 (~6 mo): pilot.** Rigid family goes live on one platform for 3–5 friendly lighting designers, with correct IES per configuration. *Kill* if there are no spec inclusions or quote-quality gains within 2 quarters.
- **Gate 3 (~12–18 mo): expand.** Add flexible. Consider the add-in **only** if evidence shows BOM-from-model workflow is what decides deals.
- **IT guardrails (non-negotiable):** read-only, scoped integration identity, and **never write to NetSuite**. Publish only certified-slice data. The platform generator must *consume* the rules, not become a hidden second copy of them. Contract terms must give us data ownership, lead ownership and a clean exit/export. *Issue:* Fusion Manage isn't funded yet. If it slips, a certified rules table can act as the single interim source.

---
<sub>¹ Sources: BIMStreamer tiers, bimstreamer.com/en/pricing · BIMobject pricing is custom (no public rates), business.bimobject.com/plans-pricing/general-plans-pricing-faq · CADENAS BIMcatalogs, BIMsmith and ProdLib: no public manufacturer pricing (quote-based) · Fusion Manage $1,115/yr, autodesk.com/campaigns/fusion-360/offerings · Revit families $200–500 medium / $500–1,500+ complex, bim-services.us/revit-family-creation-services · DIAL photometric lab 2026 price list (B1 €519, B2 €775), dial.de · Add-in build ranges, studiokrew.com/blog/revit-plugin-development-aec-automation-cost; maintenance $250–800/mo, reope.com · Revit 2027 on .NET 10, help.autodesk.com (Revit 2027 What's New) · BLS OOH May 2025 medians: mechanical engineers $104,110, architectural & engineering managers $171,270 (product-director proxy); BLS ECEC June 2026. EUR→USD ~1.17 is an assumption. MES ROM comes from QCo MES RFQ one-pager. All other figures are labeled assumption/ROM.</sub>
