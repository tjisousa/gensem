---
description: "Execute production plan in isolated worktree. Creates feature branch + worktree per task. Triggered by /gse:produce."
---

# GSE-One Produce — Production

Arguments: $ARGUMENTS

## Options

| Flag / Sub-command | Description |
|--------------------|-------------|
| (no args)          | Execute next pending task from current sprint backlog |
| `--task TASK-ID`   | Execute a specific task by ID |
| `--all`            | Execute all pending tasks sequentially |
| `--dry-run`        | Show what would be produced without executing |
| `--skip-tests`     | Skip automatic test execution after production (**Gate guardrail** — requires confirmation, logged as DEC-, penalizes health score) |
| `--help`           | Show this command's usage summary |

## Prerequisites

Before executing, read:
1. `.gse/status.yaml` — current sprint and lifecycle state
2. `.gse/config.yaml` — project configuration, especially `git.strategy` (worktree/branch-only/none)
3. `.gse/backlog.yaml` — sprint tasks, their status, and assignment
4. `.gse/profile.yaml` — user expertise level (affects test generation behavior)
5. `docs/sprints/sprint-{NN}/reqs.md` — requirements for the current sprint (**mandatory**)
6. `docs/sprints/sprint-{NN}/test-strategy.md` — test strategy (**mandatory**)


## Execution Model

**PRODUCE is an isolated activity** — the orchestrator delegates each TASK to a **sub-agent with a fresh context** (see Context Architecture in the orchestrator). This ensures the production phase has a clean context window, uncluttered by prior COLLECT/ASSESS/PLAN/REQS/DESIGN discussions.

**Before delegation**, the orchestrator:
1. Saves a mini-checkpoint (`.gse/checkpoints/pre-produce-{timestamp}.yaml`)
2. Spawns a sub-agent with: this SKILL.md, state files, sprint artefacts (reqs.md, design.md, test-strategy.md), and the TASK description
3. Claude Code: `Agent` tool with `isolation: "worktree"` (if worktree strategy)
4. Cursor: subagent with isolated context

**After delegation**, the orchestrator reads updated state files and displays the production summary to the user.

If `--all` is specified, each TASK is delegated to its own sub-agent **sequentially** (parallel PRODUCE is not supported — tasks may have dependencies).

## Workflow

### Step 0 — Pre-production Guardrails (Hard — cannot be skipped)

Before starting ANY task, verify these conditions. If any check fails, **STOP and do not proceed**.

**Sprint Freeze Check** (applied before all other Step 0 guardrails) — Read `.gse/plan.yaml`. If absent (Micro mode), skip this check. If `plan.yaml.status == active`, proceed to the numbered guardrails below. If `plan.yaml.status ∈ {completed, abandoned}`, the current sprint is **frozen**: do NOT transition any TASK forward and do NOT write to `backlog.yaml`. Present the Sprint Freeze Gate:

> **Question:** The current sprint (S{NN}) plan status is `{completed|abandoned}` — it is frozen. Production work cannot continue in a frozen sprint.
>
> **Options:**
> 1. **Start next sprint now** (recommended, default) — I'll run the next-sprint opening sequence: in Lightweight mode `/gse:plan --strategic`; in Full mode `/gse:collect` > `/gse:assess` > `/gse:plan --strategic`. Once Sprint S{NN+1} is active, I'll carry the planned production work into it.
> 2. **Cancel** — Abort this `/gse:produce` invocation. No changes will be made.
> 3. **Discuss** — Explore the trade-offs.

On option 1: invoke the mode-appropriate opening sequence inline; after promotion, re-read `.gse/status.yaml → current_sprint` and `.gse/plan.yaml → status`, then proceed to the numbered guardrails below in the new sprint context. On option 2: stop execution. On option 3: explain the Sprint Freeze invariant, then re-present the Gate.

**Standard guardrails (applied after Sprint Freeze passes):**

1. **Requirements check (Full and Lightweight)** — Verify that `docs/sprints/sprint-{NN}/reqs.md` exists and contains at least one REQ- artefact traced to the TASK about to start. If missing: report "Requirements not defined for this task. I need to write down what the app should do first." Then run REQS. **Exception:** Micro mode and `artefact_type: spike` — skip this check.
2. **Test strategy check (mode-differentiated):**
   - **Full mode:** Verify that a test strategy exists (`docs/sprints/sprint-{NN}/test-strategy.md` or `tests` in `.gse/plan.yaml.workflow.completed`). If missing → **Hard guardrail.** Report "Test strategy not defined. I need to describe how we'll verify each feature works." Then run TESTS `--strategy`.
   - **Lightweight mode:** If no test strategy exists → **Soft guardrail (Inform).** Auto-generate a minimal test strategy based on `config.yaml → project.domain` (default test pyramid, no formal artefact). Log in `plan.yaml.coherence` as auto-generated. Continue without blocking.
   - **Micro mode:** Skip this check.
   **Exception: `artefact_type: spike`** — skip this check in all modes.
