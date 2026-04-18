# GSE-One — opencode Quickstart

> **Goal:** install GSE-One on opencode and start working on a new project — in under 5 minutes.

This guide is the fastest path. For the full reference (Claude Code and Cursor too), see the main [README.md](README.md).

---

## 1. Prerequisites

You need:

- **opencode** installed and on your `PATH`. Check with `opencode --version`. Install instructions: https://opencode.ai
- **Python 3** (`python3 --version`). Ships with macOS and Linux; on Windows use the `python` command.
- **git** — used to clone this repo and to track your project.

That's it. No Node.js, no Bun install, no npm packages — opencode handles the TS plugin runtime itself.

---

## 2. Install (60 seconds)

Choose **one** of the two modes below. Non-plugin is simplest for a single project; plugin is better if you want GSE-One available in every project.

### Mode A — Non-plugin (per-project, recommended for your first try)

Copies GSE-One into `<project>/.opencode/` plus an `AGENTS.md` and `opencode.json` at the project root. Nothing touches your home directory except a 1-line registry (`~/.gse-one`) used to resolve tools at runtime.

```bash
# 1. Clone this repo somewhere (does NOT need to be inside your project)
git clone https://github.com/nicolasguelfi/gensem.git
cd gensem

# 2. Install GSE-One into your project's .opencode/
python3 install.py --platform opencode --mode no-plugin --project-dir /path/to/your-project
```

### Mode B — Plugin (global, ~/.config/opencode/)

Installs once; every opencode session on any project sees GSE-One.

```bash
git clone https://github.com/nicolasguelfi/gensem.git
cd gensem
python3 install.py --platform opencode --mode plugin
```

### Interactive alternative

```bash
python3 install.py
```

The installer auto-detects opencode, Claude Code, and Cursor, and walks you through the choices.

---

## 3. Start a new project and work (2 minutes)

### From scratch

```bash
# 1. Create your project directory
mkdir ~/my-project && cd ~/my-project
git init

# 2. Install GSE-One into it (skip if you picked Mode B above)
cd /path/to/gensem
python3 install.py --platform opencode --mode no-plugin --project-dir ~/my-project
cd ~/my-project

# 3. Launch opencode
opencode
```

### First command — let GSE-One drive

In opencode, type:

```
/gse-go
```

That's the entry point. GSE-One will:

1. Detect that this is a fresh project and ask a few profiling questions (your role, your goals, your preferred language).
2. Propose the first activity — usually `/gse-plan` (define what to build) or `/gse-hug` (onboarding interview).
3. From there, it orchestrates the full lifecycle (`PLAN → REQS → DESIGN → PREVIEW → TESTS → PRODUCE → REVIEW → FIX → DELIVER`) — you just confirm or redirect at each step.

You never need to memorize commands. `/gse-go` picks the next activity based on the project state.

---

## 4. What gets installed, exactly

| File | Mode A (no-plugin) | Mode B (plugin) |
|------|:------------------:|:---------------:|
| `<project>/.opencode/skills/<name>/SKILL.md` × 23 | ✓ | — |
| `<project>/.opencode/commands/gse-<name>.md` × 23 | ✓ | — |
| `<project>/.opencode/agents/<name>.md` × 8 | ✓ | — |
| `<project>/.opencode/plugins/gse-guardrails.ts` | ✓ | — |
| `<project>/AGENTS.md` (GSE-One block between markers) | ✓ | — |
| `<project>/opencode.json` (deep-merged) | ✓ | — |
| `~/.config/opencode/skills/...` etc. | — | ✓ |
| `~/.config/opencode/AGENTS.md` | — | ✓ |
| `~/.config/opencode/opencode.json` | — | ✓ |
| `~/.gse-one` (1-line registry for Python tools) | ✓ | ✓ |

**All GSE-One content in `AGENTS.md` sits between `<!-- GSE-ONE START -->` and `<!-- GSE-ONE END -->` markers.** Your own content outside those markers is preserved on reinstall and fully restored on uninstall. `opencode.json` is deep-merged — your custom keys are never overwritten.

---

## 5. Command reference (abridged)

Type any of these in opencode:

| Command | What it does |
|---------|--------------|
| `/gse-go` | **Start here.** Detects project state, proposes next activity. |
| `/gse-status` | Sprint state, artefact inventory, git state. |
| `/gse-health` | 8-dimension project health dashboard. |
| `/gse-plan` | Select backlog items for the next sprint. |
| `/gse-reqs` | Define requirements with test-driven acceptance criteria. |
| `/gse-design` | Architecture decisions, component decomposition. |
| `/gse-produce` | Execute production in an isolated worktree. |
| `/gse-review` | Code review + devil's advocate pass. |
| `/gse-deliver` | Merge, tag, cleanup branches. |
| `/gse-pause` / `/gse-resume` | Save and restore session state. |

Full list: 23 commands. Type `/gse-` and your completion should surface all of them. Details per command: [gse-one-spec.md](gse-one-spec.md) §3.

---

## 6. Upgrading

```bash
cd /path/to/gensem
git pull
cd gse-one && python3 gse_generate.py --verify
cd ..
# Reinstall — the existing install is overwritten cleanly
python3 install.py --platform opencode --mode no-plugin --project-dir /path/to/your-project
```

Reinstall is idempotent: the GSE-One block in `AGENTS.md` is surgically replaced; GSE-managed keys in `opencode.json` are updated without touching your custom keys.

---

## 7. Uninstalling

```bash
# Mode A (non-plugin)
python3 install.py --uninstall --platform opencode --mode no-plugin --project-dir /path/to/your-project

# Mode B (plugin, global)
python3 install.py --uninstall --platform opencode --mode plugin
```

After uninstall:

- `.opencode/` is emptied of GSE-One content (user-added files kept).
- `AGENTS.md` has its GSE-One block removed; if nothing else remains, the file is deleted.
- `opencode.json` loses the `gse` marker and the GSE-added bash denies; if only the `$schema` remains, the file is deleted.
- `~/.gse-one` registry is removed.

---

## 8. Troubleshooting

- **`/gse-*` commands don't appear in opencode** — verify `.opencode/commands/gse-*.md` exists (Mode A) or `~/.config/opencode/commands/gse-*.md` exists (Mode B). Restart opencode after install.
- **`/gse-go` runs but the agent doesn't apply GSE-One methodology** — `AGENTS.md` is missing or its GSE-ONE block was corrupted. Reinstall.
- **"Skill skipped: missing name/description"** — the generator must be re-run: `cd gse-one && python3 gse_generate.py --verify`. The `name:` field is required by opencode's loader.
- **Guardrails not firing on `git commit` on main** — ensure `.opencode/plugins/gse-guardrails.ts` exists. opencode auto-loads every `.ts` file under `plugins/`.
- **Both `.claude/skills/` and `.opencode/skills/` in the same project** — opencode scans both and duplicates commands. The installer warns at install time and offers to abort; if you hit this, remove one of the two directories.

---

## 9. Cross-reference

- Main README (all platforms): [README.md](README.md)
- Full methodology spec: [gse-one-spec.md](gse-one-spec.md)
- Implementation & design: [gse-one-implementation-design.md](gse-one-implementation-design.md)
- Changelog: [CHANGELOG.md](CHANGELOG.md)
