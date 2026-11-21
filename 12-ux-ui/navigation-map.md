# Navigation Map — resi-complex

> Defines the complete frontend screen structure covering **all 22 User Stories (`FR01`–`FR22`)**,
> role-based access rules across the **5 canonical authentication roles** (`00-governance/security-policy.md`),
> backing microservices (`09-microservices/service-catalog.md`), and cross-role user flows.
> **Last Synchronized:** 2026-11-21

---

## Frontend Route Structure

```
/                                      → Landing — redirects to /login if not authenticated
│
├── /login                             → HU-IAM-001 (FR01) — Login form (Public)
├── /activate                          → HU-IAM-002 (FR20) — Provisioned account activation (Public token)
│
├── /dashboard                         → Home after login; widgets scoped by role (All 5 roles)
│
├── /admin/users                       → HU-IAM-002 (FR20) — Staff account provisioning & role roster (ADMINISTRATOR)
│
├── /units                             → Units Management (ADMINISTRATOR)
│   ├── /new                           → HU-UNIT-001 (FR02) — Register a unit (RESIDENTIAL / COMMERCIAL)
│   └── /:unitId
│       ├── /                          → HU-UNIT-003 (FR04) — Unit detail + 1:1 commercial establishment profile
│       └── /edit                      → HU-UNIT-002 (FR03) — Edit or soft-delete unit (BR-05 debt guard)
│
├── /people                            → People & Occupancy Management (ADMINISTRATOR)
│   ├── /new                           → HU-PPL-001 (FR21) — Register Person + initial Unit Occupancy
│   └── /:personId
│       ├── /                          → Person detail & occupancy history
│       └── /occupancies               → HU-PPL-002 (FR21, BR-07) — Manage & deactivate tenant occupancies
│
├── /requests                          → Maintenance Requests (FR05, FR06, FR07)
│   ├── /                              → List — scoped by role (all / :own / :assigned)
│   ├── /new                           → HU-MAINT-001 (FR05) — Create private-unit or common-area request
│   └── /:requestId                    → HU-MAINT-002/003 (FR06–FR07) — Assign, progress, verify/reject WORK_COMPLETED
│
├── /fees                              → Administration Fees (FR08, FR09, FR10)
│   ├── /                              → List — ADMINISTRATOR/BOARD all; PERSON :own only
│   └── /:feeId                        → HU-BILL-002 (FR10) — Account statement, partial payment voucher & receipt
│
├── /announcements                     → Segmented Communications & Alerts (FR11)
│   ├── /                              → List segmented by unit type (All 5 roles read)
│   └── /new                           → HU-COMM-001 (FR11) — Publish segmented / emergency announcement (ADMINISTRATOR)
│
├── /front-desk                        → Security Guard Operational Screens (SECURITY_GUARD)
│   ├── /visits
│   │   ├── /                          → HU-ACC-001/003 (FR12, FR14) — Active/recent visits log
│   │   ├── /new                       → HU-ACC-001/002 (FR12–FR13) — Log visitor entry (+ optional vehicle)
│   │   └── /:visitId/exit             → HU-ACC-003 (FR14) — Record visitor departure timestamp
│   ├── /preauthorizations             → HU-ACC-006 (FR22) — Check-in pre-authorized visitors into active visits
│   └── /correspondence
│       ├── /                          → HU-ACC-004/005 (FR15–FR16) — Pending parcels & 4-digit PIN delivery
│       └── /new                       → HU-ACC-004 (FR15) — Log incoming parcel/letter + generate pickup PIN
│
├── /correspondence                    → HU-ACC-005 (FR16) — PERSON's own pending parcels & pickup PIN (:own)
├── /preauthorizations                 → HU-ACC-006 (FR22) — PERSON's own visitor pre-authorizations (:own)
│
├── /expenses                          → Extraordinary Expense Approvals (FR17, FR18)
│   ├── /                              → List (ADMINISTRATOR: all; BOARD: pending review & history)
│   ├── /new                           → HU-FIN-001 (FR17) — Create expense proposal with quorum & deadline
│   └── /:proposalId                   → HU-FIN-002 (FR18) — Proposal detail + immutable BOARD vote (approve/reject)
│
├── /reports                           → HU-REP-001 (FR19) — Requests-by-status & fee arrears (ADMINISTRATOR, BOARD)
│
└── /profile                           → Authenticated user profile & in-app notifications (All 5 roles)
```

