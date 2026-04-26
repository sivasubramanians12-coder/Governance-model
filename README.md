# Governance Runtime v4

**Drop-in governance for Claude Code. Parallel agents, cognitive modes, craft mode, and dynamic injection.**

v3 orchestration + gstack cognitive modes + craft mode protocol + `!`command`` dynamic context.

---

## Quick Start

```bash
# 1. Copy into your project root
cp -r governance-v4/* /path/to/your-project/

# 2. Open Claude Code
claude

# 3. Bootstrap
> /init

# 4. Plan your work
> /reframe "describe your feature"    # Challenge scope (optional)
> /architect                           # Force diagrams (recommended)
> /prd                                 # Plan sprints

# 5. Build
> /conductor    # Assign Wave 1
> /dev          # Build tasks (TDD)
> /sync         # Merge wave
> /ship         # Release

# 6. After shipping — iterate freely
> /craft "fixing mobile issues"
> ... work ...
> /evolve       # Updates governance files automatically
```

## 24 Commands

### Planned Mode (Sprint Lifecycle)
| `/init` | `/prd` | `/conductor` | `/dev` | `/test` |
|---------|--------|-------------|--------|---------|
| `/walkthrough` | `/sync` | `/healthcheck` | `/ship` | |

### Cognitive Modes (Any Time)
| `/office-hours` Partner | `/reframe` CEO | `/architect` Eng | `/review` Paranoid |
|------------------------|----------------|-----------------|-------------------|
| `/debug` Debugger | `/design` Designer | `/qa` QA Lead | `/retro` Analytics |
| `/doc-sync` Writer | | | |

### Craft Mode (Post-Sprint)
| `/craft` Open | `/log` Capture | `/evolve` Update | `/promote` Convert |
|--------------|----------------|-------------------|-------------------|

## Dynamic Injection

Commands use `!`command`` to inject live project state at load time. The model sees results, not commands. See `docs/DYNAMIC-INJECTION-CHEATSHEET.md` for the full reference.

## File Map

```
project-root/
├── CLAUDE.md                    ← Constitution
├── MEMORY.md                    ← Persistent memory
├── MANIFEST.md                  ← Sprint DAG
├── DESIGN.md                    ← Design system
├── OVERRIDE.md                  ← Per-project overrides
├── TODOS.md                     ← Deferred work
├── .claude/commands/            ← All 22 commands
├── src/contracts/               ← Shared interfaces
├── sprints/                     ← Sprint folders
├── sessions/                    ← Craft session files
├── workspaces/                  ← Ephemeral workspace status
├── .context/                    ← Reports & snapshots
│   ├── retros/
│   ├── qa-reports/
│   └── design-reports/
└── docs/
    ├── DYNAMIC-INJECTION-CHEATSHEET.md
    └── designs/                 ← Vision docs from /reframe
```

## Non-Negotiable Rules

1. Proof gates (all 4 tiers)
2. TDD (test-first) in planned mode
3. Boundary isolation during parallel
4. Contract immutability during parallel
5. Git branch protection on main
6. Memory persistence protocol
7. Intent-first flow
