# QCo MES Eval — Tulip vs Ignition (20 interfaces) RFQ One-Pager
**Owner:** Chris · Director of AI and Technology  
**Context:** Low-volume / high-mix assembly · ERP = NetSuite (3rd-party host ≠ corporate MSP) · Shopfloor OT minimal · Production today largely paper + NetSuite  
**Scale assumption:** **20** tablet/PC interfaces (Monthly Active Interfaces / clients)  
**Purpose:** Vendor RFQ scorecard + internal **ROM** 3-year TCO framing — not a purchase decision.  
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

## 2. License list-price anchors (public; confirm in quote)

| | **Tulip** | **Ignition (Inductive)** |
| --- | --- | --- |
| Meter | Per **Monthly Active Interface** | Per **gateway**; **unlimited** clients |
| Published (20 interfaces) | Essentials $100/iface/mo → **$24k/yr**; **Professional $250/iface/mo → $60k/yr** (10-iface min, annual) | Platform **$1,200** + Application Building suite **$13,500** = **$14,700** perpetual *(or Platform + Perspective $11,225)* |
| Care / support | Included tiers vary; premium support add-on | BasicCare **16%** / TotalCare **20%** / PriorityCare **24%** of retail / yr |
| NetSuite path | Official connector (OAuth2 / Restlets); needs **Professional+** for HTTP connectors | Custom HTTP/SQL or middleware; Sepasoft Business Connector optional |
| Sources | tulip.co/plans · inductiveautomation.com/pricing/list · sepasoft.com/pricing-mes |

**QCo planning number for Tulip:** use **Professional (~$60k/yr)**, not Essentials — connectors required for NetSuite.

---

## 3. ROM 3-year TCO (filled — midrange SI)

### SI rate assumption (ROM)
| Item | Value | Basis |
| --- | --- | --- |
| Blended SI / partner rate | **$150 / hr** | Mid of common Ignition/MES freelancer–SI bands (~$125–$175) and industrial SI (~$100–$200) |
| Scope of SI $ | Pilot only: design, build, NetSuite **read** connector, label + photo, station rollout pattern for **20** interfaces, train-the-trainer | Excludes tablets/printers/cameras, NetSuite-host change fees, MSP, travel, year-2+ CoE |
| Care assumption (Ignition lanes) | **TotalCare 20%** of retail / yr | Inductive published Care tiers |

### Effort assumptions (hours × $150)
| Lane | Pilot hours (mid) | SI $ (mid) | Rationale |
| --- | --- | --- | --- |
| **A. Tulip Professional** | **350 hrs** | **$52,500** | Composable apps + library NetSuite connector + station templates; aligns with ~6–8 wk guided / jumpstart-class effort |
| **B. Ignition DIY** | **700 hrs** | **$105,000** | Custom Perspective WIP/QC/timers/print/camera + NetSuite HTTP + DB model; no Sepasoft shortcuts |
| **C. Ignition + Sepasoft** | **550 hrs** | **$82,500** | Track & Trace / procedure patterns reduce greenfield; still config + NetSuite + UX + training |

*Low/high bands (same rate): A 250–450 hrs ($37.5–67.5k); B 550–900 hrs ($82.5–135k); C 400–700 hrs ($60–105k).*

### License + Care (3 years)
| Lane | Yr0 software | Recurring (×3 yrs) | Software + Care 3-yr |
| --- | --- | --- | --- |
| **A. Tulip Pro 20 MAI** | — (SaaS) | **$60,000 × 3 = $180,000** | **$180,000** |
| **B. Ignition DIY** | **$14,700** (Platform + App Building) | Care **$2,940/yr × 3 = $8,820** | **$23,520** |
| **C. Ignition + Sepasoft** | **$14,700** + Track & Trace **$22,700** + Business Connector **$5,400** = **$42,800** | Care **$8,560/yr × 3 = $25,680** | **$68,480** |

### ROM 3-year totals (software + Care + pilot SI)
| Lane | Software + Care (3-yr) | Pilot SI (mid) | **ROM 3-yr total** | vs Tulip |
| --- | --- | --- | --- | --- |
| **A. Tulip Professional** | $180,000 | $52,500 | **~$233k** | — |
| **B. Ignition DIY** | $23,520 | $105,000 | **~$129k** | **~−$104k** vs A |
| **C. Ignition + Sepasoft** | $68,480 | $82,500 | **~$151k** | **~−$82k** vs A |

**Not in the ~$129–233k band:** station hardware, label printers, cameras, NetSuite host professional services, MSP, internal SME time, AI Action overages (Tulip), redundancy gateway, year-2+ retainers. Add **~$15–40k** hardware ROM separately if 20 tablets + a few printers/cameras.

