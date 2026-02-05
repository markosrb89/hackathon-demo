# Implementation Roadmap

Progress tracker for building the Employee Contribution Dialog Scheduler.
Each phase is a natural checkpoint — verify before moving on.

> Reference: `SPEC.md` (architecture), `backend/README.md`, `frontend/README.md`

---

## Phase 1 — Project Scaffolding
Set up both apps so they start, even if empty.

- [ ] `backend/package.json` — express, better-sqlite3, jsonwebtoken, typescript, ts-node, uuid + dev deps
- [ ] `backend/tsconfig.json`
- [ ] `frontend/package.json` — react, react-dom, react-router-dom, typescript + Vite + dev deps
- [ ] `frontend/tsconfig.json` + `vite.config.ts`
- [ ] `.env.example` at project root (`JWT_SECRET`, `MOCK_GRAPH`)
- [ ] `npm install` in both folders

**Checkpoint:** `npm run dev` starts backend (port 3001) and frontend (port 3000) without errors.

---

## Phase 2 — Database Layer
SQLite schema and seed data. No routes yet.

- [ ] `backend/src/db/db.ts` — open/export better-sqlite3 connection (`db.sqlite`)
- [ ] `backend/src/db/schema.ts` — CREATE TABLE for all 5 models: User, ReviewCycle, ReviewAssignment, ContributionForm, MeetingEvent
- [ ] Seed data: 1 manager user, 2–3 employee users (hardcoded, inserted on startup if empty)

**Checkpoint:** Start backend → `db.sqlite` file appears → query confirms tables + seed rows exist.

---

## Phase 3 — Backend Auth & Middleware
Login endpoint + JWT protection layer.

- [ ] `backend/src/routes/auth.ts` — `POST /auth/login` accepts `{ email, role }`, looks up user in DB, signs JWT
- [ ] `backend/src/middleware/authMiddleware.ts` — verify Bearer token, attach `req.user`
- [ ] `backend/src/main.ts` — Express app setup, mount middleware + auth route

**Checkpoint:** `POST /auth/login` returns a valid JWT. Other routes reject requests without it.

---

## Phase 4 — Backend Routes & Services
All business logic and API endpoints.

- [ ] `backend/src/routes/employees.ts` — `GET /employees` (manager sees direct reports)
- [ ] `backend/src/services/reviewService.ts` — create cycle, assign employees, status updates
- [ ] `backend/src/routes/reviews.ts` — `POST /reviews`, `GET /reviews/:id/status`
- [ ] `backend/src/services/formService.ts` — save form, orchestrate post-submit flow (submit → schedule meeting)
- [ ] `backend/src/routes/forms.ts` — `GET /forms/:assignmentId`, `POST /forms/:assignmentId`
- [ ] `backend/src/services/graphService.ts` — mock `createEvent()`: log payload, return fake UUID
- [ ] `backend/src/routes/meetings.ts` — `GET /meetings/:assignmentId`
- [ ] Wire all routes into `main.ts`

**Checkpoint:** Full API works end-to-end: login → create review → submit form → meeting appears (test with curl or Postman).

---

## Phase 5 — Frontend Foundation
App shell, auth wiring, and API client.

- [ ] `frontend/src/App.tsx` — AuthProvider wrapping React Router
- [ ] `frontend/src/api/client.ts` — typed fetch helpers, auto-attach JWT, all endpoint functions
- [ ] Auth context — holds `{ user, token }`, exposes `login()` / `logout()`
- [ ] `frontend/src/pages/Login.tsx` — Manager / Employee role picker, calls login, redirects by role

**Checkpoint:** App starts → Login page renders → selecting a role logs in and navigates.

---

## Phase 6 — Frontend Pages & Components
All user-facing screens.

- [ ] Shared components: `StatusChip`, `EmployeeCard`, `FormField` (in `components/`)
- [ ] `frontend/src/pages/ManagerDashboard.tsx` — employee list, select & start review, display active cycles
- [ ] `frontend/src/pages/EmployeeDashboard.tsx` — list pending assignments, link to form
- [ ] `frontend/src/pages/ContributionForm.tsx` — load form, capture answers, submit
- [ ] `frontend/src/pages/ReviewStatus.tsx` — assignment lifecycle status, meeting details when scheduled

**Checkpoint:** All pages render, pull data from backend, and display correctly for both roles.

---

## Phase 7 — Integration & End-to-End Verification
Full round-trip as a real user would experience it.

- [ ] Manager flow: login → select employees → create review → see pending assignments
- [ ] Employee flow: login → see pending form → fill & submit → see "meeting scheduled" status
- [ ] Manager re-check: dashboard reflects submitted forms + scheduled meetings
- [ ] Console confirms mock Graph event was fired on submission
- [ ] Fix any bugs surfaced during the round-trip

**Checkpoint:** Complete manager → employee → meeting flow works without errors.

---

## Phase 8 — Beyond MVP (future)
Items from the expansion roadmap. Pick up after Phase 7 is solid.

- [ ] Swap mock Graph for real Microsoft Graph API (`MOCK_GRAPH=false` + Azure credentials)
- [ ] Email notifications on assignment creation and form submission
- [ ] Dynamic / configurable contribution form fields
- [ ] Recurring review cycles
- [ ] PDF export of completed forms
- [ ] Granular role-based access control
