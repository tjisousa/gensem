# GSE-One — Training Feedback Report

**What your test sessions changed in the plugin**

*Covering releases v0.23.0 → v0.38.0 · April 2026*

---

## 1. Introduction

You spent three days running GSE-One on real project work. Your session transcripts (Cursor agent dialogues) and your handwritten notes were read back one by one. Twenty distinct methodological improvements were extracted from that material. This report explains what was taken into account and how — release by release, card by card.

**Key numbers**

- **20** improvement proposals identified across the three days of testing.
- **19** implemented as code, spec and design changes.
- **1** absorbed into a related improvement (one less file to read, one consistent rule).
- **1** ancillary fix discovered along the way (generator safety).
- **15** plugin releases, from **v0.23.0** to **v0.38.0**.
- Three layers kept in sync throughout: **specification** (`gse-one-spec.md`), **design** (`gse-one-implementation-design.md`), **implementation** (`gse-one/src/**` and generated `plugin/`).

**Where to find the product**

- Repository: <https://github.com/nicolasguelfi/gensem>
- Full release history: [`CHANGELOG.md`](https://github.com/nicolasguelfi/gensem/blob/main/CHANGELOG.md)
- Specification: [`gse-one-spec.md`](https://github.com/nicolasguelfi/gensem/blob/main/gse-one-spec.md)
- Design: [`gse-one-implementation-design.md`](https://github.com/nicolasguelfi/gensem/blob/main/gse-one-implementation-design.md)

---

## 2. How to read this report

Each improvement is presented as a **card** with a uniform shape:

1. **The problem in one sentence** — what went wrong, in plain language.
2. **What we did** — the change, not the diff. The rule or concept, not the code.
3. **Before / After example** — a short narrative or snippet showing the effect.
4. **Go further** — pointers to the canonical sources (spec, design, commit).

You can read the cards in order, or jump to whichever matches the situation you encountered. The **Synthesis** table (§3) groups them by nature so you can pick a theme rather than an order.

A small amount of vocabulary is unavoidable — a one-line glossary lives in [Annex B](#annex-b--glossary).

---

## 3. Synthesis — the 20 improvements at a glance

Each improvement has a type tag:

- 🛡 **Guardrail** — blocks, gates or detects a risky situation.
- 📐 **New concept** — introduces a methodological idea (artefact type, activity step, principle clarification).
- 🔧 **Assistance mechanism** — automation or aid that removes friction without constraining.
- 🎯 **Precision** — clarifies, unifies or sharpens an existing rule.
- 🤝 **Feedback loop** — channels improvement back upstream.

| ID | Type | Title | Version | Commit |
|---|---|---|---|---|
| 01 | 🛡 | Sprint Freeze — closed sprints are immutable | v0.23.0 | [`b6f76e4`](https://github.com/nicolasguelfi/gensem/commit/b6f76e4) |
| 02 | 🔧 | Automatic dashboard regeneration | v0.24.0 | [`5d9a501`](https://github.com/nicolasguelfi/gensem/commit/5d9a501) |
| 03 | 🛡 | Root-Cause Discipline before patching | v0.26.0 | [`0206978`](https://github.com/nicolasguelfi/gensem/commit/0206978) |
| 04 | 🛡 | Git Identity Verification at first commit | v0.25.0 | [`53d111d`](https://github.com/nicolasguelfi/gensem/commit/53d111d) |
| 05 | 📐 | Scope Reconciliation + Inform-Tier Summary | v0.27.0 | [`67aa68e`](https://github.com/nicolasguelfi/gensem/commit/67aa68e) |
| 06 | 📐 | Shared State section in design | v0.32.0 | [`a7aca0e`](https://github.com/nicolasguelfi/gensem/commit/a7aca0e) |
| 07 | 📐 | Intent Capture for greenfield projects | v0.28.0 | [`9846172`](https://github.com/nicolasguelfi/gensem/commit/9846172) |
| 08 | 📐 | Open Questions — activity-entry scan | v0.29.0 | [`d013d6d`](https://github.com/nicolasguelfi/gensem/commit/d013d6d) |
| 09 | 📐 | Scaffold-as-preview variant | v0.33.0 | [`37bf6ff`](https://github.com/nicolasguelfi/gensem/commit/37bf6ff) |
| 10 | 🎯 | Unified complexity-point semantics | v0.34.0 | [`910934d`](https://github.com/nicolasguelfi/gensem/commit/910934d) |
| 11 | 🎯 | Preview skip + anti-preview-ahead rule | v0.34.1 | [`a1ecc3e`](https://github.com/nicolasguelfi/gensem/commit/a1ecc3e) |
| 12 | 🔧 | Config Application Transparency | v0.30.0 | [`544766f`](https://github.com/nicolasguelfi/gensem/commit/544766f) |
| 13 | 📐 | Policy tests as first-class pyramid level | v0.35.0 | [`95e4ffd`](https://github.com/nicolasguelfi/gensem/commit/95e4ffd) |
| 14 | 🤝 | Methodology feedback via `/gse:compound` Axe 2 | v0.31.0 | [`fc85447`](https://github.com/nicolasguelfi/gensem/commit/fc85447) |
| 15 | 📐 | Tutor specialized agent (superseded) | v0.36.0 | [`661c247`](https://github.com/nicolasguelfi/gensem/commit/661c247) |
| 16 | 📐 | Unified coach agent — pedagogy + workflow (8 axes) | v0.37.0 | [`84a6684`](https://github.com/nicolasguelfi/gensem/commit/84a6684) |
| 17 | — | Absorbed by AMÉL-05 (no duplicate work) | — | — |
| 18 | 🎯 | Framework isolation check in `architect` agent | v0.37.2 | [`bacc968`](https://github.com/nicolasguelfi/gensem/commit/bacc968) |
| 19 | 🛡 | Connectivity preflight before external scaffolders | v0.37.3 | [`e6d373e`](https://github.com/nicolasguelfi/gensem/commit/e6d373e) |
| 20 | 🤝 | Upstream repo resolution + URL fix | v0.37.4 | [`78c8397`](https://github.com/nicolasguelfi/gensem/commit/78c8397) |

---

## 4. The 20 cards

### AMÉL-01 · Sprint Freeze — closed sprints are immutable

🛡 Guardrail · **v0.23.0** · Commit [`b6f76e4`](https://github.com/nicolasguelfi/gensem/commit/b6f76e4) · Layers: spec + design + implementation

**The problem in one sentence.** After a sprint was marked complete or abandoned, writing activities could still modify its backlog and task state, corrupting sprint-level metrics and release boundaries.

**What we did.** Introduced the **Sprint Freeze Invariant**. When `.gse/plan.yaml.status` is `completed` or `abandoned`, four writing activities (`/gse:task`, `/gse:produce`, `/gse:fix`, `/gse:review`) check at Step 0 and present a **Sprint Freeze Gate** before any mutation. Three options: *Start next sprint now*, *Cancel*, *Discuss*. No "amend closed sprint" option is offered — the sanctioned pattern is to open a successor sprint. Read-only activities (`/gse:status`, `/gse:health`, `/gse:compound`, etc.) are exempt.

**Before / After.**

```
Before: /gse:task                               After: /gse:task
         (sprint S03 already delivered)                  (sprint S03 already delivered)
         → adds TASK-47 silently to S03                  → Sprint Freeze Gate
         → sprint metrics now include                    → Start next sprint now? / Cancel / Discuss
           a task that wasn't in the release             → TASK-47 lands in S04 with clean traceability
```

**Go further:** spec §P11 (Sprint Freeze Invariant), design doc Sprint Freeze mechanics, `src/agents/gse-orchestrator.md` → "Sprint Freeze Invariant".

---

### AMÉL-02 · Automatic dashboard regeneration

🔧 Assistance mechanism · **v0.24.0** · Commit [`5d9a501`](https://github.com/nicolasguelfi/gensem/commit/5d9a501) · Layers: spec + design + implementation

**The problem in one sentence.** Only 6 of the 23 activities explicitly regenerated the dashboard; the other 17 left it stale, causing the visible sprint/phase/activity info to lag behind reality.

**What we did.** Added an **editor hook** (Edit / Write / MultiEdit) that invokes `dashboard.py --if-stale` after any file save. The `--if-stale` flag compares the dashboard's mtime against `.gse/**/*.yaml` and `docs/sprints/**/*.md`; it regenerates only when needed, with a 5-second debounce to prevent bursty regeneration during rapid edits. The 6 existing explicit regeneration calls are kept as belt-and-suspenders guarantees at major checkpoints.

**Before / After.**

```
Before:                                          After:
  /gse:produce runs                                /gse:produce runs
  dashboard unchanged                              dashboard regenerated within 5 s
  /gse:review runs                                 /gse:review runs
  dashboard still unchanged                        dashboard regenerated within 5 s
  → user watches stale state                       → dashboard always current
```

**Go further:** spec §7 dashboard policy, design doc "Dashboard Sync — Design Mechanics", `plugin/hooks/hooks.claude.json`.

---

### AMÉL-03 · Root-Cause Discipline before patching

🛡 Guardrail · **v0.26.0** · Commit [`0206978`](https://github.com/nicolasguelfi/gensem/commit/0206978) · Layers: spec + design + implementation

**The problem in one sentence.** Agents sometimes applied speculative patches in rapid succession without first identifying the actual root cause, a pattern often called *shotgun debugging* which introduces new bugs and erodes trust.

**What we did.** Added a mandatory 4-step protocol (spec P16 *Root-Cause Discipline*) executed before any fix touches a file:

1. **Read** the relevant source files in the current turn (no patch on unread code).
2. **Symptom** — restate the defect in precise, observable terms.
3. **Hypothesis + Evidence** — write the hypothesized root cause, a test that validates or invalidates it, and a confidence tag; run the test.
4. **Patch** — only after the evidence test confirms the hypothesis; the commit trailer carries `Root cause:` and `Evidence:`.

A counter `fix_attempts_on_current_symptom` in `.gse/status.yaml` increments when a patch fails to resolve the symptom. At the threshold (2 for beginner, 3 for intermediate, 4 for expert), the agent stops patching and spawns the `devil-advocate` sub-agent in focused-review mode. The loop is broken.

**Before / After.**

```
Before:                                          After:
  symptom reported                                 symptom reported
  → patch A (guess)                                → Read files / Symptom / Hypothesis / Test
  → symptom persists                               → Patch A lands with evidence in trailer
  → patch B (different guess)                      → symptom persists (counter 1)
  → symptom persists                               → new Hypothesis / Test cycle
  → patch C (yet another guess)                    → threshold reached
  → user frustrated, code muddled                  → devil-advocate invoked, root cause identified
```

**Go further:** spec §P16 "Root-Cause Discipline before patching", `src/activities/fix.md` Step 3, `src/agents/devil-advocate.md` focused-review mode.

---

### AMÉL-04 · Git Identity Verification at first commit

🛡 Guardrail · **v0.25.0** · Commit [`53d111d`](https://github.com/nicolasguelfi/gensem/commit/53d111d) · Layers: spec + design + implementation

**The problem in one sentence.** Activities that create commits programmatically failed or produced garbage authorship when no git identity was configured.

**What we did.** Any activity about to create a commit now performs a preflight: check that a git identity is set either globally (`git config --global user.name + user.email`) or locally. If neither is present, the **Git Identity Gate** appears with five options:

1. **Set global identity** (recommended default).
2. **Set local identity** (this project only).
3. **Quick placeholder** — sets `GSE User` / `user@local` plus a one-shot reminder to replace it before sharing.
4. **I'll set it myself** — the agent prints copy-paste commands and waits for confirmation.
5. **Discuss.**

Email format is validated (`@` + dotted domain); beginners see plain-language translations of each option.

**Before / After.**

```
Before:                                          After:
  /gse:hug Step 4 creates foundational commit      /gse:hug Step 4 before the commit
  git config empty                                 git config empty → Git Identity Gate
  commit fails or lands as "unknown <nobody>"      user picks option 1, enters email
  activity aborts halfway                          commit lands cleanly, activity continues
```

**Go further:** spec §P12 (Git Identity Invariant), design doc Git Identity materialization, `src/agents/gse-orchestrator.md` → "Git Identity Verification Invariant".

---

### AMÉL-05 · Scope Reconciliation + Inform-Tier Summary (absorbs AMÉL-17)

📐 New concept · **v0.27.0** · Commit [`67aa68e`](https://github.com/nicolasguelfi/gensem/commit/67aa68e) · Layers: spec + design + implementation

**The problem in one sentence.** Creator activities could drift off-scope silently (new files outside the planned TASK), and the Inform-tier decisions the agent made along the way were invisible after the fact.

**What we did.** Two complementary additions at the closure of every creator activity (`/gse:design`, `/gse:preview`, `/gse:produce`, `/gse:task`):

- **Scope reconciliation** — the activity records its starting HEAD SHA in `status.yaml → activity_start_sha` at entry, then computes `git diff --name-status <sha>..HEAD` at closure and surfaces any changes outside the scope of the current TASK.
- **Inform-tier Decisions Summary** — a short list of the autonomous choices the agent made during the activity (library micro-choice, folder naming, utility-vs-framework, defaults). The user gets a retrospective override window: *Accept all / Promote one to Gate / Discuss*.

AMÉL-17 (an Inform-tier observability concern) turned out to be fully covered by this summary — no separate implementation was needed.

**Before / After.**

```
Before:                                          After:
  /gse:produce completes                           /gse:produce completes
  new files silently added                         Scope reconciliation: "3 files outside TASK-07"
  agent picked a test runner silently              Inform-tier summary:
  → next sprint: "why is there a Jest config?"     " I used Vitest (v1 over v2), utility-first CSS,
                                                     PREVIEW comments in // style"
                                                   → user can promote any to a DEC- or accept
```

**Go further:** spec §P16 "Inform-Tier Decisions Summary", design doc "Scope Reconciliation — Design Mechanics", `src/activities/{design,preview,produce,task}.md` Step 7.

---

### AMÉL-06 · Shared State section in the design artefact

📐 New concept · **v0.32.0** · Commit [`a7aca0e`](https://github.com/nicolasguelfi/gensem/commit/a7aca0e) · Layers: spec + design + implementation

**The problem in one sentence.** Multi-component designs silently duplicated state — e.g., a "selected month" filter that should be one shared value was invented separately in three views, leading to inconsistencies no one noticed until integration.

**What we did.** Added `/gse:design` Step 2.5 — **Shared State Identification**. The design artefact now carries a mandatory `## Shared State` table with columns *Name · Scope (components) · Mechanism · Rationale · Traces*. Critically, the section is **mandatory even if empty**: when no shared state applies (CLI tool, pure library), the agent writes the explicit disclaimer *"No shared state identified — components are independent."* An empty section is indistinguishable from "we didn't think about it"; the disclaimer confirms the question was considered.

**Before / After.**

```
Before design.md:                                 After design.md:
  ## Component Decomposition                        ## Component Decomposition
  ## Interface Contracts                            ## Shared State
  ## Architecture Decisions                         | Name | Scope | Mechanism | Rationale |
                                                    |------|-------|-----------|-----------|
                                                    | selected_month | Dashboard, Expenses,
                                                      Budgets | Streamlit session_state |
                                                      Month filter consistent across views |
                                                    ## Interface Contracts
                                                    ## Architecture Decisions
```

**Go further:** spec §3.6 `/gse:design` description, `src/activities/design.md` Step 2.5.

---

### AMÉL-07 · Intent Capture for greenfield projects

📐 New concept · **v0.28.0** · Commit [`9846172`](https://github.com/nicolasguelfi/gensem/commit/9846172) · Layers: spec + design + implementation

**The problem in one sentence.** On an empty repository, the methodology jumped straight into `/gse:reqs` before the user had a chance to state what they actually wanted to build — requirements were being formalized without a captured intent to anchor to.

**What we did.** A new **Intent Capture** step runs inside `/gse:go` when the project is greenfield (no source files after standard exclusions) and no `docs/intent.md` exists. It produces a canonical `intent.md` artefact with YAML frontmatter and four sections: *Description (user's words, verbatim)*, *Reformulated understanding*, *Users*, *Boundaries*, *Open Questions*. A new artefact type `INT-NNN` joins the existing ID prefixes (REQ-, DES-, TST-, …). `/gse:collect` has a Step 0 that verifies intent exists before analysis.

**Before / After.**

```
Before (empty repo):                             After (empty repo):
  /gse:go                                          /gse:go
  → /gse:collect (nothing to collect)              → Intent Capture elicits:
  → /gse:assess                                       "I want a budget app for my household,
  → /gse:reqs (What should it do? From scratch)       only I use it, stays on my laptop"
                                                    → docs/intent.md written
                                                    → /gse:collect Step 0 confirms intent exists
                                                    → /gse:reqs anchors in docs/intent.md
```

**Go further:** spec §3 `/gse:go` Step 5 "Intent Capture", design doc "Intent Capture — Design Mechanics", `src/templates/intent.md`.

---

### AMÉL-08 · Open Questions — activity-entry scan

📐 New concept · **v0.29.0** · Commit [`d013d6d`](https://github.com/nicolasguelfi/gensem/commit/d013d6d) · Layers: spec + design + implementation

**The problem in one sentence.** Scope-shaping or architectural questions raised during `/gse:collect`, `/gse:assess` or Intent Capture had no formal consumption slot — they drifted forward as ad-hoc deviations until somebody remembered them.

**What we did.** Every artefact may carry an `## Open Questions` block with entries tagged `OQ-NNN`, each with `resolves_in` (which future activity should answer it) and `impact` (`scope-shaping | behavioral | architectural | cosmetic`). Four activities (`/gse:assess`, `/gse:plan`, `/gse:reqs`, `/gse:design`) run a **Step 0 scan** at entry: any open question with `resolves_in` matching the current activity is surfaced as an **Open Questions Gate**, calibrated by the user's `decision_involvement` profile (autonomous / collaborative / supervised). Resolutions update the origin artefact in place (`status: resolved`, `answer`, `answered_by`, `confidence`, `traces`) and, for high-impact questions, produce a `DEC-NNN` in the sprint decision journal.

**Before / After.**

```
Before:                                          After:
  /gse:assess surfaces 9 scope-shaping questions   /gse:assess writes OQ-001..009 with
  → treated as ad-hoc deviations                     resolves_in: /gse:plan and impact tags
  → re-asked chaotically during /gse:reqs          /gse:plan Step 0 scan picks them up
  → DEC-003 created retroactively to                 Per-question Gate with proposed answer
    normalize the situation                          + consequence horizons
                                                   → resolved OQs trace to DEC-NNN where needed
```

**Go further:** spec §P6 traceability extension, design doc "Open Questions — Design Mechanics", `src/activities/{assess,plan,reqs,design}.md` Step 0.

---

### AMÉL-09 · Scaffold-as-preview variant

📐 New concept · **v0.33.0** · Commit [`37bf6ff`](https://github.com/nicolasguelfi/gensem/commit/37bf6ff) · Layers: spec + design + implementation

**The problem in one sentence.** For web and mobile projects, an ASCII-wireframe preview was throwaway work that provided less value than a minimal runnable scaffold continuing into production.

**What we did.** `/gse:preview` now presents a variant Gate at Step 1.5: *(1) static description* (default, any domain) — wireframes, API examples, written into `preview.md`, throwaway — or *(2) scaffold-as-preview* (recommended for web/mobile) — a minimal runnable project (Vite, Next.js, Streamlit, React Native) that becomes the starting base for `/gse:produce`. Placeholder code is marked with language-idiomatic comments (`// PREVIEW:`, `# PREVIEW:`, `<!-- PREVIEW: -->`). Build validation is concrete: `npm run build` (or equivalent) exiting 0. The variant is per-activity, not per-project.

**Before / After.**

```
Before (web project):                            After (web project):
  /gse:preview writes ASCII wireframes             /gse:preview Gate → scaffold chosen
  in preview.md                                    npx create-vite → scaffold in ./
  /gse:produce starts from scratch                 PREVIEW: comments mark placeholders
  reimplements the UI                              /gse:produce fills in from the scaffold
                                                   (no throwaway code)
```

**Go further:** spec §3.6 `/gse:preview` description, design doc "Preview Variants", `src/activities/preview.md` Step 1.5.

---

### AMÉL-10 · Unified complexity-point semantics

🎯 Precision · **v0.34.0** · Commit [`910934d`](https://github.com/nicolasguelfi/gensem/commit/910934d) · Layers: spec + design + implementation

**The problem in one sentence.** Two sizing scales coexisted without a clear mapping — S/M/L task sizes and P10 complexity points (1–6) — so "an S-sized task" and "a 2-point task" were sometimes the same thing and sometimes not.

**What we did.** Merged them into a single **complexity-point** unit with a temporal anchor: **1 point ≈ one session-hour for the AI + user pair**. The coupling is intentional — a point represents combined effort and complexity, because in a paired AI+human flow these cannot be decoupled. A **Cost Assessment Grid** (design doc Appendix B) replaces the earlier blanket "zero-cost items" rule with a structured table: utility dependency (1pt), framework dependency (2–3pt), external service (2–4pt), UI component (1–2pt), security surface (2–3pt), data model (1–2pt), architectural change (3–5pt), new language/framework (4–6pt). Existing backlog data remained valid.

**Before / After.**

```
Before:                                          After:
  backlog.yaml:                                    backlog.yaml:
    TASK-07 { size: M, cost: 2 }                    TASK-07 { cost: 2 }
    TASK-08 { size: L }                             TASK-08 { cost: 4 }
  two signals, possibly contradictory               one signal, cost = 4 ≈ half a session-day
```

**Go further:** spec §P10 "Complexity Budget", design doc Appendix B "Cost Assessment Grid".

---

### AMÉL-11 · Preview skip + anti-preview-ahead rule

🎯 Precision · **v0.34.1** · Commit [`a1ecc3e`](https://github.com/nicolasguelfi/gensem/commit/a1ecc3e) · Layers: spec + design + implementation

**The problem in one sentence.** `/gse:preview` triggered even when the current sprint had nothing previewable (a foundation sprint doing infrastructure only), and the temptation existed to preview future-sprint tasks from the current sprint.

**What we did.** Two sharpening rules:

- **Sprint-level skip** — `/gse:preview` is legitimately skipped when the current sprint contains no user-visible or demonstrable task (pure infrastructure sprints). The skip is recorded in `plan.yaml → workflow.skipped` with an explicit reason; this is not a deviation.
- **Preview-is-just-in-time** — preview-ahead (previewing a future-sprint task from the current sprint) is explicitly not supported. The spec and activity both carry the rule: preview the tasks of the current sprint, nothing else.

**Before / After.**

```
Before:                                          After:
  sprint S02 is all infrastructure                 sprint S02 is all infrastructure
  /gse:preview produces empty preview.md           /gse:preview recognizes no previewable tasks
                                                   → legitimate skip recorded in plan.yaml
  tempting: preview a future-sprint UI task        → no deviation flag
  "to get ahead"
                                                   preview-ahead request → rejected with rationale
                                                   (preview the task when its sprint arrives)
```

**Go further:** spec §3.6 `/gse:preview` description (Sprint-level skip paragraph), `src/activities/preview.md` skip condition.

---

### AMÉL-12 · Config Application Transparency

🔧 Assistance mechanism · **v0.30.0** · Commit [`544766f`](https://github.com/nicolasguelfi/gensem/commit/544766f) · Layers: spec + design + implementation

**The problem in one sentence.** Methodology-default config values (chosen during `/gse:hug`, never explicitly validated) materialized silently — when `/gse:produce` first created a worktree directory, the user was surprised to see it and had to figure out where it came from.

**What we did.** An **Inform-tier discipline**: whenever an activity materializes a `config.yaml` field with user-visible consequences (creates files, changes git state, enforces a hard threshold, alters delivery behavior), it prints one line at the moment of materialization:

> *Config applied: `<field>` = `<value>` (`<origin>` — to change: `/gse:hug --update` or edit `.gse/config.yaml`)*

The `<origin>` is computed at display time: if the value equals the methodology default, origin is *methodology default*; otherwise *user choice*. For beginners, the technical field name is translated per P9. No Gate, no interruption. Covered in v0.30: `git.strategy` materialized by `/gse:produce` Step 2 and `/gse:task` Step 4. The pattern extends to future fields by adding the Inform line to the relevant step.

**Before / After.**

```
Before:                                          After:
  /gse:produce                                     /gse:produce
  → creates .gse-worktrees/ directory              → Config applied: git.strategy = worktree-isolated
  → user: "where did this come from?"                 (methodology default — to change: /gse:hug
                                                      --update or edit .gse/config.yaml)
                                                   → creates .gse-worktrees/
```

**Go further:** spec §P7 Inform-tier clarification, `src/agents/gse-orchestrator.md` "Config Application Transparency Discipline", `src/activities/produce.md` Step 2.

---

### AMÉL-13 · Policy tests as a first-class pyramid level

📐 New concept · **v0.35.0** · Commit [`95e4ffd`](https://github.com/nicolasguelfi/gensem/commit/95e4ffd) · Layers: spec + design + implementation

**The problem in one sentence.** Architecture-enforcement tests (e.g., *"the domain layer must not import the UI layer"*) didn't fit the unit/integration/E2E pyramid and were awkwardly lumped into "Other" alongside accessibility and performance.

**What we did.** Added **Policy** as a first-class column in the test pyramid (spec §6.1), sitting between *Acceptance* and *Other*, with a 5% baseline (raisable to 10–15% for strict-architecture projects). Policy tests enforce **structural rules** via static scans — architecture layering, license compliance, naming conventions, file-size limits, docstring requirements. They run fast (no runtime) and guard the codebase's *shape*, not its *behavior*. Per-language tooling suggestions are documented: `pytest-archon`/`grimp` (Python), `ts-arch`/`dependency-cruiser` (TypeScript/JavaScript), `ArchUnit` (Java), `go-arch-lint` (Go), `cargo-deny` (Rust). `/gse:tests --strategy` now scans `design.md` and `decisions.md` to propose policy tests automatically.

**Before / After.**

```
Before test pyramid:                             After test pyramid:
  | Unit | Integration | E2E | Acc | Other |      | Unit | Integration | E2E | Acc | Policy | Other |
  layering rules: in docs, not enforced            layering rules: enforced as tests, fail CI
```

**Go further:** spec §6.1 pyramid table, design doc "Policy Tests — Design Mechanics", `src/activities/tests.md` Step 1 policy scan.

---

### AMÉL-14 · Methodology feedback via `/gse:compound` Axe 2

🤝 Feedback loop · **v0.31.0** · Commit [`fc85447`](https://github.com/nicolasguelfi/gensem/commit/fc85447) · Layers: spec + design + implementation

**The problem in one sentence.** Observations about the methodology itself (things that worked poorly, things that were missing) were noted during a sprint but had no consolidation channel — the signal was lost by the next sprint.

**What we did.** `/gse:compound` Axe 2 now closes with a **3-option Gate**:

1. **Export as a local feedback document only** — produces a sprint-scoped `methodology-feedback.md` artefact the user can share through any channel (email, chat, manual issue).
2. **Propose GitHub tickets** — curated, theme-grouped, deduplicated against existing issues, capped (`compound.max_proposed_issues_per_sprint`, default 3), each validated individually via a per-ticket Gate before submission through `/gse:integrate` Axe 2.
3. **Both** — export AND propose tickets.

Quality rules apply to the ticket path: every proposal must cite at least one concrete example, be theme-grouped (one ticket per theme, never per micro-friction), be deduplicated against open issues (when `gh` is available), respect the per-sprint cap, and pass a per-ticket Gate. If no observations were gathered (rare), the Gate is silently skipped.

**Before / After.**

```
Before:                                          After:
  sprint ends                                      sprint ends / /gse:compound Axe 2
  "we should report that the X activity            → observations consolidated by theme
   gave us trouble" — lost                         → 3-option Gate
                                                   → either local export or curated tickets,
                                                     capped and deduplicated, user-validated
```

**Go further:** spec §3 `/gse:compound` Axe 2, design doc "Methodology Feedback — Design Mechanics", `src/activities/compound.md` Step 2.3+.

---

### AMÉL-15 · Tutor specialized agent (superseded)

📐 New concept · **v0.36.0** · Commit [`661c247`](https://github.com/nicolasguelfi/gensem/commit/661c247) · *Superseded by v0.37.0 — see AMÉL-16*

**The problem in one sentence.** Knowledge-transfer logic (P14) was inlined in the orchestrator, mixing pedagogical reasoning with project execution and producing ad-hoc pedagogical preambles without a clean extension point.

**What we did (briefly).** Introduced a dedicated `tutor` sub-agent activated at the beginning of any activity when `learning_goals` is non-empty. It returned either *skip* (silent) or *propose* with a precise topic formulation and a 5-option P14 preamble (Quick overview / Deep session / Not now / Not interested / Discuss). It also handled inferred gap detection (repeated questions, hesitations, explicit confusion). Persistence lived in `status.yaml → learning_preambles[]` and `detected_gaps[]`. An extensible "Pedagogical recipes" section was the extension point.

This was a clean design, but field analysis showed it overlapped with the intended workflow-monitoring responsibilities (velocity, health, engagement). Rather than ship two agents reading the same signals, v0.37.0 merged both concerns into a single `coach` agent — see AMÉL-16. The v0.36 tutor was superseded without migration work (no existing deployment, no data loss).

**Go further:** superseded — read AMÉL-16 next.

---

### AMÉL-16 · Unified coach agent — pedagogy + workflow monitoring (8 axes)

📐 New concept · **v0.37.0** · Commit [`84a6684`](https://github.com/nicolasguelfi/gensem/commit/84a6684) · Layers: spec + design + implementation

**The problem in one sentence.** If pedagogy (learning goals, gap detection) and workflow monitoring (velocity, health, engagement, sustainability) were implemented as two separate agents, they would read from largely the same signal sources (profile, status history, P16 counters, activity transitions) — doubling the work and splitting extensibility across two files.

**What we did.** Merged both concerns into a single `coach` sub-agent with **8 axes**:

| # | Axis | Category | What it watches |
|---|---|---|---|
| 1 | Pedagogy | Pedagogy | Explicit learning goals + inferred gaps → 5-option P14 preambles, LRN- notes |
| 2 | Profile calibration | Workflow | Drift between declared HUG profile and observed behavior |
| 3 | Sprint velocity | Workflow | Pace vs complexity budget, stall detection |
| 4 | Workflow health | Workflow | Broken flow sequences, uncompleted transitions |
| 5 | Quality trends | Workflow | Test pass-rate, review findings, design debt trends |
| 6 | Engagement pattern | Workflow | P16 acceptance streaks / pushback-dismissed signals |
| 7 | Process deviation | Workflow | Sanctioned activities bypassed (e.g., produce without preview) |
| 8 | Sustainability | Workflow | Session length / cadence drift from profile |

Each axis is toggleable individually via `config.yaml → coach.axes.<axis>`. A single invocation contract, a single extensibility surface (**Coaching recipes**, tagged per axis, dual-maintenance: user-editable + agent-auto-updatable via `/gse:compound` Axe 3). Persistence: `learning_preambles[]`, `detected_gaps[]`, `profile_drift_signals{}`, `workflow_observations[]` in `status.yaml`.

**Before / After.**

```
v0.36:                                           v0.37:
  tutor agent (pedagogy only)                      coach agent (pedagogy + workflow)
  workflow monitoring: inline in orchestrator      8 axes, per-axis toggles
  two implementations, overlapping signals         single agent, single signal read
```

**Go further:** spec §P14 "Knowledge Transfer (Coaching)", design doc "Coach agent — Design Mechanics", `src/agents/coach.md`.

---

### AMÉL-17 · Absorbed by AMÉL-05

Originally planned as an Inform-tier observability concern distinct from scope reconciliation. Analysis showed complete overlap with AMÉL-05 *Inform-Tier Decisions Summary* — the retrospective override window already surfaces every silent autonomous decision. A separate implementation would have duplicated the mechanism with no additional value. Closed out as part of v0.27.0 ([`67aa68e`](https://github.com/nicolasguelfi/gensem/commit/67aa68e)).

---

### AMÉL-18 · Framework isolation check in the `architect` agent

🎯 Precision · **v0.37.2** · Commit [`bacc968`](https://github.com/nicolasguelfi/gensem/commit/bacc968) · Layers: implementation

**The problem in one sentence.** The framework-free domain module pattern (keeping `src/domain/**` free of heavy UI/I/O framework imports, so business logic stays testable and replaceable) emerged spontaneously in testing but was never proposed systematically.

**What we did.** Enriched the existing `architect` sub-agent:

- **New priority** — "Framework isolation — when a heavy UI or I/O framework is present, business logic is kept framework-free so it stays testable and replaceable".
- **New checklist item** — conditional on the design containing a heavy UI/I/O framework (Streamlit, React, Next.js, Django, Flask, FastAPI, Express, Spring, …) AND non-trivial business logic. On match, the agent proposes a framework-free domain module, records a DEC, and flags a policy test (`src/domain/** must not import <framework>`). Skipped for CLI / library / scientific / embedded projects.
- **New output example** — `DES-004 [INFO] — Framework isolation opportunity`.
- One-line guideline in `/gse:design` Step 2 for workflow visibility.

**Rejected alternative: a new principle P17.** The pattern is a corollary of existing principles (*Dependency direction*, *Separation of concerns*, *Layering violations*) rather than a transversal invariant, and it is conditional on project type — elevating it to a P-level rule would break the universality of P1–P16. Enriching the `architect` agent placed the recommendation exactly where it applies.

**Before / After.**

```
Before:                                          After:
  design.md emerges with logic inlined             /gse:design / /gse:review
  in page modules alongside Streamlit              architect evaluates checklist
  → only surfaced if the model was rigorous        → DES-004 [INFO] — Framework isolation
                                                      opportunity: propose src/domain/ module +
                                                      DEC + policy test enforcing import boundary
```

**Go further:** `src/agents/architect.md` (checklist), `src/activities/design.md` Step 2.

---

### AMÉL-19 · Connectivity preflight before external scaffolders

🛡 Guardrail · **v0.37.3** · Commit [`e6d373e`](https://github.com/nicolasguelfi/gensem/commit/e6d373e) · Layers: design + implementation

**The problem in one sentence.** When an external scaffolder command (`create-next-app`, `create-vite`, `streamlit init`) failed because of network / proxy / sandbox restrictions, agents retried blindly — up to 3 identical attempts — instead of presenting a structured choice to the user.

**What we did.** A **connectivity preflight** was added to `/gse:preview` scaffold-as-preview variant. Before invoking any scaffolder, the agent issues a **short, ecosystem-appropriate reachability probe** to confirm the target package registry is reachable from the current environment. The exact probe command is left to the coding agent's judgment based on the detected ecosystem (Node, Python, Rust, …) — the methodology specifies *what to verify and when*, not *how*.

On probe failure, the agent does **not** retry the scaffold command. It presents a **4-option Gate**:

1. **Retry** — the user may have adjusted the proxy or sandbox permission.
2. **Run locally, then resume** — the agent prints the exact scaffold command, the user runs it in their own terminal, confirms completion, the agent resumes from the created directory.
3. **Fallback to static preview** — switches `preview_variant` back to `static` for this sprint.
4. **Discuss.**

Scope intentionally limited to `/gse:preview` for now; extension to `/gse:produce` or `/gse:tests` installs is deferred until further signals emerge.

**Before / After.**

```
Before:                                          After:
  npx create-next-app                              probe → registry unreachable
  → fail: EAI_AGAIN registry.npmjs.org             → Gate: Retry / Run locally / Fallback / Discuss
  → retry (same command)                           → user picks "Run locally"
  → fail                                           → agent prints the exact command
  → retry                                          → user runs it in terminal, confirms
  → fail — ~5 minutes of blind retries             → agent resumes from scaffold
```

**Go further:** `src/activities/preview.md` Step 1.5 option 2, design doc "Preview Variants" fail-modes.

---

### AMÉL-20 · Upstream repository resolution + URL fix

🤝 Feedback loop · **v0.37.4** · Commit [`78c8397`](https://github.com/nicolasguelfi/gensem/commit/78c8397) · Layers: design + implementation

**The problem in one sentence.** `/gse:integrate` Axe 2 (submitting methodology feedback as GitHub issues) couldn't reach upstream because (a) the hardcoded manifest URL pointed to a non-existent repository, (b) the opencode manifest was missing the `repository` field entirely, and (c) there was no way for the user to redirect feedback to a private fork or corporate tracker.

**What we did.** Three coordinated fixes:

- **Canonical URL corrected** — introduced a `UPSTREAM_REPO` constant in the generator, single source of truth, propagated to all three platform manifests (Claude, Cursor, opencode).
- **opencode parity** — `opencode.json` now carries `gse.repository` alongside `gse.version`.
- **User-level override** — new `github.upstream_repo` field in `config.yaml`. When set, it takes precedence over the plugin manifest. Supports private forks, corporate issue trackers, training-environment redirections without editing the shipped plugin.
- **Privacy Gate strengthened** — the final submission Gate now states explicitly that GitHub issues are public.

The **resolution order** is documented consistently across `orchestrator.md`, `integrate.md`, `compound.md` and the design doc: (1) `config.yaml → github.upstream_repo` if set, (2) plugin manifest, (3) skip Axe 2 with an Inform note.

**Rejected alternative: a new `upstream.issues_url` field.** The initial analysis proposed adding a new manifest field, but closer reading showed `repository` already existed and was referenced by the methodology — the real problems were a hardcoded wrong URL, opencode missing the field, and no user override. A new field would have duplicated existing infrastructure.

**Before / After.**

```
Before:                                          After:
  /gse:integrate Axe 2                             /gse:integrate Axe 2
  → gh issue create --repo gse-one/gse-one         → resolve: config.upstream_repo → manifest
  → fails silently (dead URL)                      → gh issue create --repo <resolved>
  opencode: Axe 2 disabled                         opencode: Axe 2 works (gse.repository)
  no way to redirect to internal tracker           github.upstream_repo in config.yaml redirects
```

**Go further:** `gse-one/gse_generate.py` (UPSTREAM_REPO constant), `src/templates/config.yaml` section 10, `src/activities/integrate.md` Step 2.

---

## 5. Deferred or reshaped items

Honesty matters. Two items in this report did not ship in the form first proposed.

**AMÉL-17 — absorbed.** Initial categorization surfaced Inform-tier observability as a concern of its own. Deeper analysis showed full overlap with the Inform-Tier Decisions Summary added by AMÉL-05 (v0.27.0). Implementing a second mechanism would have split visibility across two channels without adding clarity. Closed out as part of AMÉL-05, no separate release.

**AMÉL-20 — rejected first design.** The initial analysis proposed a new `upstream.issues_url` manifest field. A critical relecture before implementation found that the `repository` field already existed in plugin manifests and was referenced by the methodology; a new field would have duplicated it with weaker semantics. The actual defects were (a) a hardcoded wrong URL, (b) opencode missing the field, (c) no user-level override. The shipped version fixes these three defects and introduces exactly one new field (`github.upstream_repo` in `config.yaml`) serving the specific purpose that was not already covered.

**Ancillary fix — generator `--clean` safety (v0.37.1, commit [`9198e4b`](https://github.com/nicolasguelfi/gensem/commit/9198e4b)).** Discovered during AMÉL-16 implementation. The generator's `--clean` flag wiped the hand-maintained `plugin/tools/` directory along with the regenerated content. Corrected: `--clean` now preserves `plugin/tools/`, and `verify()` hard-fails when `plugin/tools/dashboard.py` is missing (defense-in-depth).

---

## 6. What was intentionally not retained

Not every observation from the test sessions became an improvement. Some were deliberately left out because they fall outside the methodology's scope:

- **Individual preferences** — framework choices (e.g., Streamlit vs React, Tailwind vs CSS modules) are user/model decisions, not plugin defects.
- **Model-specific artifacts** — bugs tied to a particular JS/Python ecosystem version (e.g., `@types/node` resolution under Vitest 4.x) target the external ecosystem, not the plugin.
- **Perceived duration** — "is it normal that this takes so long?" is a perception that is shaped by communication cadence (already covered by P9), not a methodological defect.
- **Inter-model variance** — GPT vs Opus behavior differences cannot be corrected by the plugin directly. The integrity guardrails added by AMÉLs 03, 05 and (absorbed) 17 reduce this variance by forcing discipline at explicit checkpoints.

Keeping these out was a conscious choice aligned with the plugin's mandate: codify what *must* happen reproducibly, leave what *may* happen to the model and user.

---

## Annex A — Release chronology

All dates are April 2026.

| Release | Date | Main content |
|---|---|---|
| v0.23.0 | 19 Apr | AMÉL-01 Sprint Freeze |
| v0.24.0 | 19 Apr | AMÉL-02 Dashboard auto-regen |
| v0.25.0 | 19 Apr | AMÉL-04 Git Identity |
| v0.26.0 | 19 Apr | AMÉL-03 Root-Cause Discipline |
| v0.27.0 | 19 Apr | AMÉL-05 Scope Reconciliation + Inform-Tier (absorbs AMÉL-17) |
| v0.28.0 | 19 Apr | AMÉL-07 Intent Capture |
| v0.29.0 | 19 Apr | AMÉL-08 Open Questions |
| v0.30.0 | 19 Apr | AMÉL-12 Config Application Transparency |
| v0.31.0 | 19 Apr | AMÉL-14 Methodology feedback |
| v0.32.0 | 19 Apr | AMÉL-06 Shared State |
| v0.33.0 | 19 Apr | AMÉL-09 Scaffold-as-preview |
| v0.34.0 | 19 Apr | AMÉL-10 Unified complexity-points |
| v0.34.1 | 19 Apr | AMÉL-11 Preview skip + anti-preview-ahead |
| v0.35.0 | 19 Apr | AMÉL-13 Policy tests |
| v0.36.0 | 19 Apr | AMÉL-15 Tutor agent (superseded) |
| v0.37.0 | 20 Apr | AMÉL-16 Unified coach agent |
| v0.37.1 | 20 Apr | Ancillary fix — generator `--clean` safety |
| v0.37.2 | 20 Apr | AMÉL-18 Framework isolation |
| v0.37.3 | 20 Apr | AMÉL-19 Connectivity preflight |
| v0.37.4 | 20 Apr | AMÉL-20 Upstream repo resolution |
| v0.38.0 | 20 Apr | Milestone — cycle closure |

---

## Annex B — Glossary

A minimal set of terms used in the cards.

- **Activity** — one of the 23 commands under the `/gse:` prefix (e.g., `/gse:design`, `/gse:produce`). Each activity has explicit steps and produces a defined artefact.
- **Agent (specialized)** — a named sub-agent with a focused mandate (architect, security-auditor, coach, devil-advocate, …). Invoked by the orchestrator at specific moments.
- **Artefact** — any project deliverable under GSE-One's control: code, requirements (REQ-), design (DES-), tests (TST-), decisions (DEC-), learning notes (LRN-), intent (INT-).
- **Complexity point** — the single sizing unit (since v0.34.0). 1 point ≈ one session-hour for the AI + user pair. Costs are assigned per the Cost Assessment Grid.
- **DEC-NNN** — a decision record. Every Gate-tier decision produces one in the sprint decision journal.
- **Gate** — a human-in-the-loop decision point. Format: question + context + options with consequence horizons + a mandatory *Discuss* option.
- **Guardrail (Soft / Hard / Emergency)** — protection against risky actions. Soft = warn, Hard = block + explain, Emergency = halt until explicit risk acknowledgment.
- **HUG** — the user profile capture activity (`/gse:hug`). Stores `profile.yaml` with expertise level, decision involvement, language, learning goals, etc.
- **Inform-tier** — decisions the agent made autonomously because they were individually low-stakes; surfaced retrospectively at activity closure so the user can override.
- **Open Question (OQ-NNN)** — a question that cannot be resolved now; attached to an artefact with `resolves_in` (which future activity will answer) and `impact`.
- **Policy test** — a test that enforces a structural rule on the codebase (architecture layering, license compliance, naming) via static scan. Fast, deterministic, fails CI.
- **Sprint** — a complexity-budgeted iteration. Ends when the budget is consumed or all tasks are delivered. Never by calendar.
- **Worktree** — an isolated working directory per task; the user and agent can work on multiple tasks in parallel without interference.

---

## Annex C — Getting the current version

```
git clone https://github.com/nicolasguelfi/gensem.git
cd gensem
python3 install.py              # Claude Code + Cursor + opencode
```

The installer writes `~/.gse-one` (path to the plugin directory) so tools can resolve at runtime. Uninstall via `python3 install.py --uninstall`.

Current version: **v0.38.0** (see [`VERSION`](https://github.com/nicolasguelfi/gensem/blob/main/VERSION)).

---

*End of report.*
