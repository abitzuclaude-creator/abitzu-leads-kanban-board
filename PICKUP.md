# PICKING UP: Abitzu Lead Pipeline (READ FULLY BEFORE ACTING)

You are continuing work on the **Abitzu Lead Pipeline** — a live, deployed CRM
Kanban board for Abitzu Salon & Spa Software. This is a **REAL, WORKING app with
REAL customer data, published on the web.** It is NOT a prototype and NOT mock
data. Do not re-scaffold it, and do not "decide the stack" — those decisions are
already made. Your job is to understand the current live version, then enhance it.

> The person you're working with may not be technical. Explain things in plain
> language and translate their intent into the actual work yourself.

## What it is (plain terms)
A single-screen sales tool where the team logs in and tracks salon/spa leads
across 12 pipeline stages (New Lead → Contacted → Demo Requested → Demo Scheduled
→ Demo Completed → Follow Up → Proposal Sent → Negotiation → Converted → Lost →
Called·No Answer → Junk). Drag-and-drop cards, filters, a lead detail drawer with
a notes/history timeline, a New Lead form, an 8-card KPI strip, and an Analytics
view (funnel, conversion donut, lead sources, monthly closures, salesperson
performance, expected revenue). Changes sync live across all signed-in teammates.

## Where everything lives
- **CODE (GitHub):** repo `abitzuclaude-creator/abitzu-leads-kanban-board`.
  Work on branch `claude/abitzu-lead-pipeline-ok2uvn`. The whole app is ONE file:
  `index.html`.
- **HOSTING (Vercel):** Vercel serves `index.html` directly. Whatever is committed
  and pushed is what goes live. There is no separate build/source repo.
- **DATABASE (Supabase):** project "Abitzu Lead Lifecycle Management Kanban v1",
  ref `gkxgnxskrkonobcbtmjs`, region ap-northeast-1, Postgres 17. The front-end
  connects with a publishable (anon) key embedded in `index.html`; Row Level
  Security protects the data, and login is Supabase email/password.

## How the file is structured (important — don't get stuck here)
`index.html` is a self-contained bundle, not plain source:
- A gzipped/base64 "manifest" holds fonts + a small "dc-runtime" framework.
- The actual app (HTML template + a `class Component`) lives inside a single
  large JSON string in a `<script type="__bundler/template">` tag.

So the **editable source of truth is that template string.** To change the app you
must: decode that string → edit the template/component → re-pack it back into
`index.html`. (Use a small script to JSON-decode and re-encode it; don't hand-edit
the escaped blob.) The `class Component` holds all logic: Supabase load, realtime
sync, the 12-stage board, drawer, KPIs, and analytics. A leftover `seedLeads()`
function exists but is **DEAD CODE** — the app runs entirely on Supabase.

## Orientation checklist — do ALL of this before proposing changes
1. Read `index.html` and decode the template string; read the full `Component`
   class and template so you know how the board, drawer, KPIs, and analytics work.
2. Verify the Supabase connection via the Supabase MCP tools: `list_projects` →
   confirm ref `gkxgnxskrkonobcbtmjs`; `list_tables` on `public`; note the tables
   (`leads`, `lead_activities`, `pipeline_stages`), their columns, row counts,
   RLS status, and triggers. Confirm realtime is on.
3. Find the current version via the GitHub MCP tools: locate the repo, check the
   latest commit and any open/merged PRs on the working branch — that is the
   "current live version."
4. Confirm the deployed Vercel site matches this repo's `index.html` (repo = source
   of truth for what's live).
5. Post a short "state of the world" summary (what's live, DB row counts, latest
   commit, anything that looks off) and **confirm with the user before editing.**

## Data model (Supabase, all snake_case)
- `leads` — one row per lead (salon/contact/qualification/pipeline/demo fields).
  Has `updated_at` (auto-bumped on every edit) and `inserted_at`.
- `lead_activities` — the follow-up history timeline (`lead_id` → `leads.id`).
- `pipeline_stages` — the 12 columns as data (`key`, `label`, `hint`, `accent`,
  `position`); `leads.status` is a foreign key to `pipeline_stages.key`.
- Triggers on `leads`: `touch_updated_at` (sets `updated_at = now()` on any update)
  and `enforce_cold_on_archive` (forces priority `cold` for junk/lost/called_no_answer).

## Brand (keep exactly)
Accent `#5acec4`, accent ink `#08332f`, accent soft `#5acec424`. Backgrounds
`#f3f6f5` / surface `#fff`, borders `#e7eceb`/`#e1e7e6`, ink `#0f1b1a`.
Priority: Hot `#ef4444`, Warm `#f59e0b`, Cold `#22a06b`. Font: Plus Jakarta Sans;
icons: Material Symbols Outlined. Rounded cards (11–16px), subtle shadows.

## Rules
- This board is LIVE with real leads. Never wipe or bulk-mutate data. Treat schema
  changes and any destructive/irreversible action as needing explicit confirmation.
- Keep all data fields snake_case (Supabase-ready). Preserve the brand look above.
- Develop on the branch above; commit with clear messages; only push when asked;
  don't open a PR unless asked.

## Known quirks (cleanup candidates, not blockers)
- **Stage order** differs between code and DB: the board's `stageDefs()` lists
  `called_no_answer` *before* `junk`, while the `pipeline_stages` table has `junk`
  at position 11 and `called_no_answer` at 12. The board renders from the code order.
- **Some KPI trend badges are hardcoded** (e.g. `+12%`, `+5`) — decorative, not
  computed from live data.
- `seedLeads()` is dead code left over from the original prototype.

## Ready state
Once the checklist is done and you've summarized the current state, you are ready.
Ask the user, in plain language, what they'd like to enhance.
