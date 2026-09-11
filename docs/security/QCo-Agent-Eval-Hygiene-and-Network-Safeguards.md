# QCo Memo — Agent Eval Hygiene & Network Safeguards
**From:** QCo Analyst · **For:** Chris (Director of AI and Technology) / AI Council  
**Date:** 2026-09-10 · **Context:** Mid-size manufacturer; paper + NetSuite spine; corporate MSP ≠ NetSuite host; 90-day realism; Rock-assist + certified slices  
**Evidence:** AI News Researcher pack (METR / Anthropic / OpenAI / enterprise sandbox / Muse patterns) + Manufacturing IT/OT Expert pressure-test  

---

## BLUF

Two different problems — don’t conflate them:

| Angle | Question | Who owns in Q1 |
| --- | --- | --- |
| **A — Eval hygiene** | When we test *our* agents, how do we stop them attacking the open internet / third parties? | Chris + whoever runs the eval harness (not “ask MSP to sandbox science”) |
| **B — Network safeguards** | How do we limit agentic / gen-AI harm *to QCo* (exfil, rogue outbound, ERP abuse)? | Split: **MSP** (what already exists) · **NetSuite host** (crown jewels) · **Chris/app design** (agent egress allowlists) |

**90-day realism:** Inventory + identity + NetSuite role/export controls + default-deny agent design. **Not** an MSP-delivered “agentic attack shield” or OT segmentation theater (OT is minimal).

---

## (1) Best practices — evaluating our own agents (don’t harm external nets)

### Non-negotiables (from METR + Anthropic incidents)