---

## Screen Map (26 Screens · 100% of `FR01`–`FR22` Covered)

| # | Screen | Route | Component | Minimum Role | Backing Microservice | Related HU / FR |
|---|---|---|---|---|---|---|
| 01 | Login | `/login` | `LoginPage` | Public | `iam-service` (`:8081`) | `HU-IAM-001` (`FR01`) |
| 02 | Account Activation | `/activate` | `ActivationPage` | Public (token) | `iam-service` (`:8081`) | `HU-IAM-002` (`FR20`) |
| 03 | Dashboard | `/dashboard` | `DashboardPage` | Any authenticated role | `reports-service` (`:8089`) + role feeds | Stitch `#02` |
| 04 | Staff Accounts | `/admin/users` | `UserProvisioningPage` | `ADMINISTRATOR` | `iam-service` (`:8081`) | `HU-IAM-002` (`FR20`) |
| 05 | Units List | `/units` | `UnitListPage` | `ADMINISTRATOR` | `units-service` (`:8083`) | `HU-UNIT-001`, `HU-UNIT-002` (`FR02`–`FR03`) |
| 06 | Register Unit | `/units/new` | `UnitFormPage` | `ADMINISTRATOR` | `units-service` (`:8083`) | `HU-UNIT-001` (`FR02`) |
| 07 | Unit Detail & Establishment | `/units/:unitId` | `UnitDetailPage` | `ADMINISTRATOR` | `units-service` (`:8083`) | `HU-UNIT-003` (`FR04`) |
| 08 | Edit / Delete Unit | `/units/:unitId/edit` | `UnitEditPage` | `ADMINISTRATOR` | `units-service` (`:8083`) | `HU-UNIT-002` (`FR03`) |
| 09 | People List | `/people` | `PersonListPage` | `ADMINISTRATOR` | `people-service` (`:8085`) | `HU-PPL-001` (`FR21`) |
| 10 | Register Person | `/people/new` | `PersonFormPage` | `ADMINISTRATOR` | `people-service` (`:8085`) | `HU-PPL-001` (`FR21`) |
| 11 | Person Occupancies | `/people/:personId/occupancies` | `OccupancyHistoryPage` | `ADMINISTRATOR` | `people-service` (`:8085`) | `HU-PPL-002` (`FR21`) |
| 12 | Requests List | `/requests` | `RequestListPage` | `PERSON` (`:own`) / `MAINTENANCE_STAFF` (`:assigned`) / `ADMINISTRATOR` / `BOARD` | `maintenance-service` (`:8086`) | `HU-MAINT-001`, `HU-MAINT-002` (`FR05`–`FR06`) · Stitch `#03` |
| 13 | Create Request | `/requests/new` | `RequestFormPage` | `PERSON` / `ADMINISTRATOR` | `maintenance-service` (`:8086`) | `HU-MAINT-001` (`FR05`) |
| 14 | Request Detail & Verify | `/requests/:requestId` | `RequestDetailPage` | `PERSON` (`:own`) / `MAINTENANCE_STAFF` (`:assigned`) / `ADMINISTRATOR` / `BOARD` | `maintenance-service` (`:8086`) + `communications-service` | `HU-MAINT-002`, `HU-MAINT-003` (`FR06`–`FR07`) |
| 15 | Fees List | `/fees` | `FeeListPage` | `PERSON` (`:own`) / `ADMINISTRATOR` / `BOARD` (read) | `billing-service` (`:8084`) | `HU-BILL-001`, `HU-BILL-002` (`FR08`–`FR10`) |
| 16 | Fee Detail & Payment | `/fees/:feeId` | `FeeDetailPage` | `PERSON` (`:own`) / `ADMINISTRATOR` / `BOARD` (read) | `billing-service` (`:8084`) | `HU-BILL-002` (`FR10`) |
| 17 | Announcements Feed | `/announcements` | `AnnouncementListPage` | Any authenticated role | `communications-service` (`:8082`) | `HU-COMM-001` (`FR11`) |
| 18 | Publish Announcement | `/announcements/new` | `AnnouncementFormPage` | `ADMINISTRATOR` | `communications-service` (`:8082`) | `HU-COMM-001` (`FR11`) |
| 19 | Front Desk — Visits | `/front-desk/visits` | `VisitLogPage` | `SECURITY_GUARD` / `ADMINISTRATOR` (read) | `access-control-service` (`:8087`) | `HU-ACC-001`, `HU-ACC-003` (`FR12`, `FR14`) · Stitch `#05` |
| 20 | Log Visit (+ Vehicle) | `/front-desk/visits/new` | `VisitFormPage` | `SECURITY_GUARD` | `access-control-service` (`:8087`) | `HU-ACC-001`, `HU-ACC-002` (`FR12`–`FR13`) |
| 21 | Front Desk — Correspondence | `/front-desk/correspondence` | `CorrespondenceLogPage` | `SECURITY_GUARD` | `access-control-service` (`:8087`) | `HU-ACC-004`, `HU-ACC-005` (`FR15`–`FR16`) |
| 22 | Front Desk — Pre-Auth Check-In | `/front-desk/preauthorizations` | `PreAuthCheckInPage` | `SECURITY_GUARD` | `access-control-service` (`:8087`) | `HU-ACC-006` (`FR22`) |
| 23 | My Correspondence & Pre-Auths | `/correspondence`, `/preauthorizations` | `MyFrontDeskPortalPage` | `PERSON` (`:own`) | `access-control-service` (`:8087`) | `HU-ACC-005`, `HU-ACC-006` (`FR16`, `FR22`) |
| 24 | Expenses List & Create | `/expenses`, `/expenses/new` | `ExpenseListPage` / `ExpenseFormPage` | `ADMINISTRATOR` / `BOARD` (read) | `finance-approval-service` (`:8088`) | `HU-FIN-001` (`FR17`) |
| 25 | Expense Detail & Vote | `/expenses/:proposalId` | `ExpenseDetailPage` | `ADMINISTRATOR` (read) / `BOARD` (vote) | `finance-approval-service` (`:8088`) | `HU-FIN-002` (`FR18`) · Stitch `#04` |
| 26 | Reports & Profile | `/reports`, `/profile` | `ReportsPage`, `ProfilePage` | `/reports`: `ADMINISTRATOR`, `BOARD` · `/profile`: All 5 roles | `reports-service` (`:8089`), `iam-service`, `communications-service` | `HU-REP-001` (`FR19`), `HU-MAINT-003` (`FR07`) |

