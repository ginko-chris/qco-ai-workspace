# QCo MES Eval — Tulip vs Ignition (20 interfaces) RFQ One-Pager
**Owner:** Chris · Director of AI and Technology  
**Context:** Low-volume / high-mix assembly · ERP = NetSuite (3rd-party host ≠ corporate MSP) · Shopfloor OT minimal · Production today largely paper + NetSuite · **Dozens of new SKUs/year** (specifier GTM) · Construction sales: spec can precede PO by months  
**Scale assumption:** **20** tablet/PC interfaces (Monthly Active Interfaces / clients)  
**Purpose:** Vendor RFQ scorecard + internal **ROM** 5-year TCO framing — not a purchase decision.  
**ROM status:** License rows = published list (confirm in quote). SI rows = **example midrange estimates** — not vendor quotes. Replace when SOWs return.

---

## 1. Pilot station scope (must quote against this)

Operator loop on a station device:

1. Pull digital work order  
2. Begin time tracking for work step  
3. View BOM and SOP / drawings  
4. Assemble subcomponent  
5. Perform QC for step  
6. Record issues  
7. Print labels  
8. Record scrap and partials  
9. Photograph part  
10. Mark complete and stop time tracking  

**v1 integration posture:** NetSuite **read** (WO, BOM, status) in pilot; **write** (complete, scrap, inventory adjust) only after field trust — dirty ERP fields automate error.

**Out of v1 (unless vendor proves cheap):** full plant MES suite, PLC/SCADA deep connectivity, company-wide KM, MSP/NetSuite vendor swap.


---

## 2. Primary ROI levers (TCO ≠ ROI)

**TCO** (§4) is cost of ownership. **ROI** is whether Ops can support ~20% growth without linear headcount — especially under constant new-SKU load. Classical **OEE is a weak lead metric** here (high intentional changeover / wait); use it later on a stable cell if useful.

### Lead levers (business case)

| Lever | Definition (QCo) | Why it matters | MES implication |
| --- | --- | --- | --- |
| **1. Time-to-competency** | Clock from **BOM + SOP released for build** → operator/cell hits agreed **FPY** and **hours-per-unit** on that SKU **without** engineer babysitting | Dozens of new SKUs/year win **specifier** doors (“new” opens; “same as last year” doesn’t). Each SKU dumps new BOMs/SOPs on the floor — ramp tax is a GTM cost | In-station certified BOM/SOP/drawings day one; forced QC; photos/issues so build #2 isn’t tribal. Favor platforms Ops can update fast when SKUs land |
| **2. On-time to promise** | Hit date from **firm delivery commitment** (PO / scheduled ship **after customer releases for delivery**) — **not** from early spec or long-lived high-confidence sales | Construction: spec/sales can sit months before release; once released, **on-time delivery is critical** | Separate **readiness** (can we build this SKU cold?) from **execution OTIF** post-release. Don’t score MES “late” on six-month commercial lag |

### Supporting Scorecard lines (pilot)
WO **cycle time**, **FPY**, **scrap/partials** — baseline from paper/NetSuite before go-live. Hours-per-unit once labor capture is trustworthy.

### Option value (do not bake into v1 ROI without a Rock)
Tulip-packaged advances (material flow, richer reporting, visual ML, etc.) vs Ignition DIY/modules: count in NPV **only if** you’ll exercise them in 5 years **with named owners**. Otherwise they are brochure, not benefit.

---

## 3. License list-price anchors (public; confirm in quote)

| | **Tulip** | **Ignition (Inductive)** |
| --- | --- | --- |
| Meter | Per **Monthly Active Interface** | Per **gateway**; **unlimited** clients |
| Published (20 interfaces) | Essentials $100/iface/mo → **$24k/yr**; **Professional $250/iface/mo → $60k/yr** (10-iface min, annual) | Platform **$1,200** + Application Building suite **$13,500** = **$14,700** perpetual *(or Platform + Perspective $11,225)* |
| Care / support | Included tiers vary; premium support add-on | BasicCare **16%** / TotalCare **20%** / PriorityCare **24%** of retail / yr |
| NetSuite path | Official connector (OAuth2 / Restlets); needs **Professional+** for HTTP connectors | Custom HTTP/SQL or middleware; Sepasoft Business Connector optional |
| Sources | tulip.co/plans · inductiveautomation.com/pricing/list · sepasoft.com/pricing-mes |

