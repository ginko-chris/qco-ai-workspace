# Fusion Manage Validation Research Pack: Enforcing Correct Fields, Properties and Values

**For:** Chris, Director of AI and Technology, QCo
**Date:** 2026-10-07
**Status:** Research only, from public sources. Nothing was changed in any system, and we have no Fusion Manage connector or tenant to test against.
**Scope:** Autodesk Fusion Manage (FM), formerly Fusion Lifecycle (FLC) and Fusion 360 Manage, a cloud PLM (product lifecycle management) system. It covers native validation, workflow and release gates, API and external checks, limits, and a tiered plan for a thin-IT team.

---

## 1. BLUF (bottom line up front)

Fusion Manage can enforce most "right fields, right values" rules with no code. Field validators cover required, regex, ranges, min/max length, conditional-required and uniqueness. Picklists and filtered picklists restrict values to valid combinations. Derived and roll-up fields fill in values automatically, and section and workflow locks stop late edits. Those checks fire whenever someone saves an item. Rules that span several fields, the BOM (bill of materials) or related records need **Validation scripts**: server-side JavaScript that runs **only on workflow transitions** and blocks the transition with a list of messages. The natural place for them is the change order (ECO, engineering change order) **Release** step, because item lifecycle changes only happen through a change order. There is **no code hook that can reject an ordinary save** with a custom message. Scripts also have short timeouts (about 9 seconds, unverified in current docs), and imports and scripts can skip field validation. **Recommendation:** for the first product family, build Tier 1 (native configuration) plus a Tier 2 Release gate on the change order. Run Stardog SHACL (Shapes Constraint Language) as a Tier 3 audit that writes a pass/fail flag back to FM. Start Tier 3 as a warning, and make it a blocking gate only once the shapes have proven themselves.

---

## 2. Capability table

"Fires" means when the rule is checked. Sources are numbered and listed in §8.

