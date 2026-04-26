# Compliance Framework — Governance Runtime v4

> Single source of truth for all rules, enforcement, evidence, and violation handling.
> Every command reads this. /healthcheck audits against it. No exceptions.

---

## §1 — Rule Classification

Every rule in the governance runtime falls into one of three tiers:

| Tier | Name | Can Override? | Enforcement | Violation Response |
|------|------|--------------|-------------|-------------------|
| **T1** | **Iron** | Never — not even with OVERRIDE.md | Automated (built into commands) | Hard block — work stops until resolved |
| **T2** | **Steel** | Only via OVERRIDE.md with expiry date | Automated with escape hatch | Warning + logged + must acknowledge |
| **T3** | **Copper** | Freely adjustable per project | Advisory — recommended but not blocked | Logged in /retro metrics |

---

## §2 — Iron Rules (T1) — Non-Negotiable

These cannot be overridden under any circumstances. They are the constitutional bedrock.

### R1: Intent Before Code
Every piece of work follows: INTENT → PLAN → TEST → IMPLEMENT → PROOF → COMMIT → DONE.
- **Enforcement:** `/dev` refuses to proceed without stated intent
- **Evidence:** Intent block in every commit's associated task entry
- **Violation:** Task marked invalid, must restart from intent

### R2: Test-First in Planned Mode
Failing test must exist before implementation code in any `/dev` task.
- **Enforcement:** `/dev` Tier 1 proof gate checks test file timestamps
- **Evidence:** Git log shows test commit before implementation commit
- **Violation:** Hard block — cannot proceed to implement step
- **Scope:** Planned mode only. Craft mode tracks test debt instead (see R14).

### R3: Boundary Isolation During Parallel
During parallel sprint execution, each agent may only write to files in its declared scope.
- **Enforcement:** `/dev` runs scope check before every file modification
- **Evidence:** Workspace status file logs every file touched
- **Violation:** Hard block — file write rejected, task marked BLOCKED

### R4: Contract Immutability During Parallel
`src/contracts/` is READ-ONLY once parallel execution begins (after scaffold sprint).
- **Enforcement:** `/dev` rejects writes to `src/contracts/` for non-scaffold sprints
- **Evidence:** Git diff shows no contract changes outside Wave 1
- **Violation:** Hard block — must coordinate contract changes through /sync

### R5: Main Branch Protection
Only `/ship` writes to main. Direct commits to main are forbidden.
- **Enforcement:** Git branch protection + `/ship` gating
- **Evidence:** All main commits come via merge from sprint branches
- **Violation:** Hard block — `/ship` refuses if uncommitted changes exist on main

### R6: Memory Persistence Protocol
MEMORY.md is READ-ONLY during parallel sprints. Writable only by `/sync` (planned) and `/evolve` (craft).
- **Enforcement:** `/dev` prohibits MEMORY.md writes during parallel execution
- **Evidence:** Git blame shows MEMORY.md only modified by sync/evolve commits
- **Violation:** Hard block during parallel — changes buffer in workspace status

### R7: Root Cause Before Fix
`/debug` follows the Iron Law: no fix without root cause investigation.
- **Enforcement:** `/debug` requires hypothesis statement before any code change
- **Evidence:** Commit message must include root cause explanation
- **Violation:** 3-fix rule escalation — after 3 failed hypotheses, architecture review required

---

## §3 — Steel Rules (T2) — Override With Justification

These can be relaxed via OVERRIDE.md but require an expiry date and rationale.

### R8: Four-Tier Test Gates
- Tier 1 (task): unit + types + lint after every task
- Tier 2 (sprint): + integration + coverage after sprint
- Tier 3 (wave): + cross-sprint after merge
- Tier 4 (ship): + full regression + /qa before main
- **Default enforcement:** Each tier gates the next phase
- **Override format:**
  ```markdown
  OVERRIDE R8: Relax Tier 2 to skip E2E for prototype sprint v3-prototype
  RATIONALE: Prototype will be discarded, not shipped to production
  EXPIRES: 2026-04-15
  APPROVED BY: Siva
  ```
- **Evidence:** Test results logged to `.context/last-test-result`

### R9: Architect Review Required Before /ship
`/architect` must show CLEAR in Review Readiness Dashboard before `/ship` proceeds.
- **Default enforcement:** `/ship` warns if architect review missing
- **Override:** Can skip for hotfix branches
- **Evidence:** `.context/review-readiness.json`