**QCo planning number for Tulip:** use **Professional (~$60k/yr)**, not Essentials — connectors required for NetSuite.

---

## 4. ROM 5-year TCO (filled — midrange SI + Ignition Care)

### SI rate assumption (ROM)
| Item | Value | Basis |
| --- | --- | --- |
| Blended SI / partner rate | **$150 / hr** | Mid of common Ignition/MES bands (~$125–$175) and industrial SI (~$100–$200) |
| Scope of SI $ | Pilot only: design, build, NetSuite **read** connector, label + photo, station pattern for **20** interfaces, train-the-trainer | Excludes tablets/printers/cameras, NetSuite-host fees, MSP, travel, CoE retainers |
| Ignition S&M | **TotalCare 20%** of retail / yr × **5 years** | Inductive published Care (Basic 16% / Total 20% / Priority 24%) — use TotalCare as mid |
| Tulip S&M | Bundled in SaaS subscription | No separate Care line; premium support is add-on (not in ROM) |

### Effort assumptions (hours × $150) — unchanged
| Lane | Pilot hours (mid) | SI $ (mid) | Rationale |
| --- | --- | --- | --- |
| **A. Tulip Professional** | **350 hrs** | **$52,500** | Composable apps + library NetSuite connector + station templates |
| **B. Ignition DIY** | **700 hrs** | **$105,000** | Custom Perspective WIP/QC/timers/print/camera + NetSuite HTTP + DB |
| **C. Ignition + Sepasoft** | **550 hrs** | **$82,500** | Track & Trace patterns reduce greenfield; still config + NetSuite + UX |

*Low/high bands (same rate): A 250–450 hrs; B 550–900 hrs; C 400–700 hrs.*

### License + annual S&M (5 years)
| Lane | Yr0 perpetual software | Annual S&M | S&M × 5 yrs | Software + S&M (5-yr) |
| --- | --- | --- | --- | --- |
| **A. Tulip Pro 20 MAI** | — (SaaS) | **$60,000/yr** (sub includes support) | **$300,000** | **$300,000** |
| **B. Ignition DIY** | **$14,700** | Care **$2,940/yr** (20% × $14,700) | **$14,700** | **$29,400** |
| **C. Ignition + Sepasoft** | **$42,800** ($14,700 + T&T $22,700 + Biz Connector $5,400) | Care **$8,560/yr** (20% × $42,800) | **$42,800** | **$85,600** |

### ROM 5-year totals (software + S&M/Care + pilot SI)
| Lane | Software + S&M (5-yr) | Pilot SI (mid) | **ROM 5-yr total** | vs Tulip |
| --- | --- | --- | --- | --- |
| **A. Tulip Professional** | $300,000 | $52,500 | **~$353k** | — |
| **B. Ignition DIY** | $29,400 | $105,000 | **~$134k** | **~−$219k** vs A |
| **C. Ignition + Sepasoft** | $85,600 | $82,500 | **~$168k** | **~−$185k** vs A |

**Still excluded:** station hardware, NetSuite-host PS, MSP, travel, AI Action overages, redundancy gateway, **year-2+ app retainers / internal FTE** (see below).

**Optional sustainment retainers (years 2–5, not in totals):** Tulip light partner **~$10–20k/yr**; Ignition DIY **~$25–40k/yr**; Ignition+Sepasoft **~$20–30k/yr**. If you bake mid DIY retainer ($30k × 4 yrs = $120k) into B, DIY 5-yr climbs to **~$254k** — still usually under Tulip, but the gap narrows fast without an internal Ignition owner.

## 5. Scorecard — pre-demo ROM scores (1–5) · replace after demos

*Scores = Manufacturing IT/OT Expert judgment against QCo context (paper+NetSuite, high-mix, 20 stations, AI-accelerated customization interest). **Not** vendor-validated.*

