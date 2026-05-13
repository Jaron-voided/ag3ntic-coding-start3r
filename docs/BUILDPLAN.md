# Build Plan — EdgeDesk

_This file is the phased build plan for the project. It's the bridge between `docs/PRD.md` (what to build) + `docs/DESIGN.md` (what it looks like) and the actual code. Re-run the `build-plan` skill whenever reality has diverged from the plan._

> **Status:** In Progress
> **Last updated:** 2026-05-07
> **Current phase:** Phase 0

---

## Why a build plan exists

Claude Code sessions have a finite context window. The cheaper a session is to start, the better the work tends to be. A good build plan slices the project into phases where each phase:

- Has a single user-visible outcome.
- Touches a bounded set of files.
- Names exactly which docs and files Claude should load to execute it.
- Leaves the repo in a clean, testable state at the end.

That way each phase fits in a focused session — no full-repo loads, no thrashing, no context exhaustion mid-implementation.

---

## Strategy

- **Slicing principle:** Vertical slices by user story — each phase ships one user-visible outcome end-to-end.
- **Critical path:** Phase 0 (scaffolding + live URL) → Phase 1 (desktop shell) unblock everything else. The desktop shell is built first so every subsequent panel has a home from day one.
- **Auth deferred on purpose:** v1 is single-client. Auth (Phase 5) is built last — right before handing off to a real client — so the demo is showable without login friction during development.
- **Context boundaries:** Every phase boundary is a `/clear` point. Start each phase in a fresh session loading only what "Context to load" lists for that phase.

---

## Phases

### Phase 0 — Scaffolding

**Goal:** Repo bootstrapped, Cloudflare Pages + Workers wired up, smoke test passes, live URL exists.

**Context to load:** `CLAUDE.md`, `docs/PRD.md §6` (tech shape), `docs/DESIGN.md §3` (component approach).

**Files this phase creates/modifies:**
- `wrangler.toml` — Workers + Pages config, R2/D1/KV bindings declared
- `package.json` — dependencies (Hono, React, Tailwind, Headless UI, Vitest)
- `src/index.ts` — Worker entry point with a single health-check route
- `src/App.tsx` — React app shell (router only, no real pages yet)
- `vitest.config.ts` — test config with `@cloudflare/vitest-pool-workers`
- `tailwind.config.ts` — color tokens from DESIGN.md §4 added
- `README.md` — live URL added after first deploy

**Tests this phase adds:**
- `smoke.test.ts` — Worker `GET /health` returns 200

**Done-when:**
- [ ] `npm test` passes.
- [ ] `wrangler deploy` produces a public URL.
- [ ] Live URL is committed to `README.md`.
- [ ] Tailwind color tokens match `docs/DESIGN.md §4` exactly.
- [ ] R2, D1, and KV bindings are declared in `wrangler.toml` (even if unused yet).

**Session budget:** ~1 session.

**Risks / unknowns:**
- Cloudflare Pages + Workers routing config can be fiddly — verify the `[site]` and `[routes]` sections in `wrangler.toml` before moving on.

---

### Phase 1 — Desktop shell

**Goal:** The `/desktop` route renders the cyberpunk app-launcher: icon grid, working slide-in panels, and all visual tokens in place.

**Context to load:** `CLAUDE.md`, `docs/DESIGN.md §1` (visual identity), `§2` (IA + nav model), `§3` (components), `§4` (tokens), `§5` (a11y floor).

**Files this phase creates/modifies:**
- `src/pages/Desktop.tsx` — the `/desktop` route; renders icon grid
- `src/components/AppIcon.tsx` — icon tile with label and notification badge slot
- `src/components/Panel.tsx` — Headless UI `Dialog`-based slide-in panel wrapper
- `src/pages/Login.tsx` — placeholder `/login` page (no logic yet — just the route)
- `tailwind.config.ts` — Orbitron + Inter fonts wired up
- `public/` — any static assets (wallpaper, logo placeholder)

**Tests this phase adds:**
- `desktop.test.tsx` — Desktop renders 6 app icons
- `panel.test.tsx` — clicking an icon opens Panel; pressing Escape or close button closes it