---

## 5-Role Access Matrix (Synchronized 1:1 with `00-governance/security-policy.md` & `07-api/authentication.md`)

| Screen / Route | `ADMINISTRATOR` | `BOARD` | `PERSON` | `MAINTENANCE_STAFF` | `SECURITY_GUARD` |
|---|:---:|:---:|:---:|:---:|:---:|
| `/dashboard` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `/admin/users` (`FR20`) | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/units*` (`FR02`–`FR04`) | ✅ | 👁️ read | ❌ | ❌ | ❌ |
| `/people*` (`FR21`) | ✅ | 👁️ read | ❌ | ❌ | ❌ |
| `/requests` (list) | ✅ all | 👁️ read | ✅ own only | ✅ assigned only | ❌ |
| `/requests/new` (`FR05`) | ✅ | ❌ | ✅ | ❌ | ❌ |
| `/requests/:id` (`FR06`) | ✅ assign / verify | 👁️ read | ✅ own (verify/reject) | ✅ assigned (progress) | ❌ |
| `/fees*` (`FR08`–`FR10`) | ✅ all | 👁️ read | ✅ own only | ❌ | ❌ |
| `/announcements` (read) | ✅ | ✅ | ✅ (scoped) | ✅ | ✅ |
| `/announcements/new` (`FR11`) | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/front-desk/*` (`FR12`–`FR15`, `FR22`) | 👁️ read | 👁️ visits read | ❌ | ❌ | ✅ full ops |
| `/correspondence` & `/preauthorizations` | ❌ | ❌ | ✅ own only | ❌ | ❌ |
| `/expenses` (read) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `/expenses/new` (`FR17`) | ✅ | ❌ | ❌ | ❌ | ❌ |
| `/expenses/:id` (vote `FR18`) | 👁️ read | ✅ approve/reject | ❌ | ❌ | ❌ |
| `/reports` (`FR19`) | ✅ | ✅ | ❌ | ❌ | ❌ |
| `/profile` (incl. in-app notifications) | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## Main User Flows