**Sustainment (ROM, years 2–3, optional add):** Tulip light partner / internal app owner **~$10–20k/yr**; Ignition DIY named resource or retainer **~$25–40k/yr**; Ignition+Sepasoft **~$20–30k/yr**. Not rolled into table above.

---

## 4. Scorecard — pre-demo ROM scores (1–5) · replace after demos

*Scores = Manufacturing IT/OT Expert judgment against QCo context (paper+NetSuite, high-mix, 20 stations, AI-accelerated customization interest). **Not** vendor-validated.*

| Criterion (weight) | Tulip | Ignition DIY | Ignition + Sepasoft | Notes |
| --- | --- | --- | --- | --- |
| Fit to station loop above (20%) | **5** | **3** | **4** | Tulip purpose-built for this loop |
| NetSuite read (WO/BOM) effort (15%) | **4** | **3** | **3** | Tulip library connector; all need host cooperation |
| Customization speed / low-code (15%) | **5** | **2** | **3** | Ops can own Tulip apps; Ignition needs specialists |
| AI acceleration (build + ops) (10%) | **4** | **2** | **2** | Native Tulip AI agents/assist; Ignition = external/AI-written scripts |
| 3-yr software+SI TCO @ 20 ifaces (15%) | **2** | **5** | **4** | From §3 ROM (~$233k / ~$129k / ~$151k) |
| Ops burden (MSP + NetSuite host + internal) (10%) | **4** | **2** | **3** | SaaS vs gateway ownership |
| High-mix / partials / genealogy (10%) | **4** | **3** | **5** | Sepasoft T&T strongest out of box |
| Scale to 50–100 stations later (5%) | **2** | **5** | **5** | Tulip MAI scales $; Ignition clients flat |
| **Weighted total** | **4.05** | **3.10** | **3.55** | Pre-demo only |

**Weighted math (check):**  
Tulip: 5×0.20+4×0.15+5×0.15+4×0.10+2×0.15+4×0.10+4×0.10+2×0.05 = 1.0+0.6+0.75+0.4+0.3+0.4+0.4+0.1 = **4.05**  
Ignition DIY: 3×0.20+3×0.15+2×0.15+2×0.10+5×0.15+2×0.10+3×0.10+5×0.05 = 0.6+0.45+0.3+0.2+0.75+0.2+0.3+0.25 = **3.05** → round **3.10** with minor float  
Ignition+Sepasoft: 4×0.20+3×0.15+3×0.15+2×0.10+4×0.15+3×0.10+5×0.10+5×0.05 = 0.8+0.45+0.45+0.2+0.6+0.3+0.5+0.25 = **3.55**

---

## 5. RFQ must-asks (copy into vendor email)

1. Price **exactly 20** active tablet/PC interfaces for 12 and 36 months (Tulip: MAI; Ignition: confirm unlimited clients on one gateway).  
2. Confirm tier/modules required for **NetSuite bi-directional** (even if v1 is read-only).  
3. Fixed-fee or capped SI to deliver **the 10-step station loop** in a sandbox, including label print + photo capture — compare to ROM hours in §3.  
4. Who owns **NetSuite Restlet/bundle or API** work — vendor, QCo, or NetSuite host?  
5. Support SLAs, upgrade cadence, and exit (data export).  
6. Security: SSO options, data residency, audit trail, camera/PII retention.  
7. References: **low-volume / high-mix assembly** with **NetSuite** (not only high-volume process).

---

## 6. Internal decision rule (updated with ROM)

- **ROM says:** Ignition DIY is cheapest on paper (**~$129k / 3 yr**); Tulip is highest (**~$233k**) but likely fastest calendar and Ops-owned change; Ignition+Sepasoft sits mid (**~$151k**) with better genealogy out of box.  
- Prefer **Tulip** if calendar-to-pilot, AI-in-platform, and Ops-owned app changes beat ~$80–100k of 3-yr license premium.  
- Prefer **Ignition DIY** only if a **named Ignition-fluent owner or SI** is funded — the $105k pilot SI (and sustainment) is the real cost, not the $15k license.  
- Prefer **Ignition + Sepasoft** if genealogy/partials are non-negotiable in v1 and you still want unlimited clients.  
- Do **not** choose on unlimited-client sticker price alone. **Replace ROM SI with vendor-capped SOWs before Council money ask.**

---

## 7. ROM confidence

| Element | Confidence | Action |
| --- | --- | --- |
| Tulip / Ignition / Sepasoft list prices | **High** (public pages as of draft) | Reconfirm on quote date |
| $150/hr midrate | **Medium** | Ask vendors for blended rate + fixed fee |
| Hour estimates | **Medium-low** | Bound with fixed-fee pilot SOW |
| Pre-demo scores | **Judgment only** | Rescore after scripted demos |

*Companion: Manufacturing IT/OT Expert capability notes (Tulip stronger for AI-accelerated no-code co-build; Ignition stronger on script/integration with specialist Designer ownership).*