3. **Preview check (web/mobile only)** — If `config.yaml → project.domain` is `web` or `mobile` and no preview artefact exists (`docs/sprints/sprint-{NN}/preview.md` or equivalent), present a Gate: "A preview was not done for this project. For a visual project, it's recommended to validate the look before coding." Options: **Proceed without preview** / **Run preview first** / **Discuss**. For beginners: "Before I build, it's helpful to show you a sketch of what the app will look like — want me to do that first?" **Exception: `artefact_type: spike`** — skip this check.
4. **Upstream test-review findings check** (spec §6.5) — If `docs/sprints/sprint-{NN}/review.md` exists, scan for unresolved findings tagged `[STRATEGY]` or `[TST-SPEC]` with severity HIGH. If any exist, **Hard guardrail: block production**. Report: "Tests were reviewed upstream and HIGH findings remain open. Resolve them first — writing code against an incorrect test plan wastes the sprint." List the open findings and their IDs. Redirect the user to `/gse:fix --severity HIGH --task TASK-upstream` or manual resolution of the strategy/spec. **Exception: `artefact_type: spike`** — skip this check.

**Decision tier override:**
5. **Supervised mode** — If `profile.decision_involvement` is `supervised`, ALL technical choices in this TASK are escalated to **Gate-tier** decisions. This includes: library/dependency selection, data format, folder structure, persistence strategy, API design, naming conventions. The agent MUST present options and wait for user confirmation — it MUST NOT make these choices silently.

### Step 1 — Select Task

Read `backlog.yaml` and identify tasks for the current sprint with `status: planned` or `status: ready`.

If multiple tasks are pending:
1. Sort by priority (P1 > P2 > P3), then by dependency order
2. Present the next task to produce with summary: ID, title, artefact_type, estimated complexity
3. Wait for user confirmation (Gate) unless `--all` was specified

If no tasks are pending:
1. Report: "All tasks for sprint S{NN} are complete or in-progress."
2. Propose `/gse:review` if unreviewed tasks exist, otherwise `/gse:deliver`

### Step 2 — Git Setup (Before Production)

**Precondition — git baseline:** If `git.strategy` is `worktree` or `branch-only`, verify `main` has at least one commit (`git rev-parse --verify main`). If not → **Hard guardrail**: "The repository has no foundational commit. Run `/gse:hug` or `/gse:go` to initialize it before producing."

**Git branch check:** Before starting work on any TASK, verify that the current branch is NOT `main`. If on `main`, remind the user that the methodology recommends working on a dedicated branch and create one per the convention below. If the user explicitly chooses to stay on `main`, respect the choice and note it as a known process deviation in the sprint review.

Read `config.yaml` field `git.strategy` and branch accordingly:

#### Strategy: `worktree` (default)

1. Determine the sprint integration branch: `gse/sprint-{NN}/integration`
2. Create feature branch from sprint integration branch:
   ```
   git branch gse/sprint-{NN}/{type}/{name} gse/sprint-{NN}/integration
   ```
   Where `{type}` is the artefact type (feat, fix, doc, test, refactor) and `{name}` is a slug of the task title.
3. Create worktree for the feature branch:
   ```
   git worktree add .worktrees/sprint-{NN}-{type}-{name} gse/sprint-{NN}/{type}/{name}
   ```
4. Update the TASK in `backlog.yaml`:
   - `status: in-progress`
   - `git.branch: gse/sprint-{NN}/{type}/{name}`
   - `git.branch_status: active`
   - `git.worktree: .worktrees/sprint-{NN}-{type}-{name}`
   - `git.worktree_status: active`
5. All subsequent file operations happen inside the worktree directory.

#### Strategy: `branch-only`

1. Determine base branch:
   - If sprint integration branch `gse/sprint-{NN}/integration` exists (full mode): branch from it.
   - If no sprint branch exists (Lightweight mode): branch from `main` directly.
2. Create feature branch:
   ```
   # Full mode:
   git branch gse/sprint-{NN}/{type}/{name} gse/sprint-{NN}/integration
   git checkout gse/sprint-{NN}/{type}/{name}

   # Lightweight mode (no sprint branch):
   git branch gse/{type}/{name} main
   git checkout gse/{type}/{name}
   ```
3. Update the TASK in `backlog.yaml`:
   - `status: in-progress`
   - `git.branch: gse/sprint-{NN}/{type}/{name}` (full) or `gse/{type}/{name}` (lightweight)
   - `git.branch_status: active`

#### Strategy: `none`

1. Work directly on current branch (no branch creation).
2. Update the TASK in `backlog.yaml`:
   - `status: in-progress`

### Step 3 — Execute Production

Produce the artefact according to the task specification:

1. Read the task description and acceptance criteria from `backlog.yaml`
2. If the task references requirements (REQ-NNN), read them from the requirements artefact
3. If the task references design decisions (DES-NNN), read them from the design artefact
4. Execute the work, creating or modifying files as specified
5. **Pre-commit self-review (P16 — before every commit):**
   Before committing, the agent MUST run these 5 checks on the changes about to be committed:
   - **(a) Hallucination hunt** — verify that all referenced APIs, libraries, and functions actually exist in the versions used
   - **(b) Assumption check** — list any assumptions made during implementation; flag those not validated by a REQ-
   - **(c) Complaisance check** — does the code match exactly what was asked in the requirements? No extra features, no missing features?
   - **(d) Edge cases** — have boundary conditions been considered (empty input, max values, concurrent access, error states)?
   - **(e) Temporal validity** — are the dependency versions, API endpoints, and syntax current and not deprecated?
   If any check fails, fix before committing. If uncertain, flag as a finding for REVIEW.
6. Commit at logical checkpoints using the convention:
   ```
   gse(sprint-{NN}/{type}): description

   Sprint: {NN}
   Task: TASK-{ID}
   Traces: [REQ-NNN, DES-NNN] (if applicable)
   ```

### Step 4 — Test Execution (After Production)

PRODUCE invokes the **canonical test run** defined in spec §6.3. The seven canonical steps (execute → capture → save → TCP report → test_evidence → inline summary → health update) are never skipped and never duplicated here. This step only documents the PRODUCE-specific pre/post-conditions around that canonical call.

**Pre-condition A — `--skip-tests` flag:**
1. Present a **Gate decision** (cannot be silently bypassed): "Skipping tests means we won't verify this task works correctly. Are you sure?" Options: **Skip tests** / **Run tests anyway** / **Discuss**. For beginners: "I'd normally check that what I built works correctly. Do you want me to skip that check? I don't recommend it."
2. If confirmed: record a DEC- artefact in the decision journal with rationale; write `test_evidence.status: skipped` on the TASK; health score `test_pass_rate` receives a penalty for the skipped task.
3. In `supervised` mode: require **double confirmation** ("This is unusual — please confirm again").
4. **Do not invoke** the canonical run.

**Pre-condition B — no tests exist** and the artefact type warrants testing. Read `profile.yaml → user.it_expertise`:
- **beginner**: Auto-generate tests, inform user ("I've created tests for this task.")
- **intermediate**: Propose ("This task should have tests. Shall I generate them?")
- **expert**: Propose with options ("No tests found. Options: generate unit tests / generate integration tests / skip / discuss")

After generation (if any), proceed to the canonical run.

**Run** — invoke the canonical test run (spec §6.3). All outputs (TCP-NNN campaign, test_evidence update, inline summary, health refresh) are produced by that procedure.

**Post-condition — failure handling:** if the canonical run ends with `test_evidence.status: fail`, present a Gate decision:
- **Fix** — attempt to fix the failing tests, then re-invoke the canonical run (default)
- **Skip** — leave `status: fail`, continue; flagged as HIGH in the next review
- **Discuss** — explore the failure with the user

### Step 5 — Finalize

1. Ensure all changes are committed (no uncommitted work in worktree)
2. Update TASK in `backlog.yaml`:
   - `status: done`
   - `completed_at: {timestamp}`
   - `git.uncommitted_changes: 0`
3. Update `status.yaml`:
   - `last_activity: produce`
   - `last_activity_timestamp: {now}`
   - `last_task: TASK-{ID}`
4. Update complexity budget: subtract task complexity from sprint remaining budget
5. **Beginner auto-execution** — For beginner users, do NOT list commands for the user to run (`npm install`, `npm run dev`, `npm test`, `pip install`, etc.). Instead, **execute them automatically** and report the result in plain language. Say "I'm starting the application for you" instead of "Run `npm run dev`". The beginner should never need to type a terminal command. If a command requires user interaction (e.g., opening a URL in a browser), give clear instructions: "Open this link in your browser: http://localhost:5173".
6. **Manual testing procedure** — After each completed task, provide the user with a step-by-step procedure to manually verify the result. Adapt to the project type: for web apps, the URL and actions to perform in the browser; for APIs, the curl commands or test tool instructions; for CLIs, the commands to run with expected output; for libraries, a usage example. For beginners, write the procedure in simple language with numbered steps. For experts, a concise summary is sufficient. The goal is to enable the user to validate the produced work themselves — complementing automated tests with human verification.
7. Report production summary:
   - What was produced (in beginner terms: feature descriptions, not file paths)
   - Test Campaign Summary (already shown inline during Step 4 — reiterate at summary time)
   - Manual testing procedure (from Step 6)
   - Remaining sprint budget (for intermediate/expert; hidden for beginner)
   - Next task suggestion (if any)
8. **Regenerate dashboard** — Run `python3 "$(cat ~/.gse-one)/tools/dashboard.py"` to update `docs/dashboard.html` with the final task status (done) and budget consumption. Note: the dashboard was already refreshed right after the test run (step 7 of the canonical run); this second regen captures the TASK status transition.