### R10: Design System Enforcement
If DESIGN.md exists, `/dev` must follow it for UI work. `/qa` checks against it.
- **Default enforcement:** `/qa` flags design system violations
- **Override:** Can disable for backend-only sprints
- **Evidence:** `/qa` report includes design compliance section

### R11: Documentation Sync on Release
`/doc-sync` must run before `/ship` tags a release.
- **Default enforcement:** `/ship` runs `/doc-sync` as part of its pipeline
- **Override:** Can skip for hotfix releases
- **Evidence:** Git log shows docs(sync) commit before tag

### R12: Conventional Commits
All commits follow: `{type}({scope}): {description}`
- Types: feat, fix, test, refactor, docs, style, chore
- **Default enforcement:** `/dev` commit step follows format
- **Override:** Can relax for experimental branches
- **Evidence:** Git log parseable by conventional-changelog

### R13: Single Responsibility per Commit
Each commit addresses one concern. Atomic, bisectable.
- **Default enforcement:** `/dev` commits per-task, `/qa` commits per-bug
- **Override:** Can batch for trivial changes
- **Evidence:** `git log --oneline` shows clear single-purpose entries

### R14: Craft Mode Test Debt Tracking
Every craft change without test coverage is logged. `/promote` enforces payback.
- **Default enforcement:** `/evolve` tracks test debt, `/promote` creates test tasks
- **Override:** Can defer for pure CSS/copy changes
- **Evidence:** Craft session files list test debt items

---

## §4 — Copper Rules (T3) — Recommended Practices

These are tracked in `/retro` metrics but never block work.

### R15: Office Hours Before New Features
Run `/office-hours` before building new product features.
- **Tracking:** `/retro` reports ratio of features with vs without office hours
- **Recommendation:** Always for HIVE (product-market fit), optional for Ops OS (internal tool)

### R16: Reframe Before PRD
Run `/reframe` before `/prd` for user-facing features.
- **Tracking:** `/retro` reports reframe coverage
- **Recommendation:** Especially for features touching end-user workflows

### R17: Weekly Retro
Run `/retro` at least once per week.
- **Tracking:** `.context/retros/` directory shows frequency
- **Recommendation:** Every Monday, covers previous 7 days

### R18: Craft Session Hygiene
Use `/craft` and `/evolve` to bookend ad hoc work sessions.
- **Tracking:** `/retro` shows craft-to-evolve ratio (sessions opened vs closed)
- **Recommendation:** Don't let more than 3 sessions accumulate without `/evolve`

### R19: Health Score Threshold
`/qa` health score should be ≥ 80 before shipping.
- **Tracking:** `.context/qa-reports/` shows score trends
- **Recommendation:** Hard requirement for production, advisory for staging

### R20: Completeness Over Shortcuts
When the complete version is a "lake" (achievable), build it. Don't take shortcuts.
- **Tracking:** `/review` flags "LAKE" completeness gaps
- **Recommendation:** Always show both effort scales (human vs CC time)

---

## §5 — Evidence Chain

Every rule produces evidence. Here's where to find it:

| Evidence | Location | Produced By | Consumed By |
|----------|----------|-------------|-------------|
| Task intent blocks | `sprints/*/prd.md` task entries | `/dev` | `/review`, `/healthcheck` |
| Test results per tier | `.context/last-test-result` | `/test` | `/ship`, `/retro` |
| File scope logs | `workspaces/*.md` | `/dev`, `/conductor` | `/sync`, `/healthcheck` |
| Review readiness | `.context/review-readiness.json` | `/architect`, `/review`, `/design` | `/ship` |
| QA health scores | `.context/qa-reports/*.json` | `/qa` | `/ship`, `/retro` |
| Design audit reports | `.context/design-reports/*.md` | `/design` | `/qa`, `/review` |
| Craft session files | `sessions/craft-*.md` | `/craft`, `/log` | `/evolve`, `/promote`, `/retro` |
| Test debt items | `sessions/craft-*.md` | `/evolve` | `/promote` |
| Retro snapshots | `.context/retros/*.json` | `/retro` | Next `/retro` (trend comparison) |
| Debug hypotheses | Commit messages | `/debug` | `/review` (pattern learning) |
| Review false positives | `.context/review-history.md` | `/review` | Future `/review` runs |
| Decisions log | `MEMORY.md` | `/sync`, `/evolve` | All commands |
| Sprint dependency graph | `MANIFEST.md` | `/prd`, `/evolve`, `/promote` | `/conductor`, `/sync` |
| Override justifications | `OVERRIDE.md` | Manual | `/healthcheck` |
| TODOS provenance | `TODOS.md` | `/reframe`, `/review`, `/retro` | `/promote`, `/prd` |