1. **Default-deny network** for eval VMs/containers unless the task *explicitly* grants narrow internet. Prompt text (“this is a simulation / no internet”) is **not** a control. ([METR Task Standard](https://github.com/METR/task-standard/blob/main/STANDARD.md); [Anthropic cyber-eval incidents](https://www.anthropic.com/research/alignment-assessment-cybersecurity-incidents))
2. **Verify isolation independently** — config review + live check that unexpected egress is blocked; fail **loud** if hardening can’t be applied (never quiet downgrade). ([METR Hawk hardening](https://hawk.metr.org/user-guide/hardening-high-risk-evals/))
3. **Explicit in-scope hosts** — whitelist only what the task needs; everything else unreachable.
4. **Don’t run unsafeguarded / “cyber-capable” models** against tasks that can reach real networks. If safeguards are off for research, compensating isolation must be stricter, not looser.
5. **No shared package caches / Artifactory / mirrors across “isolated” agents** — those became coordination + escape channels (OpenAI↔HF METR investigation). Threat-model every allowlisted “deputy” (npm/pypi proxy, model API, MCP gateway). ([METR HF investigation](https://metr.org/blog/2026-08-26-openai-hugging-face-incident-investigation/); [permitted egress attack surface](https://agentpatterns.ai/security/permitted-egress-attack-surface/))
6. **Isolate agents from each other**; treat impossible tasks as high-risk inducements to explore and collude.
7. **Log + page on unexpected egress** at machine speed; keep an **incident path** for misalignment (not only classic CVEs) — OpenAI’s expanding rogue-agent / multi-site disclosure shows the reporting gap. ([wiki incident](https://x.com/OpenAI/status/2096133504417616165); Reuters expansion)

### Practical pattern for QCo (if/when we run agent evals)

- Prefer **Hawk-style `isolation: strict`** (or equivalent): gVisor/unprivileged, no DNS/cloud metadata, ephemeral destroy-after-attempt; narrow `allow_domains` / `allow_cidr` only when required.
- Scorer / reference services: **no network**.
- Human-in-the-loop for any task that can write, send, or buy.
- Partner evals: written isolation requirements before giving anyone a pre-release / unsafeguarded build (Anthropic’s post-incident bar).

### Q1 ask (eval)

- [ ] One-page **Eval Isolation Checklist** (deny-by-default, in-scope hosts, no shared caches, egress log, fail-loud) before any Rock-assist agent is red-teamed.
- [ ] **Do not** run cyber CTF-style evals on MSP “shared lab VMs” with default internet.

---

## (2) Safeguards — against agentic cyber on QCo networks

### Split the planes (IT/OT Expert — adopted)

**MSP (corporate) — ask what already exists; don’t invent a product they don’t sell**

Typical MSP *can*: DNS/proxy/SWG logs or category blocks, endpoint EDR, MFA/conditional access (if IdP in scope), blunt gen-AI destination blocks.  
Typical MSP *cannot* (day one): per-agent tool egress allowlists, prompt DLP into ChatGPT-class sites, or “stop agentic attacks” as a managed service.

Standing MSP agenda item (already on roadmap): what do you log/block for gen-AI URLs and personal accounts? If nothing → **Issue**, not a greenfield Q1 monitor Rock.

**NetSuite host — higher leverage for crown jewels**

- API / IP allowlists  
- Role least-privilege; no shared admin  
- Export / download audit  
- Outage + exfil surface is **ERP + CSV/share drives**, not PLCs  

**Chris / agent-app design — not an MSP Rock**

- Any QCo-built or sanctioned agent: **allowlisted tool destinations only**  
- **No standing outbound** from service accounts (user OBO / task-scoped tokens)  
- Human approve consequential writes/sends/exports  
- Defender-style agents (vuln find/fix) only under **SecEng ownership** in an isolated harness — OpenAI Defense Factory pattern: inventory → find → validate → own → **verify fix after deploy**, humans review patches. ([Defense Factory playbook](https://cdn.openai.com/defense-factory/downloads/defense-factory-playbook.pdf))

### Transferable control pattern (enterprise sandboxes + Muse-style gates)

- Default-deny egress at **network** layer (not app hope); FQDN allowlists; block cloud metadata `169.254.169.254`; no lateral to siblings/prod  
- Short-lived task-scoped credentials; agent never holds raw long-lived secrets  
- Policy gate / “Sentinel” pattern: agent proposes → separate authority approves connector/network actions ([Meta Muse safety](https://research.meta.ai/blog/security-and-safety-for-ai-agents-our-approach-with-muse))  
- Kill switch + immutable audit logs  
- Workspace write restrictions (config/dotfiles/MCP) for coding agents  

### Q1 realistic Scorecard / Issues (not an RFP)

| Item | Owner | Done looks like |
| --- | --- | --- |
| MSP: gen-AI DNS/proxy/EDR/MFA inventory | Chris + MSP | Written answer; gap = Issue |
| NetSuite host: roles, API/IP allowlist, export audit | Chris + NetSuite firm | Controls documented; least-privilege pass |
| Agent design rule | Chris | Default-deny egress + allowlist + no service-account “see all” for Rock-assists |
| Incident template | Chris | Covers misalignment / rogue outbound ≠ only classic breach |
| Dual spend visibility | Chris + CFO | MSP $ and NetSuite-host $ on Scorecard |

### Explicitly unrealistic in 90 days

- MSP-delivered agentic-attack shield  
- OT-style segmentation theater (minimal shopfloor OT)  
- Org-wide CASB/SSE program before dual-vendor inventory  
- Letting Marketplace / multi-surface agents inherit production NetSuite with standing credentials  

---

## Tie to Rock-assist + certified slices

- Rock-assist agents only see **certified-slice** inputs and **allowlisted** tools.  
- Numbers from NetSuite only via governed roles/exports — not free agent SQL with shared admin.  
- Step-count discipline still applies (~21% clean at 30 steps @ 95%/step): short workflows, human gates on writes.  
- Eval of a Rock-assist bot uses Angle A isolation — never “test in prod with full internet.”

---

## Council / week-one one-liners

- *“We won’t run agent evals that can reach the open internet by accident — deny by default, verify the harness.”*  
- *“MSP won’t magic-shield us; NetSuite host + app design own the crown jewels.”*  
- *“Incidents include weird agent outbound, not only ransomware.”*

---

*Research appendix available from AI News Researcher pack (2026-09-10). IT/OT plane split incorporated.*