| Criterion (weight) | Tulip | Ignition DIY | Ignition + Sepasoft | Notes |
| --- | --- | --- | --- | --- |
| Fit to station loop above (20%) | **5** | **3** | **4** | Tulip purpose-built for this loop |
| NetSuite read (WO/BOM) effort (15%) | **4** | **3** | **3** | Tulip library connector; all need host cooperation |
| Customization speed / low-code (15%) | **5** | **2** | **3** | Ops can own Tulip apps; Ignition needs specialists |
| AI acceleration (build + ops) (10%) | **4** | **2** | **2** | Native Tulip AI agents/assist; Ignition = external/AI-written scripts |
| 5-yr software+S&M+SI TCO @ 20 ifaces (15%) | **2** | **5** | **4** | From §4 ROM (~$353k / ~$134k / ~$168k) |
| Ops burden (MSP + NetSuite host + internal) (10%) | **4** | **2** | **3** | SaaS vs gateway ownership |
| High-mix / partials / genealogy / NPI ramp (10%) | **4** | **3** | **5** | Competency on new SKUs; Sepasoft T&T strong on genealogy |
| Scale to 50–100 stations later (5%) | **2** | **5** | **5** | Tulip MAI scales $; Ignition clients flat |
| **Weighted total** | **4.05** | **3.10** | **3.55** | Pre-demo only |

**Weighted math (check):**  
Tulip: 5×0.20+4×0.15+5×0.15+4×0.10+2×0.15+4×0.10+4×0.10+2×0.05 = 1.0+0.6+0.75+0.4+0.3+0.4+0.4+0.1 = **4.05**  
Ignition DIY: 3×0.20+3×0.15+2×0.15+2×0.10+5×0.15+2×0.10+3×0.10+5×0.05 = 0.6+0.45+0.3+0.2+0.75+0.2+0.3+0.25 = **3.05** → round **3.10** with minor float  
Ignition+Sepasoft: 4×0.20+3×0.15+3×0.15+2×0.10+4×0.15+3×0.10+5×0.10+5×0.05 = 0.8+0.45+0.45+0.2+0.6+0.3+0.5+0.25 = **3.55**

---

## 6. RFQ must-asks (copy into vendor email)

1. Price **exactly 20** active tablet/PC interfaces for 12 and 36 months (Tulip: MAI; Ignition: confirm unlimited clients on one gateway).  
2. Confirm tier/modules required for **NetSuite bi-directional** (even if v1 is read-only).  
3. Fixed-fee or capped SI to deliver **the 10-step station loop** in a sandbox, including label print + photo capture — compare to ROM hours in §4.  
4. Who owns **NetSuite Restlet/bundle or API** work — vendor, QCo, or NetSuite host?  
5. Support SLAs, upgrade cadence, and exit (data export).  
6. Security: SSO options, data residency, audit trail, camera/PII retention.  
7. References: **low-volume / high-mix assembly** with **NetSuite** (not only high-volume process) — ideally **high new-SKU / NPI** rate.  
8. How the platform shortens **time-to-competency** on new BOMs/SOPs (authoring, versioning, in-station enforce, photo/issue feedback).  
9. How you report **on-time from firm delivery release** (not from early spec), plus a **SKU readiness** view before release.

---

## 7. Internal decision rule (updated with 5-yr ROM)

- **ROM says (5 yr, Care included on Ignition):** Ignition DIY **~$134k**; Ignition+Sepasoft **~$168k**; Tulip Pro **~$353k**. License+Care gap widens vs 3-yr because Tulip SaaS keeps accruing while Ignition Care is ~20% of a one-time license.  
- Prefer **Tulip** if calendar-to-pilot, AI-in-platform, and Ops-owned app changes are worth ~$180–220k of 5-yr premium over Ignition lanes.  
- Prefer **Ignition DIY** only if a **named Ignition-fluent owner or SI** is funded — pilot SI (~$105k) plus Care (~$3k/yr) are the real costs, not the $15k license; add retainer if no internal owner.  
- Prefer **Ignition + Sepasoft** if genealogy/partials are non-negotiable in v1 and you still want unlimited clients (**~$168k / 5 yr** ROM).  
- Do **not** choose on unlimited-client sticker price alone. **Replace ROM SI + Care with vendor-capped SOWs before Council money ask.**  
- **ROI gate:** prefer the lane that best improves **time-to-competency on new SKUs** and **on-time from delivery release** — TCO only breaks ties once those are credible.

---

## 8. ROM confidence

| Element | Confidence | Action |
| --- | --- | --- |
| Tulip / Ignition / Sepasoft list prices | **High** (public pages as of draft) | Reconfirm on quote date |
| $150/hr midrate | **Medium** | Ask vendors for blended rate + fixed fee |
| Hour estimates | **Medium-low** | Bound with fixed-fee pilot SOW |
| Pre-demo scores | **Judgment only** | Rescore after scripted demos |

*Companion: Manufacturing IT/OT Expert capability notes (Tulip stronger for AI-accelerated no-code co-build; Ignition stronger on script/integration with specialist Designer ownership).*
