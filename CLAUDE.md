# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Status
The project is scaffolded structurally but has no code yet. `IMPLEMENTATION.md` is the phase-by-phase progress tracker — check it to see where work currently stands before writing anything.

## Key Documents
- `SPEC.md` — single source of truth for data models, API contract, and system data flow
- `IMPLEMENTATION.md` — ordered build phases with checkpoints; update checkboxes as phases complete
- `backend/README.md` — backend layer details (routes, services, DB flows)
- `frontend/README.md` — frontend layer details (pages, auth context, API client contract)

---

## Commands (once scaffolding is complete — Phase 1 of IMPLEMENTATION.md)

```sh
# Backend (port 3001)
cd backend
npm install
npm run dev          # start Express via ts-node/tsx

# Frontend (port 3000)
cd frontend
npm install
npm run dev          # start Vite dev server
npm run build        # production build
```

Backend and frontend are independent — each has its own `package.json` and `node_modules`.

---

## Architecture — What to Know Before Writing Code

### Backend layering: route → service → DB
Every feature follows the same three-layer pattern. Routes handle HTTP shape (parse, validate, respond). Services own business logic and DB writes. Never put DB queries directly in a route handler.

```
routes/reviews.ts  →  services/reviewService.ts  →  db queries
routes/forms.ts    →  services/formService.ts     →  db queries + graphService
```

### The critical post-submit chain
The most interconnected flow in the system. When an employee submits a form, `formService` does five things in order:
1. INSERT ContributionForm
2. UPDATE ReviewAssignment → `submitted`
3. Call `graphService.createEvent()` (mock or real)
4. INSERT MeetingEvent with the returned event ID
5. UPDATE ReviewAssignment → `meeting_scheduled`

Any change to this flow needs to keep all five steps in sync.

### Mock ↔ Real Graph toggle
`graphService.ts` is the only file that touches Outlook. It reads `MOCK_GRAPH` from env: `true` = log + fake UUID, `false` = live Microsoft Graph API. No other file needs to know which mode is active. Do not spread Graph logic elsewhere.

### Auth is intentionally thin
Mock login: `POST /auth/login` accepts `{ email, role }`, looks up the user in the seeded DB, signs a JWT. `authMiddleware` verifies it on every other route. No password hashing, no registration flow. This is by design for the current phase.

### Frontend auth context + API client are coupled by design
`AuthContext` holds the token. `client.ts` reads it from that context and attaches it as a Bearer header on every request. When adding a new API call, add the typed function to `client.ts` — pages should never call `fetch()` directly.

### Seeded data
The DB seeds 1 manager and 2–3 employees on first startup. Tests and manual verification rely on these users. Don't assume a signup/registration flow exists.

---

## Conventions
- **TypeScript everywhere** — both backend and frontend. No `.js` files.
- **SQLite is the DB** — `better-sqlite3`, synchronous API. File lives at `backend/db.sqlite` (gitignored).
- **Ports** — backend 3001, frontend 3000. Frontend proxies API calls to backend in dev.
- **Status enum on ReviewAssignment** — only three values: `pending`, `submitted`, `meeting_scheduled`. Keep this tight; don't add statuses without updating SPEC.md.
- **env vars** — `JWT_SECRET` and `MOCK_GRAPH` are the only required vars. Documented in `.env.example`.
