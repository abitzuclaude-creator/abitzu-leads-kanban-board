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

## Delivery workflow — how changes reach the live site (FOLLOW THIS EXACTLY)
The user is non-technical and **reviews every change on a preview before it goes
live.** Never push straight to production. The pipeline is:

1. **Develop** on the working branch `claude/abitzu-lead-pipeline-ok2uvn` and commit.
2. **Push** the branch. This does NOT change the live site — `main` is production;
   the working branch is not.
3. **Open a Pull Request to `main`.** Vercel automatically builds a **preview
   deployment** and posts a preview link on the PR (as a `vercel[bot]` comment, and
   as a "Vercel" commit status with a `target_url`). Fetch that preview URL with the
   GitHub tools (`pull_request_read` → `get_comments` or `get_status`) and give it to
   the user as their testing link.
4. **The user tests on the preview** and approves — or asks for changes, in which
   case iterate on the same branch; each new push refreshes the same preview URL.
5. **Only after explicit approval, merge the PR into `main`.** That merge is the
   moment it goes live. Never merge without the user's clear go-ahead.

So pushing the branch and opening a PR is expected and safe (preview only). Merging
to `main` is the single irreversible "go live" step and always needs approval.

## Rules
- This board is LIVE with real leads. Never wipe or bulk-mutate data. Treat schema
  changes and any destructive/irreversible action as needing explicit confirmation.
- Keep all data fields snake_case (Supabase-ready). Preserve the brand look above.
- Develop on the working branch with clear commit messages. Pushing the branch and
  opening a PR to `main` (for the preview) is expected; **never merge to `main` (go
  live) without the user's explicit approval.** See "Delivery workflow" above.

## Roles & Permissions / access control (IN PROGRESS — keep this complete)
An admin-only, role-based access-control system is being built from the
`Roles & Permissions.dc.html` design (in the user's Claude Design project). Shape:
- **Data model:** `roles`, `permissions`, `role_permissions` (the matrix),
  `team_members` (login → role). Supabase-ready, snake_case.
- **Screen:** an admin-only "Access control" page with two tabs — **Permissions**
  (a role × permission toggle matrix) and **Team** (members list with a role
  dropdown + status). Reached via an entry point only Admins can see.
- **Roles:** exactly ONE **Administrator** role — always full access, not editable,
  and NOT shown as a column in the matrix. All other roles are created by the user
  via the **New Role** button (nothing else pre-seeded).
- **Enforcement is three layers:** (1) the screen sets permissions, (2) the app
  hides/disables actions per the current user's role, (3) Supabase RLS enforces it
  for real at the DB. Only layer 3 is true security.
- **Hide vs. disable (agreed hybrid):** *hide* whole pages/sections a role has no
  business in (e.g. no "View analytics" → the Analytics entry point is absent);
  *disable* individual fields/buttons inside a screen they CAN see (greyed,
  `not-allowed` cursor, hover tooltip/lock icon "you don't have permission").
- **Permission list (Phase 0, agreed):** 4 groups — **Board & Leads** (view / view
  all POCs' / create / priority / reassign POC / source), **Lead details — edit by
  section** (Follow-up, Client Information, Qualification, Demo Information, Demo
  Feedback, plus Add follow-up), **Pipeline** (single Move & archive toggle),
  **Analytics** (single View). No Demos/Communication/Administration groups. No
  separate "filter" permission (filtering only reshapes your own visible set;
  "View all POCs' leads" is the real gate).
- **Edit-by-section UX (agreed):** the lead drawer keeps ONE Edit button (no
  per-section buttons). Clicking it flips the card to edit mode where only the
  allowed sections become editable; disallowed sections stay visible/readable but
  locked (greyed, not-allowed cursor, hover lock icon). If a role can edit NO
  section, the single Edit button is hidden entirely.

**Progress:** Phase 0 (standalone `/access/` preview screen) done. **Phase 1a DONE**
— Supabase tables `roles` / `permissions` / `role_permissions` / `team_members`
created (RLS on; reads = any authenticated, writes = owners only via
`public.is_access_owner()`, a TEMP email allowlist: chhavi@/anuj@/abitzu.claude@
— replace with a team_members→locked-role lookup once real logins are wired).
Seeded: Owner role (locked) + the 14 permissions + Owner=all. `team_members` empty
(Team tab still shows front-end mock names by user's choice). **Phase 1b DONE** —
`/access/` is wired to Supabase: owner-gated on the signed-in session (only
abitzu.claude@gmail.com; others get a denied panel — NO separate login form), the
matrix loads from the tables and toggles save via `role_permissions` upsert, New
Role inserts a real role + its permission rows. Owner is hidden from the matrix;
until a non-owner role exists it shows a "Create your first role" empty state. The
live board (`index.html` bundle) now has an **owner-only "Access" header button**
(sc-if `isOwnerUser` === abitzu.claude@gmail.com) linking to `/access/`. Team tab
still mock (labelled "Preview"). Board (`index.html` bundle) has an owner-only
"Access" header button linking to `/access/`. **Phase 1c DONE** — Team tab is REAL:
`team_members` seeded from the 7 auth.users logins (Abitzu Claude/Chhavi/Anuj =
owner; Rupesh/Hitesh/Vipul/Sales = unassigned); the tab lists real logins with a
role dropdown (Unassigned + real roles) that saves to `team_members.role_key`, real
last-active + status, and matrix role headers now show real member counts.
`team_members` has unique(user_id) + unique(lower(email)). **Next = Phase 2:**
enforce permissions on the LIVE board — read the signed-in user's role via
team_members → its role_permissions → apply the hybrid hide/disable (single Edit
button unlocks only allowed lead-card sections). CONFIRM the exact gates with the
user before changing board behavior. Then Phase 3 (RLS on `leads`, keyed off
team_members role). NOTE: `role_permissions`/config write-RLS still uses the temp
owner-email allowlist (chhavi@/anuj@/abitzu.claude@) — swap to a team_members
locked-role lookup in Phase 3. New Supabase logins won't auto-appear in team_members
(no trigger yet) — add one, or seed manually, when onboarding real teammates.

**STANDING RULE — when you build ANY new user-facing feature or action, you MUST
also wire its permission into this system:** define the permission (in the matrix /
`permDefs`), gate the feature by it in the app, and enforce it at the DB (RLS) if it
touches data. A new action with no matching permission is an access-control gap —
access control must always stay complete.

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
