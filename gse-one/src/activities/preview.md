---
description: "Simulate planned artefacts before production. Triggered by /gse:preview."
---

# GSE-One Preview — Preview

Arguments: $ARGUMENTS

## Options

| Flag / Sub-command | Description |
|--------------------|-------------|
| (no args)          | Preview all planned artefacts for the current sprint |
| `<task-id>`        | Preview a specific task (e.g., `TASK-010`) |
| `--type <type>`    | Preview only artefacts of a given type (ui, api, arch, data, feature, import) |
| `--compare`        | Side-by-side comparison for imported elements |
| `--help`           | Show this command's usage summary |

## Prerequisites

Before executing, read:
1. `.gse/status.yaml` — current sprint number
2. `.gse/backlog.yaml` — sprint tasks to preview
3. `docs/sprints/sprint-{NN}/reqs.md` — requirements (for validation context)
4. `docs/sprints/sprint-{NN}/design.md` — design (for structural context)
5. `.gse/sources.yaml` — external sources (for import previews)
6. `.gse/profile.yaml` — user profile (apply P9 Adaptive Communication to all output)

## Workflow

### Step 1 — Identify Preview Targets

For each planned task in the current sprint, determine the appropriate preview type:

| Task Domain | Preview Type | Method |
|-------------|-------------|--------|
| Web frontend, UI component | **UI preview** | Wireframe description |
| REST/GraphQL endpoint | **API preview** | Request/response examples |
| System structure, modules | **Architecture preview** | Component diagram |
| Database, schema, models | **Data model preview** | Entity list with relationships |
| User-facing feature | **Feature preview** | User story walkthrough |
| External source import | **Import preview** | Side-by-side original vs planned |

### Step 2 — Generate Previews

#### UI Preview (Wireframe Description)

For tasks involving user interface elements:

```
Preview: TASK-010 — Admin Dashboard (UI)
────────────────────────────────────────

  ┌─────────────────────────────────────────┐
  │  Header: "Admin Dashboard"    [Logout]  │
  ├────────┬────────────────────────────────┤
  │        │                                │
  │  Nav   │  Main Content Area             │
  │  ----  │  ┌──────────┐ ┌──────────┐    │
  │  Users │  │ Card:    │ │ Card:    │    │
  │  Roles │  │ Total    │ │ Active   │    │
  │  Logs  │  │ Users    │ │ Sessions │    │
  │  Settings│ └──────────┘ └──────────┘    │
  │        │                                │
  │        │  ┌────────────────────────┐    │
  │        │  │ Table: Recent Activity │    │
  │        │  │ User | Action | Time   │    │
  │        │  │ ...  | ...    | ...    │    │
  │        │  └────────────────────────┘    │
  └────────┴────────────────────────────────┘

  Interactions:
  - Nav items highlight on selection, load corresponding view
  - Cards show real-time counts, update every 30s
  - Table is sortable by column, paginated (20 rows)
  - Logout triggers confirmation dialog

  Responsive: sidebar collapses to hamburger menu below 768px
```

#### API Preview (Request/Response Examples)

For tasks involving API endpoints:

```
Preview: TASK-013 — Webhook Handler (API)
─────────────────────────────────────────

  POST /api/v1/webhooks
  Headers:
    Content-Type: application/json
    X-Webhook-Secret: sha256=abc123...

  Request Body:
    {
      "event": "payment.completed",
      "data": {
        "order_id": "ORD-456",
        "amount": 99.99,
        "currency": "EUR"
      },
      "timestamp": "2026-01-20T14:30:00Z"
    }

  Response (200 OK):
    {
      "status": "accepted",
      "webhook_id": "WH-789",
      "processed_at": "2026-01-20T14:30:01Z"
    }

  Response (401 Unauthorized):
    {
      "error": "invalid_signature",
      "message": "Webhook signature verification failed"
    }

  Response (429 Too Many Requests):
    {
      "error": "rate_limited",
      "retry_after": 60
    }
```

#### Architecture Preview (Component Diagram)

For tasks involving system structure:

```
Preview: TASK-017 — Plugin Architecture (Architecture)
──────────────────────────────────────────────────────

  ┌─────────────┐     ┌──────────────┐
  │  Core App    │────>│ PluginLoader │
  └──────┬──────┘     └──────┬───────┘
         │                    │
         │           ┌───────┴────────┐
         │           │ PluginRegistry │
         │           └───────┬────────┘
         │                   │
    ┌────┴───┐    ┌──────────┼──────────┐
    │ Config │    │          │          │
    └────────┘  ┌─┴──┐  ┌───┴──┐  ┌───┴──┐
                │ P1  │  │  P2  │  │  P3  │
                └─────┘  └──────┘  └──────┘
                (plugins implement PluginInterface)

  Plugin Interface:
    - initialize(config) -> Result
    - execute(context) -> Result
    - shutdown() -> void

  Extension points: event hooks, middleware chain, CLI subcommands
```