**Done-when:**
- [ ] `/desktop` loads in a browser showing 6 placeholder icons on the cyberpunk background.
- [ ] Clicking any icon slides open a panel from the right.
- [ ] Close button and Escape key return to the desktop.
- [ ] Tab navigation moves through icons and into the open panel (a11y floor from `DESIGN.md §5`).
- [ ] Neon blue focus rings visible on keyboard nav.
- [ ] `npm test` passes.

**Session budget:** 1–2 sessions.

**Risks / unknowns:**
- Headless UI `Dialog` animation direction — verify it supports slide-from-right or add a custom CSS transition.
- Orbitron is a display font; confirm it loads from Google Fonts without FOUC.

---

### Phase 2 — CRM / Lead list panel

**Goal:** The CRM panel shows real leads fetched from Airtable — client opens it and sees who to follow up with.

**Context to load:** `CLAUDE.md`, `docs/PRD.md §4 story 2`, `§6` (Airtable API, tech shape), `§7` (Airtable rate limit risk), `docs/DESIGN.md §2` (CRM panel description), `src/pages/Desktop.tsx`, `src/components/Panel.tsx`.

**Files this phase creates/modifies:**
- `src/workers/api/leads.ts` — `GET /api/leads` route; fetches from Airtable, returns JSON
- `src/types/lead.ts` — TypeScript type for a lead record
- `src/components/panels/CRMPanel.tsx` — renders lead list inside Panel wrapper
- `src/pages/Desktop.tsx` — wire CRM icon to open CRMPanel

**Tests this phase adds:**
- `leads.test.ts` — `GET /api/leads` returns shaped lead records (mock Airtable response)
- `CRMPanel.test.tsx` — renders lead list; shows empty state when no leads

**Done-when:**
- [ ] CRM icon on desktop opens the CRM panel.
- [ ] Panel shows real Airtable lead data in a list.
- [ ] Loading state shown while fetch is in flight.
- [ ] Empty state shown when no leads exist.
- [ ] `npm test` passes.

**Session budget:** 1 session.

**Risks / unknowns:**
- Airtable API field names may not match assumptions — inspect the real base before writing the type.
- Airtable's 5 req/sec rate limit is fine for demo; flag if multiple panels hit it simultaneously.
- Airtable API key must be stored in `.dev.vars` locally and `wrangler secret` for production — do not hardcode.

---

### Phase 3 — EdgeDrive panel

**Goal:** Client uploads a photo in the EdgeDrive panel; it lands in R2 and fires the n8n webhook that triggers the AI caption + Facebook post.

**Context to load:** `CLAUDE.md`, `docs/PRD.md §4 story 1`, `§6` (R2, n8n webhook), `§7` (n8n reliability risk), `docs/DESIGN.md §2` (EdgeDrive panel), `docs/DESIGN.md §3` (`react-dropzone`), `src/components/Panel.tsx`, `wrangler.toml`.

**Files this phase creates/modifies:**
- `src/workers/api/upload.ts` — `POST /api/upload` route; stores file in R2, fires n8n webhook
- `src/components/panels/EdgeDrivePanel.tsx` — `react-dropzone` UI inside Panel wrapper; shows upload progress, success, and error states
- `src/pages/Desktop.tsx` — wire EdgeDrive icon to open EdgeDrivePanel

**Tests this phase adds:**
- `upload.test.ts` — `POST /api/upload` stores to R2 and fires webhook (mock R2 + fetch)
- `EdgeDrivePanel.test.tsx` — renders dropzone; shows success state after upload

**Done-when:**
- [ ] EdgeDrive icon on desktop opens the upload panel.
- [ ] Drag-and-drop (and click-to-select) file upload works.
- [ ] File appears in R2 bucket after upload.
- [ ] n8n webhook fires (verify in n8n execution log).
- [ ] Success message shown to client after upload.
- [ ] Error state shown if upload or webhook fails.
- [ ] `npm test` passes.

**Session budget:** 1–2 sessions.

**Risks / unknowns:**
- n8n webhook URL stability — confirm the URL and whether it requires auth headers before writing the Worker (PRD §7).
- No feedback loop if n8n fails silently after the webhook fires — this is a known gap (PRD §7), not an EdgeDesk v1 fix.
- R2 multipart upload for large files — out of scope for v1; add a client-side file size warning if needed.

