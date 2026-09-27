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

**Table 3: Assumptions QCo must fill in: GM = 40%, large project = $500k (mid-sized = $100k)**
| Scenario | 3-yr Revit cost | Break-even revenue / yr | ≈ Extra projects / yr |
|---|---|---|---|
| Low | ~$248k | ~$310k | <1 large (~3 mid-sized) |
| **Mid** | **~$431k** | **~$539k** | **~1 large (~5–6 mid-sized)** |
| High | ~$662k | ~$828k | ~2 large (~8 mid-sized) |

**Plain read:** at mid cost, **the bet pays back if it moves ~1 additional large project ($500k), or ~5–6 mid-sized projects (~$540k revenue), per year** starting in Year 2. That's less if error savings are real. If Revit also had to carry the full foundation, the mid hurdle rises to ~$790k (~1.6 large projects). Sales should sanity-check this against our annual large-spec opportunity count, win rate, and deals we know we lost to the competitor's plugin.

## 4. Decision asks and stage gates
- **Ask now:** (1) Make it a Rock to certify polyurethane-rigid rules: R&D engineer at ~0.5 FTE, product director in fixed review blocks. (2) Authorize quote calls with BIMStreamer, CADENAS and BIMobject. (3) Sales supplies the Table 3 inputs.
- **Gate 1 (~90 days): rules certified.** Certified rules reproduce a sample of past orders with **zero unfulfillable configs**, and the director signs off. *Kill/pause* if director hours run more than 50% over plan or the back-test match stalls.
- **Gate 2 (~6 mo): pilot.** Rigid family goes live on one platform for 3–5 friendly lighting designers, with correct IES per configuration. *Kill* if there are no spec inclusions or quote-quality gains within 2 quarters.
- **Gate 3 (~12–18 mo): expand.** Add flexible. Consider the add-in **only** if evidence shows BOM-from-model workflow is what decides deals.
- **IT guardrails (non-negotiable):** read-only, scoped integration identity, and **never write to NetSuite**. Publish only certified-slice data. The platform generator must *consume* the rules, not become a hidden second copy of them. Contract terms must give us data ownership, lead ownership and a clean exit/export. *Issue:* Fusion Manage isn't funded yet. If it slips, a certified rules table can act as the single interim source.

