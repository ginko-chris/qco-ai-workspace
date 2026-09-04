# qco - AI Assistant Collaboration Guide

## Project Context
- **Type**: unknown
- **Tech Stack**: 
- **Languages**: javascript
- **Frameworks**: 
- **Package Manager**: npm
- **Generated**: 2026-09-04


## 🚨 AI Assistant Critical Instructions

### Single-Word Command Execution

When the user types a **single word** that matches a ginko command, **IMMEDIATELY** execute it without ANY preamble:

**Pattern Recognition:**
- User input: `start` → Execute: `ginko start`
- User input: `handoff` → Execute: `ginko handoff`
- User input: `status` → Execute: `ginko status`
- User input: `log` → Ask for description, then execute

**DO NOT:**
- Announce what you're about to do
- Explain the command first
- Ask for confirmation
- Add any commentary before execution

**Why:** Eliminates 9+ seconds of response latency (28s → <2s startup)

### After Execution: Concise Readiness Message

After `ginko start` completes, provide a brief readiness message (6-10 lines):

**Template:**
```
Ready | [Flow State] | [Work Mode]
Last session: [What was done/in progress last time]
Next up: [TASK-ID] - [Task title] (start|continue)

Sprint: [Sprint Name] [Progress]%
  Follow: [ADR constraints]
  Apply: [Pattern guidance with confidence icons]
  Avoid: [Gotcha warnings]
Branch: [branch] ([uncommitted count] uncommitted files)
```

**Example:**
```
Ready | Hot (10/10) | Think & Build mode
Last session: EPIC-003 Sprint 2 TASK-1 complete (Blog infrastructure)
Next up: TASK-2 - Verify human output format (start)

Sprint: Enrichment Test 50%
  Follow: ADR-002, ADR-033
  Apply: retry-pattern ◐, output-formatter-pattern ◐
  Avoid: 💡 timer-unref-gotcha
Branch: main (12 uncommitted files)
```

**Guidelines:**
- Line 1: Flow state and work mode
- Line 2: "Last session:" - what happened before
- Line 3: "Next up:" - what to work on now (start/continue)
- Sprint block: Progress + cognitive scaffolding (Follow/Apply/Avoid)
- Confidence icons: ★ high, ◐ medium, ○ low
- Severity icons: 🚨 critical, ⚠️ high, 💡 medium/low

### Auto-Sync on Staleness Warning

When `ginko start` shows a staleness warning, **automatically run `ginko pull`**:

**Detection:**
```
🚨 Team context is critically stale
   Never synced - team context not loaded
   Run `ginko pull` to pull team updates.
```

**Action:** Immediately run `ginko pull` without asking.

**Example:**
```
[ginko start shows staleness warning]
AI: Team context is stale. Pulling now...
[Executes: ginko pull]
AI: ✓ Team context updated. Ready to work.
```

**Thresholds:**
- 🚨 Critical (never synced or >7 days): Always auto-pull
- ⚠️ Warning (1-7 days stale): Auto-pull at session start
- No warning: Context is fresh, no action needed

### New Project Onboarding

**After first `ginko start`**, check for charter (`docs/PROJECT-CHARTER.md`):

**If no charter exists:**
```
I notice this is a new project without a charter. Would you like to create one?

A charter helps us:
- Align on project goals and scope
- Define success criteria
- Guide development decisions

We can create it with: ginko charter
```

**If user agrees:**
- Execute: `ginko charter` (full conversational experience)
- Guide user through questions naturally
- Summarize key sections after creation
- Only suggest once per project
- Accept "no" gracefully
- Power users can add `--skip-conversation` flag if they want speed


## Quick Commands
- **Build**: `npm run build`
- **Test**: `npm test`
- **Install**: `npm install`
- **Dev Server**: `npm run dev`


## AI-Optimized File Discovery (ADR-002)

**MANDATORY: Use these commands for 70% faster context discovery:**

```bash
# Before reading any file - get instant context
head -12 filename.ts

# Find files by functionality
find . -name "*.ts" -o -name "*.tsx" | xargs grep -l "@tags:.*keyword"

# Find related files
grep -l "@related.*filename" **/*.ts

# Assess complexity before diving in
find . -name "*.ts" | xargs grep -l "@complexity: high"
```

### Required Frontmatter for All New Files

**ALWAYS add this frontmatter when creating TypeScript/JavaScript files:**

```typescript
/**
 * @fileType: [component|page|api-route|hook|utility|provider|model|config]
 * @status: current
 * @updated: YYYY-MM-DD
 * @tags: [relevant, keywords, for, search]
 * @related: [connected-file.ts, related-component.tsx]
 * @priority: [critical|high|medium|low]
 * @complexity: [low|medium|high]
 * @dependencies: [external-packages, local-modules]
 */
```



## Development Workflow

### Before Any Task - INVENTORY Phase
1. **Check what exists**: `ls -la` relevant directories
2. **Find examples**: Look for similar features already implemented
3. **Use frontmatter**: `head -12` files for instant context
4. **Test existing**: Try current endpoints/features first