### Flow 1 — Login & Role-Scoped Session (`HU-IAM-001`)

```
Landing (/)
    │
    ▼ Redirect (no valid session)
Login (/login) ── POST /api/v1/auth/login via api-gateway (:8080) → iam-service (:8081)
    │
    ├── Valid credentials ──► Dashboard (/dashboard) scoped to canonical role
    └── Invalid / Locked  ──► Generic 401/423 error message (no credential enumeration)
```

### Flow 2 — Maintenance Request Lifecycle with Verified Closure (`HU-MAINT-001`–`003`)

```
Person (/requests/new) → Submit PRIVATE_UNIT or COMMON_AREA ticket → status: PENDING
    │
    ▼ (Administrator assigns via /requests/:id)
Maintenance Staff (/requests) → Assigned ticket → IN_PROGRESS → WORK_COMPLETED
    │  (async RabbitMQ → communications-service delivers in-app alert FR07)
    ▼
Person (/requests/:id) reviews completed repair (`BR-09`):
    ├── Verify (+ optional 1–5 rating) ──► status: RESOLVED (terminal)
    └── Reject (+ mandatory notes)     ──► status: IN_PROGRESS (technician alerted)
```

### Flow 3 — Front-Desk Visitor, Vehicle & Pre-Authorization Flow (`HU-ACC-001`–`003`, `006`)

```
Security Guard (/front-desk/visits or /front-desk/preauthorizations)
    │
    ├── Pre-authorized guest ──► Convert ACTIVE pre-authorization to Visit (FR22)
    └── Walk-in guest        ──► /front-desk/visits/new (unit, PERSONAL / COMMERCIAL_CLIENT, optional vehicle)
                                      │
                                      ▼ Active visits table (Alt+N quick add)
                                 Record Exit (/front-desk/visits/:visitId/exit) → Toast confirmation
```

### Flow 4 — Extraordinary Expense Proposal & Immutable Board Voting (`HU-FIN-001`–`002`)

```
Administrator (/expenses/new) → Submit proposal + quorum + deadline → status: PENDING_REVIEW
    │  (async RabbitMQ → communications-service notifies BOARD)
    ▼
Board Member (/expenses/:proposalId) → Reviews justification & destructive confirmation Modal
    │  POST /api/v1/expense-proposals/:id/votes (append-only ledger, 1 vote per member)
    ▼ (Automatic quorum evaluation — BR-15)
    ├── Quorum Approve ──► status: APPROVED (locked, Administrator notified)
    └── Quorum Reject  ──► status: REJECTED (locked, Administrator notified)
```

---

## Navigation Rules

| Rule | Description |
|------|-------------|
| Authentication | Every route except `/login` and `/activate` redirects to `/login` when unauthenticated |
| Authorization | Accessing an unauthorized route redirects to `/dashboard` with a clear `403` feedback banner |
| Ownership Filtering (`P5`) | Client UI filters `/requests`, `/fees`, `/correspondence`, and `/preauthorizations` strictly to own/assigned records, complementing mandatory backend enforcement |
| Destructive Confirmation | Deleting a unit, deactivating a tenant occupancy, or casting an irreversible Board vote always triggers a confirmation `Modal` (`design-system.md`) |
| Session Expiration | Any `401` response triggers silent `/api/v1/auth/refresh` rotation once; on failure, clears session and redirects to `/login` |

---

## Correlations

- Design System & Domain Status Badges → `12-ux-ui/design-system.md`
- High-Fidelity Stitch Screens & UI Kit → `12-ux-ui/wireframes.md`
- 22 BDD User Stories → `04-requirements/user-stories.md`
- 5 Canonical Roles & Permissions → `00-governance/security-policy.md`, `07-api/authentication.md`
- Reconciled 10-Service Catalog → `09-microservices/service-catalog.md`
