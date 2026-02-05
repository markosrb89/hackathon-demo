# Frontend — Employee Contribution Dialog Scheduler

> See `../SPEC.md` for system-wide architecture, data models, and API contract.

---

## Technology
- **Framework:** React 18 + TypeScript
- **Routing:** React Router (client-side)
- **State:** Component-local state + React Context for auth
- **API calls:** Thin typed wrapper in `src/api/client.ts` (fetch-based)

---

## Folder Layout (`src/`)
```
src/
├── App.tsx              # Root: AuthProvider + Router (routes defined here)
├── pages/
│   ├── Login            # Role picker (Manager | Employee) — calls POST /auth/login
│   ├── ManagerDashboard # Lists employees, lets manager create review cycles
│   ├── EmployeeDashboard# Shows pending contribution forms for the logged-in employee
│   ├── ContributionForm # Form UI — loads questions, captures answers, POSTs submission
│   └── ReviewStatus     # Per-assignment status view (pending → submitted → meeting scheduled)
├── api/
│   └── client.ts        # Base URL config + typed helpers: get(), post(), etc.
│                         #   Attaches JWT from auth context to every request
└── components/          # Shared UI: StatusChip, EmployeeCard, FormField, etc.
```

---

## Page Responsibilities & Data Flows

### Login
- Renders two buttons: **Manager** / **Employee**.
- On click: `POST /auth/login { email, role }` → stores returned JWT in AuthContext.
- Redirects to the appropriate dashboard based on role.

### ManagerDashboard
- On mount: `GET /employees` → renders employee list.
- Manager selects one or more employees and clicks "Start Review".
- On submit: `POST /reviews { employeeIds }` → creates cycle + assignments.
- Displays active review cycles and their aggregate status (`GET /reviews/:id/status`).

### EmployeeDashboard
- On mount: fetches assignments for the current user (via `/reviews` or a dedicated endpoint).
- Shows each pending assignment as a card with a "Fill Form" link.

### ContributionForm
- Receives `assignmentId` from route params.
- `GET /forms/:assignmentId` — loads the form structure (fields + any existing answers).
- Employee fills in fields; on submit: `POST /forms/:assignmentId { answers }`.
- On success: navigates to ReviewStatus for that assignment.

### ReviewStatus
- Displays the lifecycle status of a single assignment:
  `pending → submitted → meeting scheduled`.
- Fetches meeting details from `GET /meetings/:assignmentId` when status is `meeting_scheduled`.

---

## Auth Context
- Wraps the app; holds `{ user, token }`.
- `user` is decoded from the JWT on login (no extra network call).
- Exposes `login(email, role)` and `logout()`.
- `client.ts` reads the token from context and appends it as a Bearer header automatically.

---

## API Client (`src/api/client.ts`)
- Single source of truth for backend URL (from env var `REACT_APP_API_URL`, default `http://localhost:3001`).
- Exports typed functions that match the backend routes:
  ```
  login(email, role)         → { token }
  getEmployees()             → User[]
  createReview(employeeIds)  → ReviewCycle
  getReviewStatus(cycleId)   → Assignment[]
  getForm(assignmentId)      → FormTemplate
  submitForm(assignmentId, answers) → { meetingId }
  getMeeting(assignmentId)   → MeetingEvent
  ```
- Handles HTTP errors centrally (throws typed errors the pages can catch).
