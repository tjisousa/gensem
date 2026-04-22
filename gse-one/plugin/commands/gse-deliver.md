---
name: "gse-deliver"
description: "Merge feature branches, tag release, cleanup. Triggered by /gse:deliver."
---


# GSE-One Deliver — Delivery

Arguments: $ARGUMENTS

## Options

| Flag / Sub-command | Description |
|--------------------|-------------|
| (no args)          | Deliver all completed tasks for the current sprint |
| `--dry-run`        | Show merge plan without executing |
| `--skip-tag`       | Skip version tagging |
| `--skip-cleanup`   | Skip branch and worktree cleanup |
| `--help`           | Show this command's usage summary |

## Prerequisites

Before executing, read:
1. `.gse/status.yaml` — current sprint and lifecycle state
2. `.gse/config.yaml` — git strategy, `tag_on_deliver`, `post_tag_hook`, `backup_retention_days`
3. `.gse/backlog.yaml` — all tasks and their statuses (must all be `done` or `delivered`)

## Workflow

### Step 0 — Safety: Backup Tags

Before any destructive operation, create **two classes of backup tags** per feature branch, aligned with spec §10.6 and design §5.15:

**Tag class 1 — merge reversal** (created BEFORE the merge into the integration branch):

```bash
git tag gse-backup/sprint-{NN}-pre-merge-{type}-{name} $(git rev-parse gse/sprint-{NN}/integration)
```

This tags the **integration branch ref** at its state before each merge. To revert a problematic merge:
```bash
git checkout gse/sprint-{NN}/integration && git reset --hard gse-backup/sprint-{NN}-pre-merge-{type}-{name}
```

**Tag class 2 — branch recovery** (created BEFORE each feature branch is deleted in Step 5 cleanup):

```bash
git tag gse-backup/sprint-{NN}-{type}-{name}-deleted $(git rev-parse gse/sprint-{NN}/{type}/{name})
```

This tags the **feature branch ref** at its last commit. To recreate an accidentally-deleted feature branch:
```bash
git checkout -b gse/sprint-{NN}/{type}/{name} gse-backup/sprint-{NN}-{type}-{name}-deleted
```

Both tag classes are retained for 30 days by default (configurable via `config.yaml → git.backup_retention_days`). Cleanup happens at the next `/gse:deliver` (see Step 8 — Cleanup Backup Tags).

This ensures rollback is always possible.

### Step 1 — Pre-flight Checks

Verify delivery readiness:

1. **Review status** — Check that all sprint TASKs are ready for delivery (`backlog.yaml` TASK `status: reviewed` or `status: done` — both are terminal "ready to merge" statuses per spec §12.3 Status lifecycle). If any task has `status: in-progress`, `status: planned`, `status: review`, or `status: fixing`:
   - Report: "Task TASK-{ID} is not complete (status: {status})."
   - Present Gate: **Wait** / **Deliver partial** / **Discuss**

2. **Merge conflict detection** — For each feature branch, check for conflicts:
   ```
   git merge-tree $(git merge-base gse/sprint-{NN}/integration gse/sprint-{NN}/{type}/{name}) gse/sprint-{NN}/integration gse/sprint-{NN}/{type}/{name}
   ```
   Report any conflicts found with affected files.

3. **Uncommitted changes** — For each active worktree, check for uncommitted work:
   - If uncommitted changes exist: report and present Gate: **Commit** / **Stash** / **Discard** / **Discuss**

4. **Requirements coverage (REQ→TST traceability)** — For each REQ- in `docs/sprints/sprint-{NN}/reqs.md`, verify that at least one TST- artefact traces to it:
   - **Uncovered REQ with priority `must`** → **Hard guardrail: block delivery.** Report: "Requirement REQ-{NNN} ({description}) has no test covering it. Tests must be written before delivery." For beginners: "One of the things the app should do hasn't been verified yet. I need to add a check for it first."
   - **Uncovered REQ with priority `should` or `could`** → **Soft guardrail: warn.** Report: "Requirement REQ-{NNN} has no test. This is acceptable but noted." Add a RVW- finding.
   - **All must-priority REQs covered** → Proceed.

### Step 1.5 — Test Execution Evidence