---

### Phase 4 — Deliverables / Resources panel

**Goal:** The Resources panel shows all the links and assets Jaron has delivered — client can find their website, intake forms, and docs without digging through email.

**Context to load:** `CLAUDE.md`, `docs/PRD.md §4 story 3`, `docs/DESIGN.md §2` (Resources panel), `src/components/Panel.tsx`, `wrangler.toml` (D1 binding).

**Files this phase creates/modifies:**
- `migrations/001_deliverables.sql` — D1 table: `deliverables (id, title, url, category, created_at)`
- `src/workers/api/deliverables.ts` — `GET /api/deliverables` route; reads from D1
- `src/types/deliverable.ts` — TypeScript type for a deliverable record
- `src/components/panels/ResourcesPanel.tsx` — renders grouped link list inside Panel wrapper
- `src/pages/Desktop.tsx` — wire Resources icon to open ResourcesPanel
- `seed/deliverables.sql` — demo data seeded for the client

**Tests this phase adds:**
- `deliverables.test.ts` — `GET /api/deliverables` returns records from D1
- `ResourcesPanel.test.tsx` — renders link list; links open in new tab

**Done-when:**
- [ ] Resources icon on desktop opens the panel.
- [ ] Panel shows seeded deliverables grouped by category.
- [ ] All links open in a new tab.
- [ ] Empty state handled.
- [ ] `npm test` passes.

**Session budget:** 1 session.

**Risks / unknowns:**
- D1 in Workers local dev requires `wrangler dev --local` — confirm the binding works before writing API logic.

---

### Phase 5 — Auth

**Goal:** A real client can log in with a username + password and be issued a JWT; all API routes are protected.

**Context to load:** `CLAUDE.md`, `docs/PRD.md §6` (auth shape: JWT, KV sessions), `src/index.ts`, `src/pages/Login.tsx`, `wrangler.toml` (KV binding).

**Files this phase creates/modifies:**
- `src/workers/api/auth.ts` — `POST /api/login` issues JWT; `POST /api/logout` clears session
- `src/workers/middleware/auth.ts` — Hono middleware that validates JWT on protected routes
- `src/pages/Login.tsx` — login form wired to `POST /api/login`; redirects to `/desktop` on success
- `src/index.ts` — apply auth middleware to all `/api/*` routes except `/api/login`
- `wrangler.toml` — KV namespace for session storage confirmed

**Tests this phase adds:**
- `auth.test.ts` — `POST /api/login` with valid credentials returns JWT; invalid credentials return 401
- `auth.test.ts` — protected routes return 401 without a valid token
- `Login.test.tsx` — form submits credentials; redirects on success; shows error on failure

**Done-when:**
- [ ] `/login` page renders and accepts credentials.
- [ ] Successful login redirects to `/desktop`.
- [ ] JWT stored in an `httpOnly` cookie.
- [ ] All `/api/*` routes (except `/api/login`) return 401 without a valid token.
- [ ] Credentials stored via `wrangler secret` — not hardcoded.
- [ ] `npm test` passes.

**Session budget:** 1 session.

**Risks / unknowns:**
- `httpOnly` cookie vs. `localStorage` for JWT — use `httpOnly` cookie to avoid XSS exposure.
- CORS headers on auth routes if the frontend and API are on different origins.

---

## Decision log

| Date | Phase touched | Change | Reason |
|---|---|---|---|
| 2026-05-07 | All | Initial plan | — |
| 2026-05-07 | Phase 5 | Auth deferred to last phase | v1 is single-client; no auth needed for demo; add before real client handoff |
| 2026-05-07 | Phase 1 | Desktop shell moved to Phase 1 (before any data phases) | Build the panel container first so every feature phase drops into a working home |

---

## Handoff notes

The project is "done" when:

- Public URL deployed and linked from `README.md`.
- All three Must-have stories from `PRD.md §4` have green tests and work end-to-end in a browser.
- Full demo flow works: log in → see desktop → open CRM → see real leads → open EdgeDrive → upload photo → n8n fires → Facebook post goes out → open Resources → see deliverables.
- Architecture diagram regenerated and committed.
- Demo video linked from `README.md`.