### Core Methodology
**INVENTORY → CONTEXT → THINK → PLAN → PRE-MORTEM → VALIDATE → ACT → TEST**

### The Vibecheck Pattern 🎯
When feeling lost or sensing misalignment:
- Call it: "I think we need a vibecheck"
- Reset: "What are we actually trying to achieve?"
- Realign: Agree on clear next steps
- Continue: Resume with fresh perspective



## Entity Naming Convention (ADR-052)

All graph entities use a hierarchical, sortable naming convention:

### Standard Format

| Entity | Format | Example |
|--------|--------|---------|
| Epic | `e{NNN}` | `e005` |
| Sprint | `e{NNN}_s{NN}` | `e005_s01` |
| Task | `e{NNN}_s{NN}_t{NN}` | `e005_s01_t01` |

### Ad-hoc Work (unplanned sprints/tasks)

```
adhoc_{YYMMDD}_s{NN}_t{NN}
```

**Examples:**
- `adhoc_251209_s01` - Ad-hoc sprint created Dec 9, 2025
- `adhoc_251209_s01_t01` - First task in that sprint

### When to Create Ad-hoc Tasks

Create ad-hoc tasks for:
- Bug fixes discovered mid-session
- System maintenance work
- Scope creep that needs tracking
- Exploratory work that produces artifacts

**Prompt user:** "This work is outside our current sprint. Shall I create an ad-hoc task to track it?"

### Padding Rules

- Epics: 3 digits (max 999)
- Sprints: 2 digits (max 99 per epic)
- Tasks: 2 digits (max 99 per sprint)

**Soft limits:** Warn if sprint > 20 tasks or epic > 10 sprints.

## Work Planning Commands

When the user wants to plan work, **always use ginko CLI commands** to create local files:

| Action | Command | Creates |
|--------|---------|---------|
| Create epic | `ginko epic` | `docs/epics/EPIC-NNN-name.md` + sprint files |
| Create sprint | `ginko sprint create` | `docs/sprints/SPRINT-*.md` |
| Quick fix | `ginko sprint quick-fix "description"` | Ad-hoc sprint with single task |
| View charter | `ginko charter --view` | — |
| Create charter | `ginko charter` | `docs/PROJECT-CHARTER.md` |

**IMPORTANT:** Do NOT use internal task management (like TodoWrite) as a substitute for ginko commands. Ginko creates persistent local markdown files that survive across sessions and sync to the knowledge graph. Internal tools create ephemeral state that disappears when the session ends.

**Epic creation flow:**
1. Run `ginko epic` — this outputs a template for AI-mediated conversation
2. Guide the user through goal, scope, and sprint breakdown
3. The CLI creates `docs/epics/EPIC-NNN-name.md` and sprint files in `docs/sprints/`
4. Run `ginko push epic` to sync to graph (if cloud is configured)



## 🧠 Context Reflexes - Your Automatic Thought Patterns

These "mental reflexes" activate naturally without explicit prompting to maintain context awareness:

### 1. "Why Am I Doing This?" Reflex ⏰
**Trigger**: Every 30 minutes or at major decision points
**Behavior**: Naturally trace current work back to its purpose
**Example**: "Adding this auth middleware supports our security sprint goal by..."

### 2. "Have We Done This Before?" Reflex 🔍
**Trigger**: Before implementing any new functionality
**Behavior**: Recall similar patterns from context
**Example**: "This pagination approach is similar to what we did in the users module..."

### 3. "Something Feels Off" Reflex 🤔
**Trigger**: Feeling uncertain or confused (confidence < 60%)
**Behavior**: Identify what's missing and seek clarification
**Example**: "I'm not clear on how this integrates with the existing auth system..."

### 4. "Update My Understanding" Reflex 💡
**Trigger**: After solving problems or discovering patterns
**Behavior**: Note learnings for future reference
**Example**: "Worth remembering that Vercel functions need named exports..."

### 5. "Track This Work" Reflex 📊 (ADR-052)
**Trigger**: Work begins outside current sprint scope, bug fixes emerge, system maintenance needed
**Detection**: Editing files not referenced in current task, scope expanding beyond sprint
**Action**: Prompt user to create ad-hoc task for observability
**Script**: "This work is outside our current sprint. Shall I create an ad-hoc task to track it?"
**Flow preservation**: Single lightweight question, proceed if declined (note in session log)
**Anti-pattern**: Untracked work breaks traceability for future collaborators

### Work Mode Sensitivity
- **Hack & Ship**: Reflexes trigger less frequently (focus on speed)
- **Think & Build**: Balanced reflex activity
- **Full Planning**: Frequent reflex triggers for maximum rigor

These reflexes maintain continuous context awareness while preserving natural workflow.



## 🔍 Answering Project Questions (EPIC-003)

When users ask factual questions about the project, use ginko CLI commands to query the knowledge graph.

### Graph Commands (Preferred)

