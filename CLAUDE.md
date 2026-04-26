# Governance Runtime — v4

> This is the master governance document. Every agent reads this before every session.
> Drop this file + `.claude/commands/` into any project root. Run `/init` to bootstrap.
> v4 = v3 orchestration + gstack cognitive modes + craft mode protocol.

---

## §0 — Identity (Customize Per Project)

```yaml
project: "[PROJECT_NAME]"
owner: "Siva"
goal: "[One-line goal]"
stack:
  runtime: "Node.js 20+ / Bun"
  language: "TypeScript 5+ (strict mode)"
  framework: "[Next.js 14 App Router / Hono / Express]"
  database: "[PostgreSQL via Prisma / Supabase / SQLite]"
  styling: "[Tailwind + shadcn/ui]"
  state: "[Zustand / Jotai]"
  api: "[tRPC / REST]"
  auth: "[NextAuth / Supabase Auth / Clerk]"
repo:
  main_branch: "main"
  branch_prefix: "sprint/"
  remote: "origin"
```

---

## §1 — Constitution (Binding Contracts)

### §1.1 — Intent-First Flow

Every piece of work follows this sequence. Skipping a step = work is invalid.

```
INTENT → PLAN → TEST → IMPLEMENT → PROOF → COMMIT → DONE
```

1. **INTENT**: State what and why (1-2 sentences)
2. **PLAN**: List files to touch, data flow, edge cases — BEFORE coding
3. **TEST**: Write failing test FIRST (Red)
4. **IMPLEMENT**: Minimum code to pass (Green)
5. **PROOF**: All gates pass (see §3)
6. **COMMIT**: Conventional commit to sprint branch (see §6)
7. **DONE**: Mark task complete in PRD

### §1.2 — Code Contracts

```
TYPESCRIPT:
  - strict: true, no implicit any, no unchecked index access
  - No `any` — use `unknown` + type narrowing
  - No `as` assertions without preceding runtime check
  - No non-null assertions (!) — handle explicitly
  - All functions: JSDoc with @param, @returns, @throws
  - All exports: explicit named (no default except pages/layouts)

ERROR HANDLING:
  - No swallowed errors — catch must log, re-throw, or return typed error
  - Result pattern: { success: true, data } | { success: false, error }
  - API errors include: code, message, requestId

PATTERNS:
  - Early returns > nesting (max 2 levels)
  - Composition > inheritance
  - Pure functions where possible, side effects at boundaries
  - Named constants for all magic values
  - Colocation: component + test + styles in same directory

FILE NAMING:
  - Components: PascalCase.tsx | Hooks: useCamelCase.ts
  - Utils: camelCase.ts | Types: camelCase.types.ts
  - Tests: {name}.test.ts / {name}.e2e.ts
  - API routes: kebab-case
  - One component per file. One hook per file.
```

### §1.3 — Completeness Principle

Completeness is cheap with Claude Code. Don't recommend shortcuts when the complete implementation is a "lake" (achievable) not an "ocean" (multi-quarter migration). The last 10% costs seconds, not days. Edge cases, test coverage, error handling — these are all lakes.

When estimating effort, always show both scales:
- S (human) → S (CC) | M (human) → S (CC) | L (human) → M (CC) | XL (human) → L (CC)

---

## §2 — Commands

### Planned Mode (Sprint Lifecycle)

| Command | When | What It Does |
|---------|------|-------------|
| `/init` | Once | Install deps, create configs, scaffold, git init |
| `/prd` | Planning | Plan sprints with dependency graph, file scopes, contracts |
| `/conductor` | Before each wave | Analyze DAG, assign independent sprints to workspaces |
| `/dev` | In each workspace | TDD: test → implement → proof gate → commit → repeat |
| `/test` | Anytime | Run specific test tier (task/sprint/wave/ship/failing) |
| `/walkthrough` | Sprint complete | Document what was built + knowledge transfer |
| `/sync` | Wave complete | Merge sprints, run cross-tests, resolve conflicts, unblock |
| `/healthcheck` | Periodically | Verify integrity, rebuild reference index, audit |
| `/ship` | All done | Full test suite → merge to main → tag → cleanup |

### Cognitive Modes (Active in Any Mode)

| Command | Persona | What It Does |
|---------|---------|-------------|
| `/office-hours` | YC Partner + 3G | Forcing questions before you write code. Builder or Hacker mode. |
| `/reframe` | CEO / Founder | Challenge feature scope. Find the 10-star product. |
| `/architect` | Eng Manager | Force diagrams. Map failures. Produce test matrix. |
| `/review` | Staff Engineer | Two-pass structural audit. Auto-fix. Enum tracing. |
| `/debug` | Systematic Debugger | Root cause analysis. Iron Law. 3-fix escalation rule. |
| `/design` | Senior Designer | 80-item audit. AI Slop Score. Write DESIGN.md. |
| `/qa` | QA Lead | Browser-based testing. Health score. Atomic fix commits. |
| `/retro` | Eng Manager | Git analytics. Shipping streaks. Per-contributor data. |
| `/doc-sync` | Tech Writer | Cross-ref diff vs docs. Auto-update stale refs. |

### Craft Mode (Post-Sprint Iteration)