#### Data Model Preview (Entity List)

For tasks involving data structures:

```
Preview: TASK-011 — Notification System (Data Model)
────────────────────────────────────────────────────

  Notification
    id:          UUID (PK)
    user_id:     UUID (FK -> User)
    type:        enum (info, warning, error, success)
    title:       string (max 200)
    body:        text
    read:        boolean (default false)
    created_at:  timestamp
    expires_at:  timestamp (nullable)

  NotificationPreference
    user_id:     UUID (FK -> User, PK)
    channel:     enum (email, push, in-app)
    enabled:     boolean
    quiet_hours: jsonb {start: "22:00", end: "08:00"}

  Relationships:
    User 1──* Notification
    User 1──* NotificationPreference
```

#### Feature Preview (User Story Walkthrough)

For user-facing features:

```
Preview: TASK-016 — CSV Export (Feature Walkthrough)
────────────────────────────────────────────────────

  Step 1: User navigates to the data table view
  Step 2: User clicks "Export" button in the toolbar
  Step 3: Modal appears with options:
          - Format: CSV (default) / JSON / Excel
          - Scope: Current page / All pages / Selected rows
          - Columns: All (default) / Custom selection
  Step 4: User clicks "Download"
  Step 5: Browser downloads file named "{table}-{date}.csv"
  Step 6: Toast notification: "Export complete — 1,234 rows exported"

  Edge cases:
  - Empty table: button disabled, tooltip "No data to export"
  - Large dataset (>100k rows): progress bar, async generation
  - Export in progress: button shows spinner, disabled
```

#### Import Preview (Side-by-Side Comparison)

For tasks that use external sources:

```
Preview: TASK-018 — Import JWT Module from SRC-001 (Import)
───────────────────────────────────────────────────────────

  Original (SRC-001: auth/jwt.py)          Planned (src/auth/jwt.py)
  ─────────────────────────────            ────────────────────────────
  class JWTHandler:                        class JWTHandler:
    def __init__(self, secret):              def __init__(self, config: AuthConfig):
      self.secret = secret                     self.secret = config.jwt_secret
                                               self.algorithm = config.jwt_algorithm
    def create_token(self, payload):         def create_token(self, payload):
      return jwt.encode(...)                   return jwt.encode(...)
    def verify_token(self, token):           def verify_token(self, token):
      return jwt.decode(...)                   return jwt.decode(...)
                                             def refresh_token(self, token):  # NEW
                                               ...

  Changes:
  - Config injection instead of raw secret (aligns with DES-004)
  - Added algorithm configuration
  - Added refresh_token method (REQ-003)
  - Original tests (SRC-003) can be adapted with minimal changes
```

### Step 3 — Collect Feedback

After presenting previews, ask for feedback (Gate):

- **Approve** — Preview matches expectations, proceed to production
- **Adjust** — Modify specific aspects (describe changes)
- **Reject** — Rethink the approach, return to design
- **Discuss** — Explore alternatives

Note: Previews are lightweight simulations, not prototypes. They are disposable and exist only to validate direction before investing in production.

### Step 4 — Inform-Tier Decisions Summary (Creator-Activity Closure, spec P16)

Close the activity with a retrospective list of the **Inform-tier decisions** the agent made during preview work (per P7 risk classification). PREVIEW may involve many small autonomous choices — mockup layout conventions, placeholder data choices, interaction stub styles — that were individually low-stakes.

1. **Assemble the list** from the agent's conversation memory for this activity. Examples: *"used mock data with 3 sample expenses"*, *"mocked the dashboard chart with a static SVG rather than a JS library stub"*, *"labeled the placeholder button 'Save' rather than 'Submit'"*.

2. **If the list is empty** (rare — all choices were Gated), display explicitly: *"No inform-tier decisions made this activity — all choices were Gated."* Then conclude PREVIEW.

3. **If the list is non-empty, present it and the Gate:**

   ```
   **Inform-tier decisions made during this preview:**
   - {decision 1}
   - {decision 2}
   - ...

   Any of these you want to promote to a Gate decision?

   **Options:**
   1. **Accept all as-is** (default) — Record as an `## Inform-tier Decisions` section at the end of `docs/sprints/sprint-{NN}/preview.md`.
   2. **Promote one or more to Gate** — For each selected decision, walk through a standard Gate. The resulting DEC-NNN is added to `decisions.md`.
   3. **Discuss** — Explore any of the decisions before accepting or promoting.
   ```

4. Execute the chosen option. Accepted decisions are serialized as a markdown list.
