# Employee Contribution Dialog Scheduler — Spec

## Overview
Full-stack app: managers select employees for contribution reviews, employees fill out forms, and follow-up meetings are auto-scheduled in Outlook (mocked initially, real Graph API later).

---

## Tech Stack
| Layer | Choice | Rationale |
|---|---|---|
| Backend | Node.js + Express + TypeScript | Lightweight, easy setup |
| Frontend | React + TypeScript | Fast UI iteration |
| Database | SQLite (better-sqlite3) | Zero-config, file-based |
| Outlook | Mock Graph service | Simulates calendar events locally; swap to real Graph API later |
| Auth | Mock auth (role picker) | User picks Manager or Employee on login; JWT issued locally |

---

## Core Data Flow
```
Manager selects employees  →  Review assignments created  →  Employees notified
        ↓
Employees fill & submit form  →  Form saved  →  Follow-up meeting auto-created
        ↓
Manager sees status dashboard (pending / submitted / meeting scheduled)
```

---

## Data Models
```
User             { id, name, email, role (manager|employee), managerId? }
ReviewCycle      { id, managerId, createdAt, status }
ReviewAssignment { id, cycleId, employeeId, status (pending|submitted|meeting_scheduled) }
ContributionForm { id, assignmentId, answers (JSON), submittedAt }
MeetingEvent     { id, assignmentId, graphEventId, scheduledAt }
```

---

## Project Structure
```
hackathon-demo/
├── SPEC.md                         ← this file
├── backend/
│   ├── README.md                   ← backend architecture & data flows
│   └── src/
│       ├── main.ts                 # Express app entry + middleware
│       ├── db/                     # SQLite connection + schema
│       ├── routes/                 # auth, employees, reviews, forms, meetings
│       ├── services/               # graphService, reviewService, formService
│       └── middleware/             # authMiddleware (JWT verify)
├── frontend/
│   ├── README.md                   ← frontend architecture & data flows
│   └── src/
│       ├── App.tsx                 # Router + auth provider
│       ├── pages/                  # ManagerDashboard, EmployeeDashboard, ContributionForm, ReviewStatus
│       ├── api/                    # Typed API client
│       └── components/             # Shared UI primitives
└── .env.example                    # JWT_SECRET, MOCK_GRAPH flag
```

---

## API Endpoints
```
POST   /auth/login             — Mock login { email, role } → JWT
GET    /employees              — List employees (filtered by manager)
POST   /reviews                — Create review cycle + assignments
GET    /reviews/:id/status     — All assignments in a cycle + their statuses
GET    /forms/:assignmentId    — Get form for an assignment
POST   /forms/:assignmentId    — Submit form → triggers mock meeting creation
GET    /meetings/:assignmentId — Meeting status for an assignment
```

---

## Outlook Integration (mocked)
On form submit:
1. Form saved, assignment status → `submitted`
2. `graphService.createEvent()` logs the event details, returns a fake event ID
3. `MeetingEvent` row created; assignment status → `meeting_scheduled`

> To go live: set `MOCK_GRAPH=false`, add Azure credentials, replace mock with `@microsoft/microsoft-graph-client`.

---

## Expansion Roadmap
- [ ] Email notifications on assignment / submission
- [ ] Dynamic / configurable form fields
- [ ] Recurring review cycles
- [ ] PDF export of completed forms
- [ ] Granular role-based access control