---
<sub>¹ Sources: BIMStreamer tiers, bimstreamer.com/en/pricing · BIMobject pricing is custom (no public rates), business.bimobject.com/plans-pricing/general-plans-pricing-faq · CADENAS BIMcatalogs, BIMsmith and ProdLib: no public manufacturer pricing (quote-based) · Fusion Manage $1,115/yr, autodesk.com/campaigns/fusion-360/offerings · Revit families $200–500 medium / $500–1,500+ complex, bim-services.us/revit-family-creation-services · DIAL photometric lab 2026 price list (B1 €519, B2 €775), dial.de · Add-in build ranges, studiokrew.com/blog/revit-plugin-development-aec-automation-cost; maintenance $250–800/mo, reope.com · Revit 2027 on .NET 10, help.autodesk.com (Revit 2027 What's New) · BLS OOH May 2025 medians: mechanical engineers $104,110, architectural & engineering managers $171,270 (product-director proxy); BLS ECEC June 2026. EUR→USD ~1.17 is an assumption. MES ROM comes from QCo MES RFQ one-pager. All other figures are labeled assumption/ROM.</sub>

<div style="page-break-before: always;"></div>

## Appendix: Revit marketplaces
*This assumes QCo's market is primarily North America. **AUM = average users per month** (traffic). Platform-published figures are labeled "pub." Third-party traffic estimates are labeled "est." They often disagree by 2–3×, so treat them as shape, not count. "n/p" = not published.*

| Rank | Platform | What it is · who it serves · geography | AUM (monthly traffic) | Catalog | Lighting presence | Price |
|---|---|---|---|---|---|---|
| 1 | **BIMobject** | Marketplace plus an in-Revit design app. Architects and engineers, global. 2024 sales: EMEA 63%, North America 37%.[a] | **303K monthly downloading users** (pub, 2024)[a]; ~1.48M visits Feb 2026 (est)[b] | 2,400+ brands[a]; 6M registered users[c] | Cooper, Lutron/Ketra | Custom quote[d] |
| 2 | **BIMsmith** | US marketplace (Elgin, IL) run by architects. Architects and designers. | n/p; est. 94K–297K, 34% US[e] | "Hundreds" of makers, "tens of thousands" of products[f] | Signify, Lutron, Cooper | Quote |
| 3 | **BIMStreamer** | *Owned* white-label platform: PIM→BIM generation, branded Revit plugin, Revit BOM; the customer owns the data.[g] Polish, EU clients. | n/a (traffic lands on QCo's own site) | 20+ platforms; 500K generated files[h] | None seen | **€5–16K/yr** published, Enterprise by quote[g] |
| 4 | **ARCAT** | US spec, CAD and BIM library. Free to use, no registration. Architects and spec writers. | **175,727 users / 247,560 sessions** avg per month, 2025 (BPA-audited)[i] | "Thousands" of BIM objects[j] | Not checked | n/p |
| 5 | **CADdetails** | North American CAD/BIM library with intent analytics. | ~92K/mo (1.1M visitors/yr, pub)[k] | 750K+ registered NA AEC users; 2.3M downloads/yr[k] | Lumascape | n/p |
| 6 | **Sweets (Dodge)** | US product database. "96% of top 300 architecture firms."[l] | n/p | 79,000+ products[m] | Not checked | Quote |
| 7 | **CADENAS BIMcatalogs / 3Dfindit** | German. Parametric catalogs, strongest in mechanical and electrical parts. | est. 626K–895K visits (3dfindit.com)[n] | 5,500+ catalogs; 750M CAD downloads in 2025[o] | Eaton emergency lighting | Quote |
| 8 | **ProdLib** | Finnish, Nordic focus. Revit configurators (PROLICHT linear, LUOlight). | n/p | 40K+ Nordic users[p] | Euro linear brands | Quote[q] |
| 9 | **Architizer** | US design-inspiration and brand directory; not a Revit host. | est. 502K–577K (Jun 2026)[r] | n/p | Designer lighting brands | Quote |
| 10 | **NBS Source**; **SpecifiedBy** (now CausewayOne); **MEPcontent** | UK spec (NBS: 93% of AJ Top 100[s]); UK (109K registered, 2,600 makers[t]); EU MEP engineers (310K engineers, 90.6K downloads/mo, 481 makers[u]) | See catalog column | See left | Low | Quote |

*Not included: UNIFI (firms' internal content manager, not a manufacturer channel) and Revizto (a coordination tool). Polantis was checked but publishes no stats.*

**Recommendation and rationale.** No marketplace is where high-end lighting designers mainly find products. They rely on reps, manufacturer sites and IES files. A marketplace buys discovery and download signals, not specs. **Start with BIMobject, and keep QCo's master families and IES files on our own site.** BIMobject has the largest verified audience (303K monthly downloading users). Its in-Revit app matters because 18% of its survey respondents discover products through Revit/Archicad plugins. Our peers Cooper and Lutron/Ketra are already listed. It also gives firm-level download data for Test 5 in the evidence memo. Its weak spots are that it is EU-weighted (37% of sales from North America) and its pricing is opaque. So negotiate a 12-month pilot, get US lighting-category download stats before signing, and write in data export and lead ownership. **BIMsmith is the runner-up** if the BIMobject quote is high or its US reach looks thin. BIMsmith is US-native and carries Signify, Lutron and Cooper, but publishes neither traffic nor price. **BIMStreamer (or ProdLib or CADENAS) is the right tool only if Gate 3 evidence shows configured custom-cut families or BOM-from-model decide deals.** BIMStreamer is the only one with published pricing and explicit customer data ownership. ARCAT and CADdetails have credible US traffic, but they are spec- and CAD-centric and weak in lighting.

<sub>[a] investors.bimobject.com/media/0g1j00xh/bim-arsredovisning-2024-eng-webb-2025-04-30.pdf · [b] explodone.toolsurf.com/website/bimobject.com/overview (Exploding Topics data, Feb 2026) · [c] storage.mfn.se/04fe78b9-770f-4235-a11e-a94b11fb8e6f (Dec 2025 release) · [d] business.bimobject.com/plans-pricing/general-plans-pricing-faq · [e] mysite.info/analysis/bimsmith.com; linkedin.com/company/bimsmith (undated) · [f] join.bimsmith.com/join; bimsmith.com/help/getting-started/What-is-BIMsmith · [g] bimstreamer.com/en/pricing; bimstreamer.com/en/case-studies/bim-platform-and-revit-plugin-for-uponor · [h] bimstreamer.com/en/about-us · [i] arcat.com/reps/BPA_Audit.pdf · [j] arcat.com/about · [k] services.caddetails.com/blog/a-q-and-a-with-caddetails-senior-analytics-engineer · [l] construction.com/sweets · [m] construction.com/bpm · [n] xranks.com/ko/3dfindit.com; linkedin.com/company/3d-searchengine · [o] cadenas.de/files/cadenas/Downloads/PDF/Produktflyer/EN/CADENAS_BIMcatalgos_brochure_en.pdf; partsolutions.com/about · [p] prodlib.com/business · [q] tulitec.com/software/prodlib-software · [r] analytics.explodingtopics.com/website/architizer.com; sitestatsdb.com/websites/architizer.com · [s] thenbs.com/manufacturers · [t] specifiedby.com/about · [u] mepcontent.com/en (live counter, Sep 27, 2026)</sub>