For each TASK in the sprint that implements at least one must-priority REQ (via `traces.implements` in `backlog.yaml`), read `test_evidence.status` (per §12.3 Backlog schema):

- **`test_evidence.status: pass`** → Proceed silently.
- **`test_evidence.status` in `{absent, fail, skipped}`** → **Hard guardrail: block delivery.** Present Gate with 4 options:
  1. **Run tests now** (recommended default) — invoke `/gse:tests --run` inline for the affected TASKs; re-read evidence; re-evaluate guardrail. If evidence becomes `pass` for all, Proceed.
  2. **Deliver partial** — deliver only the TASKs whose REQs have `test_evidence.status: pass`; the others stay in the sprint, move to `pool`, or get re-planned (user choice). Requires DEC- documenting the partial scope.
  3. **Reclassify as spike / deferred** — the TASK is reclassified (`artefact_type: spike` or moves to pool with `priority: could`), requires DEC- documenting the justification.
  4. **Discuss** — explain the implications at the user's expertise level (P9).

  For beginners: *"Before I can deliver, I need to verify that what you asked for actually works. Some tests haven't run yet. Should I run them now?"*

- **Stale evidence** (`test_evidence.timestamp` predates latest feature-branch commit) → Soft guardrail: warn, suggest re-running. Non-blocking.

This step enforces the canonical contract defined in spec §6.3 — Test Execution and Evidence (line 1586: *"Write `test_evidence` on each covered TASK"*) and spec §9.3.1 Test-Specific Guardrails. It is the missing consumer of the `test_evidence` field.

### Step 2 — Merge Features into Sprint Branch

For each feature branch (in dependency order):

1. **Present merge strategy Gate** — Adaptive presentation by expertise level:
   - **beginner**: "How should I combine this work? **Clean summary** (one commit per feature) vs **Full history** (keep all commits)"
   - **intermediate**: "Merge strategy for `{branch}`: **squash** (clean history) / **merge** (preserve commits) / **rebase** (linear history)"
   - **expert**: "Strategy for `{branch}`: squash / merge --no-ff / rebase / discuss"

2. **Execute chosen merge**:
   - **squash**: `git checkout gse/sprint-{NN}/integration && git merge --squash gse/sprint-{NN}/{type}/{name} && git commit`
   - **merge**: `git checkout gse/sprint-{NN}/integration && git merge --no-ff gse/sprint-{NN}/{type}/{name}`
   - **rebase**: `git checkout gse/sprint-{NN}/{type}/{name} && git rebase gse/sprint-{NN}/integration && git checkout gse/sprint-{NN}/integration && git merge --ff-only gse/sprint-{NN}/{type}/{name}`

3. If merge fails due to conflicts:
   - Report conflicting files
   - Present Gate: **Resolve** (attempt auto-resolution) / **Manual** (pause for user) / **Abort** (undo merge) / **Discuss**

### Step 3 — Merge Sprint into Main

1. **Present merge strategy Gate**:
   - **merge** — `git merge --no-ff gse/sprint-{NN}/integration` (default, preserves sprint boundary)
   - **squash** — Collapse entire sprint into one commit
   - **defer** — Keep sprint branch, do not merge yet
   - **discuss** — Explore options

2. Execute the chosen strategy:
   ```
   git checkout main
   git merge --no-ff gse/sprint-{NN}/integration -m "gse(deliver): sprint S{NN} delivery"
   ```

### Step 4 — Tag Release

If `config.yaml` field `tag_on_deliver` is `true` (and `--skip-tag` was not specified):

1. Determine version by analyzing sprint content:
   - **major** — Breaking changes or major new features
   - **minor** — New features, non-breaking
   - **patch** — Bug fixes, documentation, refactoring only
2. Read last tag to compute next version
3. Create annotated tag:
   ```
   git tag -a v{version} -m "Release v{version} — Sprint {NN}"
   ```
4. **Remote backup prompt** — If no git remote is configured (`git remote` returns empty):
   ```
   Your project has no remote. Push to GitHub/GitLab to protect
   your work against local disk failure?
   1. Yes, help me set up a remote
   2. Not now
   3. Discuss
   ```
   This is an Inform-tier suggestion, not a blocker.

### Step 5 — Cleanup

Unless `--skip-cleanup` was specified:

