# GSE-One — Generic Software Engineering One

Mono-plugin architecture — cross-platform parity. See [CHANGELOG](../CHANGELOG.md) for version history.

> AI engineering companion for structured SDLC management. 22 commands, adaptive risk analysis, unified backlog, knowledge transfer, worktree isolation.

## What is GSE-One?

GSE-One is an AI engineering companion that guides users through the full software development lifecycle. It is implemented as a single plugin deployable on both Claude Code and Cursor, providing 22 commands covering planning, requirements, design, production, quality, delivery, and capitalization.

GSE-One is designed for users of **any expertise level** — from beginners building their first project to experienced engineers managing complex applications. The agent adapts its language, decisions, and level of autonomy to your profile.

## Quick Start

### 1. Install the plugin (see Installation below)

### 2. Type `/gse:go`

The agent detects your project state and proposes the next step:
- **New project?** The agent runs an onboarding interview (`/gse:hug`) to understand your expertise and goals
- **Existing project without GSE-One?** The agent proposes to adopt it non-destructively (scan, set baseline, initialize)
- **Already set up?** The agent picks up where you left off — whether that's planning, producing, reviewing, or delivering

### 3. Follow the agent's guidance

GSE-One orchestrates the full lifecycle: `COLLECT > ASSESS > PLAN > PRODUCE > REVIEW > FIX > DELIVER > COMPOUND`. You don't need to memorize the commands — the agent proposes the next activity at each step.

## Installation

### Step 1 — Clone the Repository

Clone the repo once, anywhere on your machine:

```bash
git clone https://github.com/nicolasguelfi/gensem.git ~/gensem
```

### Step 2 — Choose an Installation Scope

GSE-One provides 22 `/gse:*` commands. If you use multiple plugins, you may not want all of them loaded at all times. Choose the scope that fits your workflow:

| Scope | When GSE-One is active | Best for |
|-------|----------------------|----------|
| **Project** | Only when you work inside a specific project | Most users — keeps your `/` menu clean |
| **Global** | In every session, from any directory | Users who want GSE-One always available |
| **One-time** | Current terminal session only | Quick testing |

#### Project scope (recommended)

The plugin is only active when Claude Code or Cursor is launched inside the project directory. Other projects remain unaffected.

**Claude Code** — run this from your project root:
```bash
claude plugin install ~/gensem/gse-one/plugin --scope project
```
This creates a `.claude/settings.json` in your project (committable, shared with your team).

For a personal project-level install (not committed to git), use `--scope local` instead — it writes to `.claude/settings.local.json` (gitignored).

**Cursor:**
```bash
cp -r ~/gensem/gse-one/plugin/ ./gse-one-plugin/
# In Cursor: /add-plugin > Local > select gse-one-plugin/
```

#### Global scope

The plugin is available in every Claude Code session, regardless of the working directory.

```bash
claude plugin install ~/gensem/gse-one/plugin --scope user
```

This adds the plugin to `~/.claude/settings.json`.

#### One-time session (testing only)

Loads the plugin for the current session only. Nothing is persisted.

```bash
claude --plugin-dir ~/gensem/gse-one/plugin/
```

### Step 3 — Verify

Type `/gse:go`. The agent should respond and detect your project state.

### Marketplace (when available)

These methods are not yet operational. They will become available once the official marketplaces accept submissions.

**Claude Code:**
```bash
claude plugin install gse-one
```

**Cursor:**
```
# Search "gse-one" in /add-plugin
```

## Commands (22)

| Category | Commands | What they do |
|----------|----------|-------------|
| **Orchestration** | `/gse:go` | Detect project state, propose next activity. Entry point. |
| | `/gse:status` | Show sprint state, artefact inventory, health, git state |
| | `/gse:health` | Display the 8-dimension health dashboard (0-10 scale) |
| **Session** | `/gse:pause` | Auto-commit work, save session checkpoint |
| | `/gse:resume` | Reload checkpoint, verify worktrees, propose next action |
| **Onboarding** | `/gse:hug` | Establish your profile (expertise, language, goals) |
| **Learning** | `/gse:learn` | Start a learning session on any SE topic |
| **Backlog** | `/gse:backlog` | View and manage the unified work item list |
| **Discovery** | `/gse:collect` | Inventory project artefacts + scan external sources |
| | `/gse:assess` | Gap analysis: what's covered, what's missing |
| **Planning** | `/gse:plan` | Select backlog items for sprint, assign branches |
| **Engineering** | `/gse:reqs` | Define functional and non-functional requirements |
| | `/gse:design` | Architecture decisions, component decomposition |
| | `/gse:preview` | Simulate planned artefacts before building |
| | `/gse:tests` | Test strategy, environment setup, execution, evidence |
| | `/gse:produce` | Execute production in isolated worktree |
| | `/gse:deliver` | Merge, tag release, cleanup branches |
| **Quality** | `/gse:review` | Review branch diff + devil's advocate (AI integrity) |
| | `/gse:fix` | Apply fixes from review in isolated branch |
| **Capitalization** | `/gse:compound` | Capitalize learnings across 3 axes |
| | `/gse:integrate` | Route solutions to project/methodology/learning |
| **Ad-hoc** | `/gse:task` | Execute a task outside the standard lifecycle |

## Architecture

```
gse-one/
├── src/                        # Shared source of truth
│   ├── principles/             # 16 core principles (P1-P16)
│   ├── activities/             # 22 activity definitions
│   ├── agents/                 # 9 agents (8 specialized + orchestrator)
│   └── templates/              # 15 artefact & config templates
├── plugin/                     # Single deployable directory (both platforms)
│   ├── .claude-plugin/         # Claude Code manifest
│   ├── .cursor-plugin/         # Cursor manifest
│   ├── skills/                 # 22 skills (shared)
│   ├── agents/                 # 9 agents (shared)
│   ├── templates/              # 15 templates (shared)
│   ├── rules/                  # 1 .mdc (Cursor-only, ignored by Claude)
│   ├── hooks/                  # 2 hooks files (1 per platform)
│   └── settings.json           # Claude-only (ignored by Cursor)
├── marketplace/                # Marketplace metadata
└── gse_generate.py             # Generator: src/ → plugin/
```

## Core Principles (16)

| Category | Principles |
|----------|-----------|
| **Foundations** | P1 Iterative, P2 Agile, P3 Artefacts, P5 Planning, P6 Traceability |
| **Risk & Communication** | P4 Human-in-loop, P7 Risk Classification, P8 Consequence Visibility, P9 Adaptive Comm, P10 Complexity Budget, P11 Guardrails |
| **Infrastructure** | P12 Version Control, P13 Hooks, P14 Knowledge Transfer |
| **AI Integrity** | P15 Agent Fallibility, P16 Adversarial Review |

## License

Business Source License 1.1 — see [LICENSE](LICENSE) for details.
