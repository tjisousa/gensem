# GSE-One Development Guide

## Project overview

This is the **gensem** repository — the source for the GSE-One plugin, an AI engineering companion for structured SDLC management. It produces a single plugin deployable on both Claude Code and Cursor.

## Repository structure

- `gse-one/src/` — Single source of truth (principles, activities, agents, templates)
- `gse-one/plugin/` — Generated deployable directory (NEVER edit directly)
- `gse-one/plugin/tools/` — Python tool scripts (dashboard, etc.) with `# @gse-tool` headers
- `gse-one/gse_generate.py` — Generator: rebuilds `plugin/` from `src/`
- `install.py` — Cross-platform installer (Claude Code + Cursor)
- `gse-one-spec.md` — Methodology specification
- `gse-one-implementation-design.md` — Implementation design document
- `assets/images/` — Branding assets (logos, banners)
- `_LOCAL/` — Local-only files (gitignored via `/_*/` pattern)
- `VERSION` — Single version source, read by generator and installer

## Critical rules

### Build pipeline
- **Always run `cd gse-one && python3 gse_generate.py --verify` before committing** — it regenerates plugin/ from src/ and bumps manifest versions from VERSION.
- Never edit files in `gse-one/plugin/` directly except `plugin/tools/` — the generator overwrites everything else from `src/`.
- Changes to skills go in `src/activities/`, changes to agents go in `src/agents/`.
- The orchestrator and .mdc rule are generated from the same source — body parity is verified automatically.

### Tool architecture
- Tools live in `plugin/tools/` and are resolved at runtime via the `~/.gse-one` registry file.
- `install.py` writes `~/.gse-one` (absolute plugin path) on install, removes it on uninstall.
- Skills execute tools via: `python3 "$(cat ~/.gse-one)/tools/dashboard.py"` — agents must never read+rewrite tool content.
- Each tool has a `# @gse-tool <name> <version>` header on line 2.

### Versioning
- Bump `VERSION` file, then run the generator — it propagates to both plugin.json manifests.
- Commit style: `feat:`, `fix:`, `docs:` prefixes. Check recent `git log` for conventions.

### Files to keep in sync
- `src/activities/*.md` ↔ `plugin/skills/*/SKILL.md` (via generator)
- `src/agents/gse-orchestrator.md` ↔ `plugin/agents/gse-orchestrator.md` + `plugin/rules/000-gse-methodology.mdc` (via generator)
- Changes in spec should be reflected in design doc changelog and vice versa.

## Language

The user communicates in French. Respond in French for conversation, English for code/commits.
