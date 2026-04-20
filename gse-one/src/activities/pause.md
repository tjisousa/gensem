---
description: "Auto-commit work, save session checkpoint. Triggered by /gse:pause."
---

# GSE-One Pause — Session Pause

Arguments: $ARGUMENTS

## Options

| Flag / Sub-command | Description |
|--------------------|-------------|
| (no args)          | Auto-commit all active worktrees and save checkpoint |
| `--no-commit`      | Save checkpoint without auto-committing |
| `--note "text"`    | Attach a note to the checkpoint (context for future resume) |
| `--help`           | Show this command's usage summary |

## Prerequisites

Before executing, read:
1. `.gse/status.yaml` — current sprint and lifecycle state
2. `.gse/config.yaml` — git strategy
3. `.gse/backlog.yaml` — active tasks and their worktree assignments

## Workflow

### Step 1 — Auto-Commit Active Worktrees

Unless `--no-commit` was specified:

For each TASK in `backlog.yaml` with `git.worktree_status: active`:

1. Navigate to the worktree directory: `.worktrees/{worktree_name}`
2. Check for uncommitted changes: `git status --porcelain`
3. If changes exist:
   ```
   git add -A
   git commit -m "gse(pause): checkpoint — {task_title}

   Sprint: S{NN}
   Task: TASK-{ID}
   Reason: session pause
   "
   ```
4. Update TASK in `backlog.yaml`:
   - `git.uncommitted_changes: 0`
   - `git.last_pause_commit: {hash}`

If no worktrees are active (strategy is `branch-only` or `none`):
1. Check current branch for uncommitted changes
2. If changes exist, auto-commit with the same convention

Report: "Auto-committed {N} worktree(s) with changes."

### Step 2 — Save Checkpoint

Create a checkpoint file at `.gse/checkpoints/checkpoint-{YYYY-MM-DD-HHMM}.yaml` using the checkpoint.yaml template (`plugin/templates/checkpoint.yaml` — authoritative schema). Populate all fields from the current session state:

- `checkpoint.timestamp`: current ISO 8601 timestamp
- `checkpoint.user`: from `profile.yaml → user.name`
- `checkpoint.sprint`, `checkpoint.phase`: from `status.yaml`
- `checkpoint.last_activity`, `checkpoint.last_task`: from current session context
- `checkpoint.note`: from `--note` flag if provided, else empty
- `status_snapshot`: extract `lifecycle_phase`, `current_sprint`, `last_activity`, `last_activity_timestamp`, `health.score` from `status.yaml`
- `backlog_sprint_snapshot.tasks`: for each TASK in current sprint, record `status`, `complexity`, `branch`
- `git_state.current_branch`: from `git branch --show-current`
- `git_state.worktrees[]`: from `git worktree list`, for each: `path`, `branch`, `task` (from backlog), `last_commit` (HEAD hash), `clean` (from `git status`)

### Step 3 — Update Status

1. Update `status.yaml`:
   - `last_activity: pause`
   - `last_activity_timestamp: {now}`
   - `session_paused: true`
   - `pause_checkpoint: checkpoint-{YYYY-MM-DD-HHMM}.yaml`

### Step 4 — Report

```
SESSION PAUSED
  Checkpoint: .gse/checkpoints/checkpoint-{YYYY-MM-DD-HHMM}.yaml
  Auto-committed: {N} worktree(s)
  Sprint: S{NN} ({lifecycle_phase})
  Active tasks: {count}
  
  Resume with: /gse:resume
```
