# Backend — Employee Contribution Dialog Scheduler

> See `../SPEC.md` for system-wide architecture, data models, and API contract.

---

## Technology
- **Runtime:** Node.js + TypeScript
- **Framework:** Express
- **Database:** SQLite via `better-sqlite3` (file: `db.sqlite`, auto-created on startup)
- **Auth:** Simple JWT (signed with `JWT_SECRET` from `.env`); mock login endpoint issues tokens

---

## Folder Layout (`src/`)
```
src/
├── main.ts              # Express app factory: mounts middleware + routes
├── db/
│   ├── db.ts            # Opens/exports the SQLite connection
│   └── schema.ts        # CREATE TABLE statements + seed data (1 manager, 1 employee)
├── middleware/
│   └── authMiddleware   # Extracts & verifies JWT; attaches user to req
├── routes/
│   ├── auth             # POST /auth/login — accepts {email, role}, returns signed JWT
│   ├── employees        # GET /employees — returns user list; managers see direct reports
│   ├── reviews          # POST /reviews (create cycle + assignments), GET /reviews/:id/status
│   ├── forms            # GET & POST /forms/:assignmentId — retrieve / submit contribution form
│   └── meetings         # GET /meetings/:assignmentId — returns meeting status + details
└── services/
    ├── reviewService    # Creates ReviewCycle rows, assigns employees, updates statuses
    ├── formService      # Persists form answers, orchestrates post-submit flow
    └── graphService     # Outlook integration layer (mock or real; see below)
```

---

## Request → Response Flow
```
HTTP request
  → authMiddleware (verify JWT, attach user)
    → route handler (validate input, call service)
      → service (business logic, DB writes)
        → response (JSON)
```

## Form Submit → Meeting Schedule Flow
```
POST /forms/:assignmentId
  → formService.submit()
      1. INSERT ContributionForm (answers JSON)
      2. UPDATE ReviewAssignment status → 'submitted'
      3. graphService.createEvent({ subject, attendees, time })
           └── mock: console.log + return fake UUID
           └── real: Graph API POST /events (future)
      4. INSERT MeetingEvent (graphEventId, scheduledAt)
      5. UPDATE ReviewAssignment status → 'meeting_scheduled'
  → 201 { assignmentId, meetingId }
```

---

## Auth Details
- `POST /auth/login` receives `{ email, role }` — no password check (mock).
- Signs a JWT with `{ id, email, role }` payload.
- All other routes require `Authorization: Bearer <token>`.
- `authMiddleware` decodes the token and sets `req.user`.

## Graph Service (Mock → Real)
- Default (`MOCK_GRAPH=true`): logs event payload, returns `{ id: <uuid>, status: 'mock' }`.
- Real mode: reads `AZURE_*` env vars, calls Microsoft Graph `/me/events` or `/users/{id}/events`.
- The switch is a single `if` in `graphService`; no other code changes needed.

---

## Environment
```
JWT_SECRET=<any secret string>
MOCK_GRAPH=true          # set false + add Azure creds to use real Outlook
```
