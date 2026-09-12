# QCo MES Eval — Tulip vs Ignition (20 interfaces) RFQ One-Pager
**Owner:** Chris · Director of AI and Technology  
**Context:** Low-volume / high-mix assembly · ERP = NetSuite (3rd-party host ≠ corporate MSP) · Shopfloor OT minimal · Production today largely paper + NetSuite  
**Scale assumption:** **20** tablet/PC interfaces (Monthly Active Interfaces / clients)  
**Purpose:** Vendor RFQ scorecard + internal 3-year TCO framing — not a purchase decision.

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

## 3. Three TCO lanes (vendors must fill blanks)

Quote **software + required modules + Care/support + estimated SI for pilot** on a **3-year** horizon. Do not bury SI.

| Lane | Software (list / quote) | Yr 2–3 recurring | SI / config (pilot) | 3-yr total (fill) |
| --- | --- | --- | --- | --- |
| **A. Tulip Professional** — 20 MAI | ~$60k/yr list | Same + AI/Automation overages if any | $_____ | $_____ |
| **B. Ignition DIY MES** — Platform + App Building (or Perspective), custom station apps, NetSuite via HTTP | ~$14.7k once + Care ~$2.4–3.5k/yr | Care only | $_____ *(usually dominates)* | $_____ |
| **C. Ignition + Sepasoft** — e.g. Track & Trace (~$22.7k site list) ± Business Connector; still unlimited clients | Ignition + Sepasoft list | Care on modules | $_____ | $_____ |

*Hardware (tablets, label printers, cameras), NetSuite host change fees, and MSP work are **extra** in all lanes — ask vendors to itemize assumptions.*

---

## 4. Scorecard (1–5; weight in parentheses) — score after demos

| Criterion (weight) | Tulip | Ignition DIY | Ignition + Sepasoft | Notes |
| --- | --- | --- | --- | --- |
| Fit to station loop above (20%) | | | | Time-to-click-path in demo |
| NetSuite read (WO/BOM) effort (15%) | | | | OAuth, Restlet/bundle ownership |
| Customization speed / low-code (15%) | | | | Who can change apps after go-live? |
| AI acceleration (build + ops) (10%) | | | | In-platform vs external only |
| 3-yr software TCO @ 20 ifaces (15%) | | | | Use filled table §3 |
| Ops burden (MSP + NetSuite host + internal) (10%) | | | | Gateway, upgrades, IdP, backups |
| High-mix / partials / genealogy (10%) | | | | Serial/lot on subcomponents |
| Scale to 50–100 stations later (5%) | | | | Linear MAI vs flat clients |

**Weighted total** | | | | |

---

## 5. RFQ must-asks (copy into vendor email)

1. Price **exactly 20** active tablet/PC interfaces for 12 and 36 months (Tulip: MAI; Ignition: confirm unlimited clients on one gateway).  
2. Confirm tier/modules required for **NetSuite bi-directional** (even if v1 is read-only).  
3. Fixed-fee or capped SI to deliver **the 10-step station loop** in a sandbox, including label print + photo capture.  
4. Who owns **NetSuite Restlet/bundle or API** work — vendor, QCo, or NetSuite host?  
5. Support SLAs, upgrade cadence, and exit (data export).  
6. Security: SSO options, data residency, audit trail, camera/PII retention.  
7. References: **low-volume / high-mix assembly** with **NetSuite** (not only high-volume process).

---

## 6. Internal decision rule (draft)

- Prefer **Tulip** if calendar-to-pilot and Ops-owned changes dominate, and ~$60k/yr is acceptable vs SI risk.  
- Prefer **Ignition** if 3-yr license+Care savings fund a named Ignition-fluent owner (or trusted SI) **and** SCADA/IIoT is a near-term Rock.  
- Do **not** choose Ignition solely because “unlimited clients are cheaper” without a filled SI line in §3.

---

*Companion: capability notes from Manufacturing IT/OT Expert (Tulip stronger for AI-accelerated no-code co-build; Ignition stronger on script/integration with specialist Designer ownership). Update list prices when vendors return quotes.*
