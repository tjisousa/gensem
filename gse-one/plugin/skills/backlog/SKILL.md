---
name: backlog
description: "View and manage project backlog. Triggered when user asks about tasks, work items, what's left to do."
---

# GSE-One Backlog — Unified Work Item Management

Arguments: $ARGUMENTS

## Options

| Flag / Sub-command | Description |
|--------------------|-------------|
| (no args)          | Display the full backlog grouped by sprint and pool |
| `add <description>` | Add a new work item to the pool |
| `sprint`           | Show only items in the current sprint |
| `pool`             | Show only items in the unassigned pool |
| `--type <type>`    | Filter by artefact type (code, requirement, design, test, doc, config, import) |
| `sync`             | Synchronize backlog with GitHub Issues (bidirectional) |
| `--help`           | Show this command's usage summary |

## Prerequisites

Before executing, read:
1. `.gse/backlog.yaml` — current backlog state
2. `.gse/status.yaml` — current sprint number
3. `.gse/config.yaml` — GitHub integration settings (if enabled)
4. `.gse/profile.yaml` — user profile (apply P9 Adaptive Communication to all output)
5. `.gse/plan.yaml` — living sprint plan (used for desync detection in the `sprint` sub-command)

## Workflow

> **Workflow structure note.** `/gse:backlog` exposes three disjoint modes (Display, Add, Sync). Each mode defines its own Step sequence (`#### Step 1..N`), numbering resets per mode. The user invokes exactly one mode per call. This **multi-mode `### Mode → #### Step N` structure** is shared with `/gse:plan`, `/gse:collect`, `/gse:learn`; see CLAUDE.md — §Activity structural conventions for the full catalog.

### Display Mode (No Args / `sprint` / `pool`)

#### Step 1 — Load Backlog

Read `.gse/backlog.yaml` and parse all work items.

#### Step 2 — Group and Format

Display items grouped by assignment:

```
Sprint S02 (current)
─────────────────────
  TASK-007 [code]     ● in-progress  feat/user-auth       Implement JWT authentication
  TASK-008 [test]     ○ planned      feat/user-auth-test   Write auth endpoint tests
  TASK-009 [doc]      ○ planned      —                     Document auth API

Sprint S01 (delivered)
─────────────────────
  TASK-001 [code]     ✓ done         feat/project-init     Initialize project structure
  TASK-002 [design]   ✓ done         feat/db-schema        Design database schema
  TASK-003 [test]     ✓ done         feat/db-schema-test   Write schema migration tests

Pool (unassigned)
─────────────────
  TASK-010 [code]     ◌ pool         —                     Add rate limiting middleware
  TASK-011 [design]   ◌ pool         —                     Design notification system
  TASK-012 [requirement] ◌ pool      —                     Define admin role permissions
```

Status symbols:
- `✓` done
- `●` in-progress
- `○` planned (assigned to sprint but not started)
- `◌` open (displayed as "Pool" — unassigned to any sprint)

#### Step 3 — Summary Line

Show totals: "12 items total: 3 done, 1 in-progress, 2 planned, 6 in pool"

#### Step 4 — Plan Sync Check (if `.gse/plan.yaml` exists with `status: active`)

Compute the set of TASK IDs currently in the sprint from `backlog.yaml` (filter: `sprint == current_sprint` AND `status != delivered`) and compare to `plan.yaml.tasks[].id`.

- **If sets are identical:** no output (silent — the plan is in sync).
- **If sets differ** (tasks added, removed, or renamed since PLAN): display a **warning line** after the summary:
  ```
  ⚠  Plan out of sync: {N} task(s) added/removed since PLAN.
     Run `/gse:plan --tactical` to sync `.gse/plan.yaml` with the backlog.
  ```
  For beginners, translate per P9: "Your task list has changed since we organized the work. I should re-organize it — want me to do that now?"

This is an **Inform-tier** detection — it does not block backlog display or any other operation. It is the early-warning counterpart to the orchestrator's coherence check at activity transitions (which runs later, only when a transition happens).

Skip this step in Micro mode (no `plan.yaml` exists) and for the `pool` sub-command (pool items are always unplanned).

### Add Mode (`add <description>`)

#### Step 1 — Auto-Increment ID

Read `.gse/backlog.yaml`, find the highest TASK-NNN ID, increment by 1.

#### Step 2 — Infer Artefact Type

From the description, infer the most likely type:

| Keywords | Inferred Type |
|----------|--------------|
| implement, build, create, add feature, code | `code` |
| test, verify, validate, check | `test` |
| design, architect, structure, schema | `design` |
| require, must, should, user story | `requirement` |
| document, write docs, README | `doc` |
| configure, setup, CI, deploy | `config` |

If ambiguous, default to `code` and note the assumption.

#### Step 3 — Create Entry

Add to `.gse/backlog.yaml`:

```yaml
- id: TASK-013
  artefact_type: code
  title: "Add rate limiting middleware"
  description: "Implement rate limiting for API endpoints using token bucket algorithm"
  sprint: null  # null = pool (unplanned)
  status: open  # open | planned | in-progress | review | fixing | done | delivered | deferred
  priority: should  # default, can be changed
  complexity: null  # set during planning
  origin: user  # plan | review | collect | user | import | ad-hoc
  created: 2026-01-20T10:30:00Z
  git:
    branch: null
    branch_status: null  # null | planned | active | merged | deleted
  traces:
    derives_from: []
  github_issue: null  # top-level, aligned with backlog.yaml template
```

#### Step 4 — GitHub Issue (If Enabled)

If `.gse/config.yaml → github.enabled: true` AND `github.sync_mode ∈ {on-activity, real-time}`:
- Create a GitHub Issue with matching title and description
- Store the issue number in `github_issue` (top-level field, not nested in traces)
- Apply labels based on artefact type

### Sync Mode (`sync`)

#### Step 1 — Pull from GitHub

Fetch all open GitHub Issues for the repository:
- For each issue NOT in backlog: create a new TASK entry in pool
- For each issue that matches an existing TASK: update status if changed

#### Step 2 — Push to GitHub

For each TASK with `github_issue: null` and (`github.enabled: true` AND `github.sync_mode ∈ {on-activity, real-time}`):
- Create a GitHub Issue
- Store the issue number

#### Step 3 — Bidirectional Status Sync

Map statuses between GSE-One and GitHub:

| GSE-One Status (+ condition) | GitHub State |
|---------------|-------------|
| `open` AND `sprint: null` (pool) | open (label: `pool`) |
| `planned` | open (label: `sprint-NN`) |
| `in-progress` | open (label: `in-progress`) |
| `done` | closed |

Report sync summary: "Synced 15 items. 2 new from GitHub, 1 status updated, 12 unchanged."

### Auto-Creation by Other Skills

Other GSE-One activities create backlog items automatically:

| Source Skill | Created Items |
|-------------|---------------|
| REVIEW | Fix items from review findings (e.g., TASK-014 from RVW-002) |
| PLAN | Promotes pool items to sprint, may split large items |
| COLLECT | Import items from external sources (e.g., GitHub Issues from scanned repos) |
| DELIVER | Close items, create follow-up items for deferred scope |

These auto-created items always include `traces.derives_from` linking to the source artefact.
