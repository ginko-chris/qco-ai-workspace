# QCo Certified Slices — Minimums Checklist (1-pager)
**Owner:** QCo Analyst · **For:** Director of AI Technology / pilot owners  
**Use:** Gate for sources entering the **agent path**. Not a burden on every Teams/SharePoint file — only the **critical few** that accuracy rides on.

---

## Principle
**Certify the critical few; leave the rest out of the agent path.**  
That de-noises knowledge for AI teammates and concentrates stewardship on consequential sources only. Humans may still browse the wider ad-hoc landscape; agents answer from certified slices (plus governed metrics/SoR tools).

---

## IN — a source may enter the agent path only if all are true

### A. Criticality
- [ ] Accuracy of an approved AI initiative **rides on** this source (wrong answer would mislead a decision, customer, or compliance outcome)
- [ ] Tagged to a named **initiative** (pilot / use case ID)
- [ ] Explicitly **in scope** for that initiative’s agent answers (not “nice to have”)

### B. Ownership & freshness
- [ ] Named **human owner** (role + person) accountable for correctness
- [ ] **Last-reviewed date** set; review cadence agreed (e.g. 30/90 days by volatility)
- [ ] **Invalidation path** defined: owner (or system) can mark stale / tombstone; agent path must drop it on next sync
- [ ] Version or `last_updated` visible to consumers (humans + agents)

### C. Identity & permissions
- [ ] **Query-time ACL pre-filter** using **end-user** (or on-behalf-of) identity — no privileged “see all” service account for retrieve
- [ ] Source ACLs reviewed for **oversharing** before indexing (search will amplify bad ACLs)
- [ ] Offboarding / revoke tested: loss of access → not retrievable within agreed SLO

### D. Answer contract (anti–cognitive surrender)
- [ ] Agent must return **citations** (link / doc ID / metric ID) for claims from this slice
- [ ] Agent must **refuse or escalate** when evidence is missing, conflicting, or past review date
- [ ] Narrative docs are **not** used as authority for KPIs (route numbers to semantic layer / SoR)

### E. Naming (good enough, consistent)
- [ ] Stable path or title convention, e.g. `dept / initiative / doc-type / yyyy-mm` (or equivalent metadata fields)
- [ ] No duplicate “official” copies without a single canonical pointer

---

## OUT of agent path (explicit)
- General Teams chatter, drafts, working folders, meeting dumps
- Department file stores with no owner / no review date
- Uncertified SaaS exports and one-off spreadsheets used as “the number”
- Stale pages kept “for history” without tombstone
- Anything an agent would need a **service superuser** identity to read
- Agent memory / chat history treated as company SoT
- Free-form Text-to-SQL against raw schemas for production KPIs

*Humans can still use OUT sources in normal work. Agents must not ground answers on them.*

---

## Dual-lane reminder (per initiative)
| Lane | Certified slice type | Example |
| --- | --- | --- |
| Narrative retrieve | Policy, runbook, approved FAQ, controlled SharePoint/Teams wiki page | “How do we process X?” |
| Semantic / SoR | Certified metric or live transactional read under ACL | “What is Y this month?” |

Each approved AI initiative adds **only** its critical few to these lanes, then repeats.

---

## Pilot gate (before go-live)
- [ ] Checklist complete for every included source
- [ ] Permission-boundary test (User A never retrieves Doc B)
- [ ] Stale/tombstone test
- [ ] Citation + refuse behavior spot-checked on 10 representative questions

---

*Companion to:* `/workspace/research/QCo-Knowledge-Systems-Options-Memo.md`