| Mechanism | What it enforces | When it fires | Scripting? | Strengths | Weaknesses | Source |
|---|---|---|---|---|---|---|
| **Field types and implicit validators** | Data type (Integer, Float, Money, Date, Email, Checkbox carry a built-in validator that can't be removed) | Save (create/edit) | No | Free, always on | Type-level only | [1] |
| **Basic and format validators** | Required, Selection Required, Regular Expression Input Mask, Min/Max Length, Integer/Float Range, Date greater/less than another field, CSV masks | Save | No | Covers most single-field rules, such as part-number pattern or CCT (color temperature) range | Errors are generic. No custom message logic | [1] |
| **Dependency validators** | Conditionally Required (field X needed if field Y = value), All or None, At Least One, One or the Other, Identical, percentage-of-other-field | Save | No | Handles many "if dimmable then protocol required" style rules | One condition per validator, with no AND/OR logic | [1] |
| **Uniqueness validators** | Unique, Unique Combination of 2 fields, Unique With Wildcard, Unique in Grid, Unique CSV in BOM, **Unique Strict** (includes deleted records and is honored by scripts and imports) | Save / import | No | Stops duplicate items and SKUs | **Unique Strict is not available in revision-controlled workspaces**, which is where Items usually live | [1] |
| **Picklists (defined or workspace-linked)** | Values from a controlled list, or from records in another workspace | Save | No | One source of truth for option values. Workspace picklists can't gain new values through import | Defined lists can be extended by an import if the setting allows it | [2][3] |
| **Filtered picklists** | Valid *combinations* across several fields (e.g., Family → Output → CCT → CRI). Works in Item Details, Grid, BOM and Sourcing | Save / import (combinations are validated on import) | No | Closest native match to "certified configuration rules". The option matrix lives as data in a reference workspace | All fields must point to the same workspace picklist. No script can filter values dynamically | [2][3][4] |
| **Defaults** | Picklist "Show First Value as Default" and field defaults | Create (UI) | No | Reduces blanks | **Ignored when items are created by script** | [5][6] |
| **Derived fields** | Read-only copies of fields from a linked item, such as family-level specs pulled onto the SKU | Display / save | No | Removes re-keying | Can behave unexpectedly when linked to the "latest" revision of revision-controlled items | [7][8] |
| **Computed fields** | SQL-like formula (CASE WHEN, LENGTH, REGEXP_REPLACE) that displays derived values or warning badges | Save / display | Formula only | Good for "warning" indicators | Can't read linked items. Official reference page not found in current help (forum evidence only) | [9][10] |
| **Roll-up fields** | Sum, min/max, earliest/latest date, all/any checked across the BOM | BOM view / refresh | No | Watts per assembly, all-children-released checkbox | Display only; it does not block anything | [11] |
| **Auto Number** | System-generated IDs (read-only) | Create | No | Removes ID typos | Can't be imported | [12][3] |
| **Field-level access** | "Editable" set to False or Creation Only. Restricting a section to user groups hides its fields | Save | No | Lets the product director own rule fields | A restricted section with required fields blocks other users from saving. Restricted fields are hidden from imports but still visible in the Change Log | [3][13] |
| **Workflow lock state / section workflow locking** | Locks the item, or selected sections, once a workflow state is reached | State entry | No | Freezes released definitions | Users with "Admin Override Workflow Locks" can bypass it | [14][15][16] |
| **Transition settings** | Per-transition permission, required comments, password, precondition filters such as "user is in Approvers picklist" | Transition | No | No-code approval gates | Who-may checks, not data checks | [14][17] |
| **Condition script** | Shows or hides a transition (returns true/false) | Each time FM works out which transitions are available | Yes | Hides "Release" until it's ready | Read-only. Gives the user no explanation | [18][19] |
| **Validation script** | Any rule you can code: cross-field, BOM children, Managed Items, linked records. Returns an array of messages; an empty array passes | **Workflow transition only**, before the Action script. One per transition, reusable across workspaces | Yes (JS 1.5) | Precise, explains itself to the user, can read linked items and BOM (`item.boms`) | Doesn't run on save. Timeout of about 9 seconds (unverified) | [18][19][20][21] |
| **Action scripts (On Create / On Edit / On Demand / transition)** | Set or clean values, stamp flags, spawn items | Create, edit, user click, transition, escalation | Yes | Auto-normalizes data | **Can't show a validation message or block a save.** Ignores user permissions and defaults | [5][6][22][23] |
| **Revision control and lifecycle (Lifecycle Editor)** | Item phases (e.g., In Design → Production → Obsolete) change only through a change order. The "Managed state" performs revisioning | Change order reaches Managed state | No | Strong release discipline. FM checks that every affected item has a lifecycle transition before Managed state | Needs change-order setup. Bypassed by "Override Revision Control Locks" | [14][24][25][16] |
| **Managed Items (Affected Items) tab** | Which items a change order releases, plus custom columns | Change order transitions | Validation script for custom checks | Release gate can check every affected item | Rows can't be re-sorted. Custom checks need scripts | [26][24] |
| **Change Management template** | Ships with scripts such as "Change Orders WF Validations" and "Change Management Library" | Change order transitions | Pre-built | Tier 2 starts from Autodesk's own scripts | You still have to tune them to QCo rules | [27] |
| **Classification (Classification Manager)** | Class tree with class-specific properties. One classification section per Item Details form | Save | No | Class-specific attributes per family | Availability per edition not confirmed. Classification data is awkward in scripts and the API (forum) | [3][28] |
| **REST API v3** | Everything above, called *as a named service user*: "permissions, validation, workspaces… are all tied to the user" | Each call | Integration code | External checks and write-back | No full public reference. We found no rate-limit documentation | [29][30] |
| **APS webhooks (`adsk.flc.production`)** | Notifies on item.create, item.update, item.clone, item.lock, item.unlock, item.release, workflow.transition | After the event | Integration code | Triggers external audits without polling | Notification only and can't block (our inference). **Does not fire for transitions performed by scripts** | [31][32] |
| **Imports tool** | Choose "Strictly enforce all validations and constraints" or "Ignore validations… whenever possible" | Import run | No | Strict mode keeps migrations clean | Ignore mode **bypasses validators**. On Create/On Edit scripts run on import only if Support enables it (2015 note, current status unverified) | [3][33] |

---

## 3. Gotchas that shape the design

- **No custom message on save.** Field validators run on save. Custom logic runs only on transitions, and On Create and On Edit can't show an error [5][22][23]. So rules that can't be expressed as validators must be enforced at a transition: Submit, Ready for Review or Release.
- **Script timeouts.** Autodesk staff have described a 4-second limit for condition scripts and 9 seconds for validation and action scripts. The limit is server-side and not configurable [20][21][34]. Deep BOM recursion can time out [35]. A common workaround is an on-demand "pre-check" script that sets a flag, which the validation script then reads [34]. *Unverified in current official help.*
- **Scripts bypass field rules.** The engine doesn't honor user permissions [22], `createItem()` ignores defaults and required fields [6], and an On Edit script can write values that field validators would reject [23].
- **Bypass permissions exist.** These are Admin Override Workflow Locks, Override Revision Control Locks, Run Imports and Setup Administration [16]. Limit them to one or two named admins.
- **Imports.** Make "Strictly enforce" the policy, and remember Auto Number and Computed fields can't be mapped [3].
- **Maintenance.** Scripting is part of the **Setup Administration** permission. Autodesk recommends limiting it to admins with some programming background [22]. Script changes show up in the Setup Log [36]. We found no built-in script versioning, so export scripts to git (the community `plm-utilities` tool can extract tenant configuration and scripts [37]).
- **Sandbox and licensing.** **Fusion Manage Enterprise** adds "a sandbox environment for safe testing", SSO (single sign-on), external collaboration and more storage. Standard has none of these [38][39]. Sandbox changes must be re-created by hand in production; there is no promote tool [40]. Whether scripting and the API differ by edition is *unverified*. Our reading is that both editions include them.
- **Naming confusion.** The "Fusion Manage Extension" (PLM inside Fusion design hubs) went end-of-sale on Nov 7, 2025 and has been folded into Fusion Manage [41]. The GitHub **plm-extensions** project is a community Node.js app suite built on the FM REST APIs. Its README says it is "not an official Autodesk product" [42]. We found no Autodesk "Tulip-style" app builder for FM.

---

## 4. Tiered approach for a thin-IT SMB

| Tier | What | Rules it carries for QCo | Who maintains |
|---|---|---|---|
| **Tier 1: Native configuration, no code** | Item Details layout per family, field validators, filtered picklists driven by a **Family Option Matrix** reference workspace, derived fields from a Family record, roll-ups on the BOM, Auto Number, section locks, revision-controlled Items and the Change Management template | Required spec fields (CCT, CRI, lumen/ft, W/ft, voltage, cut increment, max run, IP rating, finish, lens, mount). Regex on part numbers. Ranges. Valid option combinations. "If dimmable then protocol required". Duplicate prevention | **FM admin:** a named internal power user trained by the reseller. **Product director** owns option-matrix *data* (adding a valid combination is a record edit, not a config change). **Product Development** owns MBOM (manufacturing BOM) fields |
| **Tier 2: Validation scripts on release and ECO transitions** | One "Release Gate" validation script on the change order's Submit and Release transitions, plus a library of reusable checks. Start from Autodesk's "Change Orders WF Validations" | For each affected item: family spec complete; values within the family's certified envelope (e.g., max run × W/ft ≤ driver capacity); BOM children released or also on this change order; no obsolete children; MBOM present where required; Stardog flag = PASS (once Tier 3 is a hard gate) | **Reseller or partner on a small retainer** writes the scripts. The internal FM admin owns the rules list and tests in the sandbox. Scripts are exported to git. Keep scripts short and bounded because of the timeout |
| **Tier 3: External audit through the API and SHACL** | A nightly job, plus an on-webhook run for `workflow.transition` into "Ready for Release". It pulls items and BOMs over REST v3 as a read-mostly service user, maps them to the OWL product model, and runs `VALIDATE` in Stardog [43]. It writes back `SHACL_STATUS` (PASS/FAIL/STALE), `SHACL_CHECKED_AT` and a report link into a section that users can't edit | Cross-family and cross-system rules FM can't express: consistency with CPQ (configure-price-quote) rule sets, NetSuite commercial identity alignment, ontology-level constraints, and portfolio-wide audits. Pricing stays in NetSuite/CPQ | **Chris's AI/KG team** owns the shapes and mappings. **MSP** hosts and monitors the job. A vaulted APS secret is tied to a dedicated FM service user |

**How Tier 3 blocks a release (design proposal, not an FM feature).** A webhook can't block a transition. Instead, the Tier 2 validation script refuses Release unless `SHACL_STATUS = PASS`. An On Edit action script resets the flag to STALE when the spec changes, so a stale PASS can't slip through. Watch for loops: the write-back is itself an edit. The On Edit script must ignore edits made by the service user, and whether a script can reliably detect the editing user is *unverified*. Test this in the sandbox.

---

## 5. How this fits the Stardog/SHACL gate

FM enforces rules where engineers work, at save and at Release. Stardog SHACL is the cross-system auditor that FM can't host, since FM has no SHACL or OWL support. Keep one rule catalog with an "enforced in" column (FM validator, FM script or SHACL shape) so that each rule has a single home. Rules duplicated on purpose, as a backstop, are marked as such. The flag write-back lets SHACL become a hard release gate without FM needing to understand RDF (Resource Description Framework), the graph data format Stardog stores.

---

## 6. Decision asks for Chris

1. **Gate design:** approve Tier 1 + Tier 2 as the release gate for family #1, with Tier 3 SHACL running **advisory first**. It becomes blocking via the flag after about 30 days with no false positives.
2. **Owners:** confirm the product director owns the property spec and Family Option Matrix data, Product Development owns MBOM rules, and name **one internal FM admin**, with a reseller retainer for scripts.
3. **Edition:** buy **Enterprise** for the sandbox (the only Autodesk-documented safe test path), or accept testing scripts in production on Standard. This should be settled before the 90-day clock starts.
4. **Import and bypass policy:** strict-mode imports only. "Run Imports", "Override Revision Control Locks" and "Admin Override Workflow Locks" limited to two named people. Decide whether to ask Support to enable scripts on import.
5. **Rule catalog home:** one rule catalog, kept in git or Stardog, with an "enforced in" column. This is the bridge between CPQ rules, FM configuration and SHACL shapes.

---

## 7. First milestone (about 6 weeks, sandbox first)

- **Scope:** one mid-complexity linear LED family, its SKUs and one MBOM.
- **Property spec v1:** at most 25 fields, each with type, validator, picklist source, owner and "enforced in". Valid option combinations go into the Family Option Matrix workspace.
- **Release gate:** change order Release validation script with 5–8 checks: spec complete, envelope math, BOM children released, no obsolete children, MBOM present.
- **Audit:** nightly SHACL run on the same family, writing back `SHACL_STATUS` as advisory only.
- **Acceptance:** a deliberately bad item (wrong CCT/CRI combination, missing driver, unreleased child) is blocked at save or at Release with a clear message. A clean item releases in one pass. The scripts run comfortably under the timeout on the largest BOM. A strict-mode re-import of 20 legacy SKUs passes or fails as expected.

---

## 8. Sources

Official Autodesk sources are marked **[A]**. Forum posts by Autodesk staff are marked **[A-staff]**. Everything else is community or partner material.

1. [A] Field Validators — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-D03FBA60-91DB-4294-8B2F-7E21FC7D2194.htm
2. [A] Picklist Fields — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-B6B00500-31AF-4D72-99C4-639F2FE8AEFF.htm
3. [A] Import Item Details — https://help.autodesk.com/cloudhelp/ENU/PLM-360-User/files/UG-IMPORT-ITEMDETAILS.htm
4. Forum, filtered picklists and precondition filters — https://forums.autodesk.com/t5/fusion-manage-forum/picklist/td-p/11997684
5. Forum, validation can't run in OnCreate/OnEdit — https://forums.autodesk.com/t5/fusion-manage-forum/can-a-validation-be-performed-on-oncreate-or-onedit-scripts/td-p/7733868
6. Forum, defaults and required fields ignored by `createItem` — https://forums.autodesk.com/t5/fusion-manage-forum/item-details-fields-with-default-or-required-values-are-always/td-p/11566871
7. [A] Derived Fields — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-811A28B2-E80F-4794-AE2E-22DC2F0C430F.htm
8. Forum, derived fields and revision-controlled picklists — https://forums.autodesk.com/t5/fusion-manage-forum/derived-fields-gt-error-derived-value-xxxxx-does-not-match-the/td-p/13442044
9. Forum, computed field formulas — https://forums.autodesk.com/t5/fusion-manage-forum/computed-field-functions-to-count-number-of-characters-in-field/td-p/13122407
10. Ideas, computed fields can't read linked items — https://forums.autodesk.com/t5/fusion-manage-ideas/allow-computed-fields-to-access-fields-of-linked-items/idi-p/7941643
11. [A] Roll-up Fields — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-E1086B11-0EB9-4E75-BAE8-3121E366C17C.htm
12. [A] Field Types — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-02815D7F-2697-4F83-8DDA-896202805676.htm
13. [A] Configure the Item Details tab (restricted sections, classification) — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-C2A1AEAB-227E-4A89-8972-CE779B7AFFA9.htm
14. [A] Workflow editor (lock and managed states, transition properties) — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-9F82E382-A25D-4917-8366-FF0995AD08B6.htm
15. [A] Section Workflow Locking (support article) — https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/How-does-Section-Workflow-Locking-work-in-Fusion-Manage.html
16. [A] Workspace Permissions — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/GUID-F3C22AD5-FC88-4752-BA6B-54051A11E0C7.htm
17. [A] Workflow management — https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-ADMIN-WORKFLOW-MGMT.htm
18. [A] Scripting Basics — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Scripting/files/DEV-SCRIPTING-BASICS.htm
19. [A] Autodesk Developer Blog, Condition and Validation scripts — https://blog.autodesk.io/script-types-condition-and-validation-script/
20. [A-staff] Forum, 4 s / 9 s timeouts — https://forums.autodesk.com/t5/fusion-manage-forum/script-timeouts-amp-bad-internet-connections/td-p/6011944
21. [A-staff] Forum, "the 9s limit" — https://forums.autodesk.com/t5/fusion-manage-forum/time-out-error-when-looping-through-project-children/td-p/8077876
22. [A] Scripting Basics (permissions and security) — same URL as 18
23. Ideas, scripts can bypass field validations; script-triggered transitions — https://forums.autodesk.com/t5/fusion-manage-ideas/allow-script-triggered-workflow-transitions-to-follow-validation/idi-p/9307177
24. [A-staff] Ideas, lifecycle checked before Managed state; lifecycle only via change order — https://forums.autodesk.com/t5/fusion-manage-ideas/check-lifecycle-from-items-in-co/idi-p/11438805
25. [A] Change management: lifecycle transitions — https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-CHGMGMT-LIFECYCLE-TRANSITIONS.htm
26. [A] Configure the Managed Items tab — https://help.autodesk.com/cloudhelp/ENU/PLM-360-Admin/files/ADM-WS-MANAGED-ITEMS-TAB.htm
27. [A] Change management administration (template scripts) — https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-ADMIN-CHGMGMT.htm
28. Forum, classification fields via v3 API — https://forums.autodesk.com/t5/fusion-manage-forum/v3-api-get-amp-put-item-classifications-and-related-fields/td-p/10656232
29. [A] REST v3 2-legged OAuth tutorial — https://help.autodesk.com/cloudhelp/ENU/FLC-RestAPI/files/FLC_RestAPI_v3_API_2_legged_Tutorial_html.htm
30. [A-staff] Forum, v3 create item and field metadata — https://forums.autodesk.com/t5/fusion-manage-forum/v3-api-post-create-workspace-item/td-p/10313348
31. [A] APS Webhooks, Fusion Lifecycle `workflow.transition` and event list — https://aps.autodesk.com/en/docs/webhooks/v1/reference/events/flc_events/workflow.transition
32. [A-staff] Forum, webhooks don't fire on script transitions — https://forums.autodesk.com/t5/fusion-manage-forum/forge-webhook-not-being-triggered-on-item-state-transition-by/td-p/10107773
33. [A-staff] Forum, scripts on import by request — https://forums.autodesk.com/t5/fusion-manage-forum/activate-scripts-running-on-import/td-p/6235089
34. Ideas, configurable timeout; on-demand pre-check flag pattern — https://forums.autodesk.com/t5/fusion-manage-ideas/making-scripting-timeout-configurable/idi-p/5835471
35. [A-staff] Forum, BOM recursion and timeouts — https://forums.autodesk.com/t5/fusion-manage-forum/scrip-bom-review-all-sub-levels/td-p/8542992
36. [A] System configuration (Setup Log, Lifecycle Editor, Picklist Manager) — https://help.autodesk.com/cloudhelp/ENU/Fusion-Manage/files/MNG-SYSCONFIGS.htm
37. dickmans/plm-utilities (community) — https://github.com/dickmans/plm-utilities
38. [A] Fusion Manage product page (Enterprise adds sandbox) — https://www.autodesk.com/products/fusion-manage/overview
39. Reseller (Novedge), Enterprise vs Standard — https://novedge.com/products/buy-fusion-manage-enterprise-subscription
40. Ideas, sandbox changes re-done by hand — https://forums.autodesk.com/t5/fusion-manage-ideas/updates-from-sandbox-to-prod-system/idi-p/12064092
41. [A] Fusion Manage Extension end-of-sale FAQ — https://damassets.autodesk.net/content/dam/autodesk/www/pdfs/fusion-manage-extension-and-manage-unification-customer-faq.pdf
42. dickmans/plm-extensions (community) — https://github.com/dickmans/plm-extensions
43. Stardog, Data Quality Constraints (SHACL, VALIDATE, guard mode) — https://docs.stardog.com/data-quality-constraints

**iPaaS (integration platform as a service) and middleware options found:**
- **Jitterbit Design Studio** has an Autodesk Fusion Lifecycle connector for Get, Create, Update, Upsert and Delete — https://docs.jitterbit.com/design-studio/design-studio-reference/connectors/autodesk-fusion-lifecycle-connector/
- **vdR Nexus** offers a Fusion Manage ↔ NetSuite integration (partner claims: Built for NetSuite and Autodesk ADN) — https://www.vdr.com/autodesk-fusion-manage-integration-with-oracle-netsuite
- **Workato:** no dedicated FM connector found, so it would need the HTTP or custom connector — https://forums.autodesk.com/t5/fusion-manage-forum/autodesk-fusion-lifecycle-with-workato/td-p/8857072
- **Celigo:** no FM connector found (unverified).

## 9. Unverified or partially verified

- The 4 s / 9 s script timeouts come from Autodesk staff forum posts (2016, 2018) and are not in current official help. They may have changed.
- Scripts running on import "by request" comes from a 2015 note. Current behavior is unknown.
- Scripting and API entitlements by edition (Standard vs Enterprise), sandbox refresh process and cadence, and API rate limits were not found.
- The official computed-field reference was not found; the formula syntax is from forum evidence. Classification availability per edition is not confirmed.
- That validation scripts run on REST-API transitions is inferred from Autodesk staff statements and the "validation tied to the user" line in the API tutorial. It was not stated explicitly for validation scripts.
- That script-triggered transitions skip validation scripts is inferred from an Ideas post.
- That webhooks can't block transitions is our inference (they are post-event notifications).
- Whether field validators apply to BOM-tab and Grid fields the same way as Item Details is not confirmed.
- Whether an On Edit script can reliably detect the service user (to avoid write-back loops) is untested.