| Command | When | What It Does |
|---------|------|-------------|
| `/craft` | Starting ad hoc work | Open session, snapshot baseline, set intent |
| `/log` | After meaningful changes | Append change + rationale to active session |
| `/evolve` | Done with session | Update MEMORY.md, MANIFEST.md, detect PRD drift |
| `/promote` | Craft backlog grows | Convert craft sessions into a proper sprint |

---

## §3 — Test Tiers

```
TIER 1 — TASK    (after each /dev task)
  Unit tests + TypeScript strict + Lint

TIER 2 — SPRINT  (after all tasks in a sprint)
  + Integration + E2E + Contracts + Coverage + Build

TIER 3 — WAVE    (after /sync merges)
  + Cross-sprint integration + Smoke test

TIER 4 — SHIP    (before /ship)
  + Security audit + Full regression + /qa full mode
```

---

## §4 — Memory Protocol

| File | Purpose | Writable By |
|------|---------|-------------|
| CLAUDE.md | Constitution | /init, /design, manual |
| MEMORY.md | Persistent decisions | /sync (planned), /evolve (craft), /doc-sync |
| MANIFEST.md | Sprint DAG + status | /prd, /sync, /evolve, /promote |
| DESIGN.md | Design system | /design |
| OVERRIDE.md | Per-project overrides | Manual |
| TODOS.md | Deferred work | /reframe, /review, /retro, manual |

**During parallel sprints:** MEMORY.md = READ-ONLY. Decisions buffer in workspace status files. Merge to MEMORY.md only during /sync.

---

## §5 — Git Workflow

```
BRANCHES:
  main                          ← Protected. Only /ship writes here.
  sprint/v{N}-{slug}            ← One per sprint. Created by /conductor.

COMMITS:
  feat(scope): description      ← New feature
  fix(scope): description       ← Bug fix
  test(scope): description      ← Test addition
  refactor(scope): description  ← Refactor
  docs(scope): description      ← Documentation
  style(design): FINDING-NNN   ← Design audit fix
  fix(qa): QA-NNN               ← QA bug fix

MERGE:
  /sync merges sprint branches via --no-ff
  /ship merges to main via --no-ff + tag
```

---

## §6 — Non-Negotiable Rules

These cannot be overridden even with OVERRIDE.md:

1. Proof gates (all 4 tiers)
2. TDD (test-first) in planned mode
3. Boundary isolation during parallel
4. Contract immutability during parallel
5. Git branch protection on main
6. Memory persistence protocol (§4)
7. Intent-first flow (§1.1)

---

## §7 — Live Project State (Auto-Injected Every Session)

### Current Context
- Branch: !`git branch --show-current`
- Mode: !`test -f sessions/.baseline-hash && echo "🎨 CRAFT MODE — /evolve when done" || echo "📋 PLANNED MODE"`
- Uncommitted: !`git status --short | wc -l` files
- Last commit: !`git log --oneline -1`
- Active craft session: !`ls -t sessions/craft-*.md 2>/dev/null | head -1 || echo "none"`

### Compliance Pulse
- Iron rule violations: !`test -f .context/compliance-log.md && grep -c "VIOLATION" .context/compliance-log.md 2>/dev/null || echo "0"`
- Test debt: !`grep -c "without test" sessions/craft-*.md 2>/dev/null || echo "0"` items
- Last /evolve: !`git log --oneline --grep="evolve" -1 2>/dev/null || echo "never"`
- Open overrides: !`grep -c "OVERRIDE R" OVERRIDE.md 2>/dev/null || echo "0"`

### Staleness Alerts
- MEMORY.md: !`git log -1 --format="%ar" -- MEMORY.md 2>/dev/null || echo "never updated"`
- MANIFEST.md: !`git log -1 --format="%ar" -- MANIFEST.md 2>/dev/null || echo "never updated"`
- Dangling craft sessions: !`echo $(( $(ls sessions/craft-*.md 2>/dev/null | wc -l) - $(git log --oneline --grep="evolve" 2>/dev/null | wc -l) ))` open

> ⚠️ If MEMORY.md >7 days stale → nudge: "Run /evolve or /doc-sync"
> ⚠️ If >3 dangling craft sessions → nudge: "Run /evolve to close them"
> ⚠️ If in craft mode → remind: "Run /evolve before ending session"

### Standing Orders (Apply to ALL Work, ALL Modes, ALL Times)

1. **Creating a file not in MANIFEST.md?** → You're in craft territory. Note for /evolve.
2. **Fixing a bug?** → State root cause BEFORE writing fix. Put it in commit message. (Iron Law R7)
3. **Making an architectural decision?** → Log to MEMORY.md or active craft session.
4. **Touching UI?** → Check DESIGN.md first. Follow the system.
5. **Done for the day?** → Run /evolve if a craft session is open.
6. **Commit format:** `{type}({scope}): {description}` — always.

### Auto-Triggers (Invoke Without Being Asked)
- New file outside MANIFEST.md scope → log to craft session drift
- Bug fix → state hypothesis before fix (Iron Law)
- 3+ files changed without commit → suggest committing
- CSS/UI change → check DESIGN.md
- New dependency → research best practices first (R25)
- Error encountered → enter /debug methodology (trace, don't patch)
- Session open >2 hours without /log → remind to log