1. **Delete merged feature branches**:
   ```
   git branch -d gse/sprint-{NN}/{type}/{name}
   ```

2. **Remove worktree directories**:
   ```
   git worktree remove .worktrees/sprint-{NN}-{type}-{name}
   ```

3. **Sprint branch cleanup** — Present Gate:
   - **Delete** — `git branch -d gse/sprint-{NN}/integration` (merged, safe)
   - **Keep** — Retain for reference
   - Default: delete if fully merged

4. **Update TASKs** in `backlog.yaml`:
   - `status: delivered`
   - `delivered_at: {timestamp}`
   - `git.branch_status: deleted`
   - `git.worktree_status: removed`

### Step 6 — Release Notes

Generate release notes at `docs/sprints/sprint-{NN}/release.md`:

```markdown
# Sprint S{NN} — Release Notes

**Version**: v{version}
**Date**: {date}
**Sprint**: S{NN}

## Delivered

| Task | Type | Description | Complexity |
|------|------|-------------|------------|
| TASK-{ID} | {type} | {title} | {complexity} |

## Review Summary
- Findings: {count} ({high} HIGH, {medium} MEDIUM, {low} LOW)
- All findings resolved: {yes/no}

## Test Summary
- Tests run: {count}
- Passed: {count}
- Failed: {count}

## Health Score
- Before: {score_before}
- After: {score_after}
```

### Step 7 — Post-Delivery Hook

If `config.yaml` field `post_tag_hook` is configured:

1. Execute the hook command
2. If hook **succeeds**: report success
3. If hook **fails**: present Gate:
   - **Retry** — Re-execute the hook
   - **Rollback** — Remove tag, undo merge (use backup tags)
   - **Investigate** — Examine hook output and diagnose
   - **Discuss** — Explore alternatives

### Step 8 — Cleanup Backup Tags

Remove backup tags older than `backup_retention_days` (default: 30):

1. List tags matching `gse-backup/*` with `git tag -l "gse-backup/*"`
2. For each tag older than the retention period, delete it with `git tag -d <tag>`

### Step 9 — Finalize

1. **Generate sprint plan snapshot** — Read `.gse/plan.yaml` and produce a read-only archive at `docs/sprints/sprint-{NN}/plan-summary.md` using the `plan-summary.md` template. **Inherit the artefact ID** from `plan.yaml.id` (typically `PLN-NNN`) into the snapshot's frontmatter `gse.id` field — this preserves P6 traceability across the live-plan → archive transition. The snapshot contains: goal, mode, budget (total/consumed/remaining), tasks delivered (from `backlog.yaml`), activity flow (from `workflow.completed` + `workflow.skipped` with reasons), scope changes (from `coherence.scope_changes`), coherence summary (alerts raised and whether resolved), and risks. This file is **read-only** — never read by the orchestrator, used only for human reference, COMPOUND process-deviation analysis, and sprint history.
2. Set `plan.yaml.status: completed` and update `plan.yaml.updated` to the current timestamp.
3. Update `status.yaml` **activity-local transitions only**:
   - `current_phase: LC03` — `/gse:deliver` is the **last LC02 activity** per spec §14 ladder; Step 9.3 marks the post-delivery transition to LC03 (Capitalization). `/gse:compound` and `/gse:integrate` then operate in LC03. The phase transition is DELIVER-specific because DELIVER is the activity that performs it — the orchestrator cannot infer LC03 from last_activity alone.
   - **Refresh all 8 health dimension scores** using the same formulas as review (final snapshot for the delivered sprint). This ensures the dashboard health radar reflects the state at delivery time.
   - Cursor fields (`last_activity`, `last_activity_timestamp`) are maintained centrally by the orchestrator after the activity closes — see `agents/gse-orchestrator.md` — section "Sprint Plan Maintenance", and `gse-one-implementation-design.md` §10.1 — Sprint Plan Lifecycle (v0.53.0).
4. Report delivery summary:
   - Tasks delivered
   - Version tagged (if applicable)
   - Branches cleaned
   - Plan snapshot: `docs/sprints/sprint-{NN}/plan-summary.md`
   - Next step: propose `/gse:compound`
5. **Regenerate dashboard** — Run `python3 "$(cat ~/.gse-one)/tools/dashboard.py"` to update `docs/dashboard.html` with delivery status and release info.