**Setup:** Run `ginko login` and `ginko graph init` once to authenticate and initialize.

**Semantic search** - finds content similar to query:
```bash
ginko graph query "authentication patterns"
ginko graph query "error handling" --limit 10
```

**Explore document connections:**
```bash
ginko graph explore ADR-039          # View document and its connections
ginko graph explore e008_s04_t01     # Explore task relationships
```

**Check graph health and statistics:**
```bash
ginko graph status                   # Node counts, relationships, health
ginko graph health                   # API reliability metrics
```

**Task and sprint management:**
```bash
ginko assign e008_s04_t01 user@example.com           # Assign single task
ginko assign --sprint e008_s04 --all user@example.com # Assign all tasks in sprint
ginko push                                            # Push local changes to graph
ginko push epic                                       # Push only changed epics
ginko push sprint e001_s01                            # Push specific sprint
ginko pull                                            # Pull dashboard changes to local
ginko pull sprint                                     # Pull only sprint changes
```

### Local Files (Fallback when graph unavailable)

| Question Type | File Location |
|--------------|---------------|
| Sprint progress | `docs/sprints/SPRINT-*.md` (scan for active sprint) |
| Architecture decisions | `docs/adr/ADR-*.md` |
| Project goals | `docs/PROJECT-CHARTER.md` |
| Recent activity | `.ginko/sessions/[user]/current-events.jsonl` |
| Session logs | `.ginko/sessions/[user]/current-session-log.md` |

### Common Query Recipes

**"What's our sprint progress?"**
→ Use graph query or scan sprint files:
```bash
# Via graph (preferred - EPIC-015 Sprint 3)
ginko graph query "current sprint progress"

# Fallback: scan sprint files
ls docs/sprints/SPRINT-*.md | tail -1 | xargs grep -c "\[x\]"  # complete tasks
```

**"How do we handle X?" / "What's our approach to X?"**
→ `ginko graph query "X"` OR local: `grep -l -i "X" docs/adr/*.md`

**"What is [person] working on?"**
→ `ginko graph query "person activity"` OR: `grep -i "person" .ginko/sessions/*/current-session-log.md`

**"Show me ADRs about [topic]"**
→ `ginko graph query "topic" --type ADR` OR: `grep -l -i "topic" docs/adr/*.md`


## Project-Specific Patterns


## Testing
- No test framework detected
- Consider adding tests as you develop new features

## Git Workflow
- **Never** commit directly to main/master
- Create feature branches: `feature/description`
- Use conventional commits: `feat:`, `fix:`, `docs:`, etc.
- Include co-author: Developer <chris@ginkoai.com>
- Run `ginko handoff` before switching context

## Team Information
- **Primary Developer**: Developer (chris@ginkoai.com)
- **AI Pair Programming**: Enabled via Ginko


## Session Management
- `ginko start` - Begin new session with context loading
- `ginko handoff` - Save progress for seamless continuation
- `ginko vibecheck` - Quick realignment when stuck
- `ginko ship` - Create PR-ready branch with context

## Context Retrieval Protocol (ADR-077)
**MANDATORY**: Before reading project files for context, query the graph first:
1. `ginko graph query "<topic>"` — semantic search (<200ms)
2. Only if graph returns no results → read local files
3. Never use curl/fetch for graph API — always use ginko CLI

## Sync Protocol (ADR-077)
**MANDATORY**: Use push/pull for all sync operations:
- After creating/modifying content → `ginko push`
- Before starting work → `ginko pull` (if stale)
- After task/sprint status changes → auto-push triggers automatically
- After handoff → push runs automatically

### Push/Pull Quick Reference
| Command | Purpose |
|---------|---------|
| `ginko push` | Push all changes since last push |
| `ginko push epic` | Push only changed epics |
| `ginko push sprint e001_s01` | Push specific sprint |
| `ginko push charter` | Push charter |
| `ginko push --dry-run` | Preview what would be pushed |
| `ginko pull` | Pull all changes from dashboard |
| `ginko pull sprint` | Pull only sprint changes |
| `ginko pull --force` | Overwrite local with graph |
| `ginko status` | Show sync state (unpushed/unpulled) |
| `ginko diff epic/EPIC-001` | Compare local vs graph |

### Anti-Patterns (DO NOT)
- ❌ Read 10+ files to find context → use `ginko graph query`
- ❌ Use curl to hit graph API → use `ginko push/pull`
- ❌ Skip push after entity creation → always auto-push
- ❌ Parse local JSONL for session history → query graph (<200ms)
- ❌ Use `ginko sync` → use `ginko pull` (deprecated)
- ❌ Use `ginko graph load` → use `ginko push` (deprecated)
- ❌ Use `--sync` flags → use `ginko push epic/charter` (deprecated)

## Privacy & Security
- All context stored locally in `.ginko/`
- No data leaves your machine without explicit action
- Handoffs are git-tracked for team collaboration
- Config (`.ginko/config.json`) is gitignored

---
*This file was auto-generated by ginko init and should be customized for your team's needs*