---

## §6 — Override Protocol

To override a Steel (T2) rule:

### Step 1: Add to OVERRIDE.md
```markdown
## Active Overrides

### OVERRIDE R{N}: {short description}
- **Rule:** {full rule name}
- **Relaxation:** {what's being changed}
- **Rationale:** {why this is justified}
- **Scope:** {which sprints/branches this applies to}
- **Expires:** {date — REQUIRED}
- **Approved by:** {name}
- **Date created:** {date}
```

### Step 2: Acknowledge in Commit
```bash
git commit -m "chore(override): relax R{N} for {reason} — expires {date}"
```

### Step 3: /healthcheck Tracks Expiry
`/healthcheck` flags overrides that have expired but haven't been removed.

### What Cannot Be Overridden
Iron Rules (T1) — R1 through R7. Period. If you need to change these, you're changing the constitution, not filing an override. That requires editing CLAUDE.md §6 and re-running `/healthcheck`.

---

## §7 — Audit Protocol

### Continuous (every command)
Each command enforces its relevant rules and logs evidence automatically.

### Periodic (/healthcheck — weekly recommended)
`/healthcheck` performs a full compliance audit:

```
COMPLIANCE AUDIT — {date}

IRON RULES (T1):
  R1 Intent-First:     ✓ All tasks have intent blocks
  R2 Test-First:       ✓ Test commits precede implementation
  R3 Boundary:         ✓ No scope violations detected
  R4 Contracts:        ✓ src/contracts/ unchanged in parallel
  R5 Main Protection:  ✓ All main commits via /ship
  R6 Memory Protocol:  ✓ MEMORY.md only modified by sync/evolve
  R7 Root Cause:       ✓ All /debug commits have root cause

STEEL RULES (T2):
  R8 Test Gates:       ✓ All tiers passing
  R9 Architect Review: ⚠️ Missing for sprint v3-ui — run /architect
  R10 Design System:   ✓ DESIGN.md enforced
  R11 Doc Sync:        ✓ Last sync: {date}
  R12 Conventional:    ✓ All commits parseable
  R13 Atomic Commits:  ✓ No multi-concern commits
  R14 Test Debt:       ⚠️ 3 items in craft sessions

OVERRIDES:
  R8 relaxed for v3-prototype — EXPIRES: 2026-04-15 (26 days remaining)

COPPER METRICS:
  Office hours coverage:    2/5 features (40%)
  Reframe coverage:         4/5 features (80%)
  Weekly retro streak:      3 weeks
  Craft hygiene:            8 opened, 6 evolved (75%)
  Average QA health score:  87
  Completeness gaps:        2 open "LAKE" items
```

### On-Demand (/retro — analyzes compliance trends)
`/retro` includes a compliance section showing trends:
- Rule violations this period vs last
- Override usage trends
- Copper metric trajectories
- Test debt accumulation rate

---

## §8 — Violation Handling

### Iron Rule Violation
```
1. HARD BLOCK — current operation stops
2. Agent displays: "⛔ IRON RULE VIOLATION: R{N} — {description}"
3. Agent explains what needs to happen to resolve
4. No workaround. No override. Fix it.
5. If the rule itself is wrong → edit CLAUDE.md §6 (constitutional change)
```

### Steel Rule Violation
```
1. WARNING — operation can continue after acknowledgment
2. Agent displays: "⚠️ STEEL RULE WARNING: R{N} — {description}"
3. Options:
   A) Fix now
   B) Acknowledge and continue (logged)
   C) Create override in OVERRIDE.md (with expiry)
4. Violation logged in .context/compliance-log.md
5. /retro surfaces violation trends
```

### Copper Rule Miss
```
1. No block, no warning during work
2. /retro includes metrics showing coverage
3. /healthcheck shows copper metrics in audit
4. Human decides whether to improve
```

---

## §9 — Compliance File Map

```
project-root/
├── CLAUDE.md                        ← Constitution (Iron Rules defined in §6)
├── COMPLIANCE.md                    ← THIS FILE — full framework
├── OVERRIDE.md                      ← Active Steel Rule overrides
├── .context/
│   ├── compliance-log.md            ← Steel Rule violation log
│   ├── review-readiness.json        ← Review gate status
│   ├── last-test-result             ← Test tier results
│   ├── review-history.md            ← False positive learning
│   ├── qa-reports/                  ← QA health scores
│   ├── design-reports/              ← Design audit results
│   └── retros/                      ← Retro snapshots with compliance trends
└── OVERRIDE.md                      ← Steel Rule overrides with expiry
```
