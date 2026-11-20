# User Stories — Backlog

> Product User Story backlog for `resi-complex` (**22 HUs** across 3 delivery Cuts).
> Every HU includes verifiable **Given / When / Then** (Gherkin) Acceptance Criteria covering
> both happy-path and negative/edge cases, aligned 1:1 with `04-requirements/functional.md`,
> `04-requirements/traceability-matrix.md`, and `06-data/models.md`.

---

## Backlog status

| Cut | Sprint | Total HUs | Refined | In progress | Completed |
|-----|--------|-----------|---------|-------------|-----------|
| Cut 1 | Sprint 1–2 | 7 | 7 | 0 | 0 |
| Cut 2 | Sprint 3–4 | 11 | 11 | 9 | 0 |
| Cut 3 | Sprint 5+ | 4 | 4 | 4 | 0 |
| **Total** | — | **22** | **22** | **13** | **0** |

---

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | IAM | Authenticate users and resolve role-based permissions for the system's 5 roles |
| EP-002 | Units Management | Register and classify units (residential/commercial) and their commercial establishment data |
| EP-003 | People Management | Register residents and commercial owners/tenants and manage their unit occupancy lifecycle |
| EP-004 | Maintenance | Manage the lifecycle of maintenance requests (private units and common areas), from creation to verified resolution |
| EP-005 | Billing | Generate differentiated administration fees by unit type and process full/partial payment vouchers |
| EP-006 | Communications | Publish announcements segmented by unit type with urgency levels and reactive in-app notifications |
| EP-007 | Access Control | Log visitor/vehicle entry-exit, manage visitor pre-authorizations, and verify correspondence handover via PIN |
| EP-008 | Financial Approval | Manage extraordinary expense proposals and the Board of Trustees' append-only quorum voting history |
| EP-009 | Reports | Consolidate read-only CQRS views (requests by status, pending fee arrears) from other services' events |

---

## User Stories

### HU-IAM-001 — User login with role-based access {#HU-IAM-001}

**Epic:** EP-001

> **As** a user with one of the 5 authentication roles (Administrator, Board of Trustees, Person, Maintenance Staff, Security Guard)
> **I want** to log in with my email and password
> **so that** I can access only the features my role allows

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful login
  Given a registered active user with valid credentials
  When  they submit email and password
  Then  the system returns a JWT with their single role and a 1-hour expiration
  And   a single-use rotatable refresh token valid for 7 days is issued and persisted as a BCrypt hash

Scenario 2: Invalid credentials
  Given a user submits the wrong password or an unknown email
  When  they attempt to log in
  Then  the system responds 401 with a generic error message, without revealing whether the email exists
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Fabián Trujillo |
| Status | Backlog |
| Dependencies | — |
| Requirement (FR) | FR01 |
| Affected service(s) | iam-service |

---

### HU-IAM-002 — Administrator creates staff accounts and triggers user activation {#HU-IAM-002}

**Epic:** EP-001

> **As** an Administrator
> **I want** to register user accounts for complex staff (Security Guards, Maintenance Staff, Board members) and issue activation tokens for Residents
> **so that** users obtain secure personal credentials without public self-registration

**Acceptance Criteria:**

```gherkin
Scenario 1: Administrator creates a Security Guard account
  Given I am logged in as Administrator
  When  I register a user with email "guardia@complex.com" and role "SECURITY_GUARD"
  Then  the user account is created with status "PENDING_ACTIVATION"
  And   a secure one-time activation token/link with expiration is generated (in-app/admin provisioning in MVP; external SMTP out of scope)

Scenario 2: Resident completes activation and sets password
  Given a resident holds a valid unexpired activation token
  When  they submit the token and a secure password meeting complexity rules
  Then  their account status becomes "ACTIVE" with a BCrypt password hash
  And   they can log in using their email and newly set password

Scenario 3: Non-administrator attempts staff creation
  Given a user with role "PERSON" or "SECURITY_GUARD"
  When  they attempt to invoke the user creation endpoint
  Then  the system returns 403 Forbidden
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | Fabián Trujillo |
| Status | Backlog |
| Dependencies | HU-IAM-001 |
| Requirement (FR) | FR20 |
| Affected service(s) | iam-service |

---

### HU-PPL-001 — Register a Person and assign Unit Occupancy {#HU-PPL-001}

**Epic:** EP-003

> **As** an Administrator
> **I want** to register a Person (Resident or Commercial Owner/Tenant), define their occupancy type (Owner or Tenant) and link them to a unit
> **so that** the complex census is accurate, the user receives an activation invitation, and ownership-level authorization is established

**Acceptance Criteria:**

```gherkin
Scenario 1: Register an owner or tenant with active occupancy
  Given a residential or commercial unit already exists
  When  the Administrator registers a Person with personal details, assigns occupancy_type "OWNER" or "TENANT", and links them to that unit
  Then  the Person is saved with an ACTIVE UnitOccupancy record and a PersonRegistered event is published via outbox
  And   an account activation token/link is provisioned for the user via iam-service (in-app provisioning in MVP; no external SMTP)

Scenario 2: Register a person against a non-existent unit
  Given the Administrator provides a unitId that does not exist
  When  they try to register the Person
  Then  the system rejects the request with 400/422 validation error
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-IAM-001, HU-UNIT-001 |
| Requirement (FR) | FR21 |
| Affected service(s) | people-service |

---

### HU-PPL-002 — Deactivate tenant occupancy upon lease termination {#HU-PPL-002}

**Epic:** EP-003

> **As** an Administrator
> **I want** to terminate/deactivate an occupant's tenancy when their lease contract ends or they move out
> **so that** their access to the unit's features (correspondence pickup, visitor pre-authorization, maintenance requests) is revoked immediately without deleting their personal profile or past history

**Acceptance Criteria:**

```gherkin
Scenario 1: Deactivate tenant occupancy on lease termination
  Given a Person has an "ACTIVE" occupancy as "TENANT" for unit "Apto 301"
  When  the Administrator records the lease termination (or the lease reaches its end date)
  Then  the UnitOccupancy status changes to "INACTIVE" and end_date is recorded
  And   the user loses ownership-level authorization (:own) over unit "Apto 301" immediately
  And   all past maintenance requests, visits, and correspondence associated with this person and unit remain intact for audit

Scenario 2: Post-termination access restriction
  Given a user whose occupancy for unit "Apto 301" is "INACTIVE"
  When  they try to access unit "Apto 301" resources (e.g., pre-authorize a visitor or view correspondence)
  Then  the system returns 403 Forbidden
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-PPL-001 |
| Requirement (FR) | FR21 |
| Affected service(s) | people-service |

---

### HU-UNIT-001 — Register a unit {#HU-UNIT-001}

**Epic:** EP-002

> **As** an Administrator
> **I want** to register a unit and classify it as residential or commercial
> **so that** the system can later apply the correct fee rate and segment announcements correctly

**Acceptance Criteria:**

```gherkin
Scenario 1: Register a residential unit
  Given I am logged in as Administrator
  When  I register a unit with unit_type = RESIDENTIAL
  Then  the unit is saved and a UnitRegistered event is published via outbox

Scenario 2: Register a commercial unit without establishment data
  Given I register a unit with unit_type = COMMERCIAL
  When  I do not provide the commercial establishment data
  Then  the system rejects the request, since a commercial unit must have an establishment (AGGR-INV-001)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-IAM-001 |
| Requirement (FR) | FR02 |
| Affected service(s) | units-service |

---

### HU-UNIT-002 — Edit and delete a unit {#HU-UNIT-002}

**Epic:** EP-002

> **As** an Administrator
> **I want** to edit or soft-delete an existing unit
> **so that** I can correct registration mistakes or deactivate units that no longer operate

**Acceptance Criteria:**

```gherkin
Scenario 1: Edit a unit's data
  Given an existing active unit
  When  the Administrator updates its data
  Then  the change is saved and a UnitUpdated event is published via outbox

Scenario 2: Delete a unit with active fees
  Given a unit has PENDING, PARTIALLY_PAID, or OVERDUE administration fees
  When  the Administrator tries to delete it
  Then  the system blocks the deletion with 409 Conflict until those fees are resolved (BR-05)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Sprint 1 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requirement (FR) | FR03 |
| Affected service(s) | units-service |

---

### HU-UNIT-003 — Register commercial establishment data {#HU-UNIT-003}

**Epic:** EP-002

> **As** an Administrator
> **I want** to register the commercial establishment data (business name, business type, operating hours) for a commercial unit
> **so that** the system has the information needed to operate and display that unit correctly

**Acceptance Criteria:**

```gherkin
Scenario 1: Register establishment for a commercial unit
  Given a unit already exists with unit_type = COMMERCIAL and no establishment attached
  When  the Administrator registers its business_name, business_type, opening_time, and closing_time
  Then  the establishment is saved 1:1 with the unit and a CommercialEstablishmentRegistered event is published via outbox

Scenario 2: Reject establishment registration for a residential unit
  Given a unit exists with unit_type = RESIDENTIAL
  When  the Administrator attempts to attach a commercial establishment to it
  Then  the system rejects the request with 422 Unprocessable Entity (BR-04)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Sprint 1 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-UNIT-001 |
| Requirement (FR) | FR04 |
| Affected service(s) | units-service |

---

### HU-MAINT-001 — Create a maintenance request {#HU-MAINT-001}

**Epic:** EP-004

> **As** a Person (Resident or Commercial Owner/Tenant)
> **I want** to create a maintenance request describing the type, description, location, and priority of the issue
> **so that** the Administrator can assign it and I can track its progress

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful private-unit request creation
  Given I am logged in as Person with an active occupancy
  When  I create a request with location_type = PRIVATE_UNIT, my unitId, type, description, and priority
  Then  the request is saved with status PENDING and a MaintenanceRequestCreated event is published via outbox

Scenario 2: Missing priority
  Given I do not select a valid priority (LOW, MEDIUM, HIGH, URGENT)
  When  I try to submit the request
  Then  the system rejects it with 400 Bad Request (INV-003: priority is required)

Scenario 3: Report common area maintenance
  Given an issue located in a common area (e.g. "Elevator Tower 1")
  When  I create a request selecting location_type = COMMON_AREA and specify common_area_name = "Elevator Tower 1"
  Then  the request is saved with status PENDING without requiring a private unitId (INV-001, BR-08)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-PPL-001, HU-UNIT-001 |
| Requirement (FR) | FR05 |
| Affected service(s) | maintenance-service |

---

### HU-MAINT-002 — Assign and track a maintenance request {#HU-MAINT-002}

**Epic:** EP-004

> **As** an Administrator
> **I want** to assign a maintenance request to a Maintenance Staff member and track its status through resident verification
> **so that** no request gets lost and there is verified traceability of what was done

**Acceptance Criteria:**

```gherkin
Scenario 1: Assign a pending request
  Given a request with status PENDING
  When  the Administrator assigns it to a Maintenance Staff member
  Then  the status changes to ASSIGNED and MaintenanceRequestStatusUpdated is published

Scenario 2: Technical work completion and resident verification
  Given a request in status IN_PROGRESS
  When  the Maintenance Staff marks the repair completed
  Then  the status becomes WORK_COMPLETED
  And   when the resident approves conformity with optional resident_rating (1–5) (or 48h elapses), status transitions to RESOLVED

Scenario 3: Resident rejects unsatisfactory repair
  Given a request in status WORK_COMPLETED
  When  the resident rejects the repair with rejection_notes
  Then  the status returns to IN_PROGRESS for rework (INV-002, BR-09)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | María Alexandra Garzón |
| Status | Backlog |
| Dependencies | HU-MAINT-001 |
| Requirement (FR) | FR06 |
| Affected service(s) | maintenance-service |

---

### HU-MAINT-003 — Notify a maintenance status change {#HU-MAINT-003}

**Epic:** EP-004

> **As** a Person who created a maintenance request
> **I want** to be notified in-app whenever its status changes
> **so that** I know the progress without having to ask

**Acceptance Criteria:**

```gherkin
Scenario 1: Status change notification
  Given my maintenance request changes from ASSIGNED to IN_PROGRESS
  When  communications-service consumes the MaintenanceRequestStatusUpdated event
  Then  I receive a single in-app notification recording the updated status

Scenario 2: Duplicate event redelivery is handled idempotently
  Given a MaintenanceRequestStatusUpdated event with the same event_id was already recorded in processed_events
  When  the broker redelivers the event
  Then  communications-service ignores the duplicate without creating a second notification
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8082`) |
| Dependencies | HU-MAINT-002 |
| Requirement (FR) | FR07 |
| Affected service(s) | communications-service |

---

### HU-BILL-001 — Generate the monthly differentiated fee {#HU-BILL-001}

**Epic:** EP-005

> **As** an Administrator
> **I want** the system to generate the monthly administration fee per unit, with a rate that differs by unit type
> **so that** residential and commercial units are billed fairly and automatically

**Acceptance Criteria:**

```gherkin
Scenario 1: Generate a fee for a commercial unit
  Given a commercial unit and its commercial rate configured for period "2026-11"
  When  the monthly fee generation runs
  Then  a fee is created with status PENDING, total_amount > 0, paid_amount = 0.00, and unit_type_at_generation = COMMERCIAL
  And   a FeeGenerated event is published via outbox

Scenario 2: Unit type changes after fee generation
  Given a fee was already generated for a unit this period
  When  the unit's type is changed afterward
  Then  the already-generated fee keeps its original unit_type_at_generation (INV-002, BR-10)

Scenario 3: Duplicate generation for same unit and period is rejected
  Given a fee already exists for unit U in period "2026-11"
  When  monthly fee generation is triggered again for the same unit and period
  Then  the system prevents duplicate creation (uq_fee_unit_period)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 8 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Félix Murcia |
| Status | In Progress (Phase 1 Repo `:8084`) |
| Dependencies | HU-UNIT-001 |
| Requirement (FR) | FR08, FR09 |
| Affected service(s) | billing-service |

---

### HU-BILL-002 — Check fee payment status and submit partial/full payments {#HU-BILL-002}

**Epic:** EP-005

> **As** a Person (Owner or active Tenant)
> **I want** to check the payment status of my own unit's fees and submit payment vouchers
> **so that** I know what I owe and can settle balances partially or in full transparently

**Acceptance Criteria:**

```gherkin
Scenario 1: View own fees (dual visibility for Owner and active Tenant)
  Given I have an ACTIVE occupancy for my unit with generated fees
  When  I open my fee status view
  Then  I see total_amount, paid_amount, and balance for my unit only (fees:read:own, INV-004)

Scenario 2: Submit payment voucher and record partial payment
  Given an unpaid or partially paid fee for my unit
  When  I submit a payment with amount_paid <= balance and a voucher_url
  Then  the payment is registered with status "PENDING_VERIFICATION"
  And   upon Administrator approval, paid_amount and balance update atomically and fee status becomes PARTIALLY_PAID or PAID (INV-005)

Scenario 3: Overpayment attempt is rejected
  Given a fee with remaining balance of 150000.00
  When  a payment of 200000.00 is submitted or approved
  Then  the system rejects the operation with 422 Unprocessable Entity (chk_fees_paid_amount, BR-11)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Félix Murcia |
| Status | In Progress (Phase 1 Repo `:8084`) |
| Dependencies | HU-BILL-001 |
| Requirement (FR) | FR10 |
| Affected service(s) | billing-service |

---

### HU-COMM-001 — Publish a segmented announcement {#HU-COMM-001}

**Epic:** EP-006

> **As** an Administrator
> **I want** to publish an announcement segmented by unit scope and urgency level
> **so that** residents and merchants receive only the information relevant to them and are immediately alerted on emergencies

**Acceptance Criteria:**

```gherkin
Scenario 1: Publish to commercial units only
  Given I write an announcement with scope = COMMERCIAL_ONLY and priority = INFORMATIVE
  When  I publish it
  Then  only Persons linked to commercial units see it, and an AnnouncementPublished event is published via outbox

Scenario 2: Emergency announcement in-app broadcast
  Given a critical complex event (e.g. immediate utility outage)
  When  I publish an announcement with scope = ALL and priority = EMERGENCY
  Then  the announcement is persisted and an urgent AnnouncementPublished event triggers real-time in-app emergency alerts to all active occupants (external email/SMS deferred to post-MVP per 01-context/scope.md)

Scenario 3: Non-administrator attempts to publish an announcement
  Given I am logged in as PERSON or SECURITY_GUARD
  When  I attempt to publish an announcement
  Then  the system returns 403 Forbidden
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Sprint 5 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8082`) |
| Dependencies | HU-UNIT-001 |
| Requirement (FR) | FR11 |
| Affected service(s) | communications-service |

---

### HU-ACC-001 — Log a visitor's entry {#HU-ACC-001}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to log an external visitor's entry into the residential perimeter, distinguishing personal visitors from commercial clients
> **so that** there is a traceable record of who enters and to which unit

**Acceptance Criteria:**

```gherkin
Scenario 1: Log a personal visitor
  Given a valid destination unit
  When  I log the entry with visitor_type = PERSONAL
  Then  the visit is saved with entry_at timestamp and a VisitRegistered event is published via outbox

Scenario 2: Invalid destination unit
  Given I enter a unitId that does not exist
  When  I try to save the visit
  Then  the system rejects it with 400/422 validation error (AGGR-INV-001)

Scenario 3: Street-facing commercial customer entry
  Given a customer visiting a street-facing commercial establishment directly from the public street
  When  they enter the store without crossing the residential perimeter
  Then  no residential visit is logged at the front desk (AGGR-INV-003, BR-13)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8087`) |
| Dependencies | HU-UNIT-001 |
| Requirement (FR) | FR12 |
| Affected service(s) | access-control-service |

---

### HU-ACC-002 — Register a visitor's vehicle {#HU-ACC-002}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to optionally register a vehicle (plate number and vehicle type) with a visit
> **so that** there is a record of the vehicle associated with that visitor

**Acceptance Criteria:**

```gherkin
Scenario 1: Vehicle with complete data
  Given I am logging a visit for a visitor arriving by vehicle
  When  I provide both plate_number and a valid vehicle_type (CAR, MOTORCYCLE, BICYCLE, TRUCK, OTHER)
  Then  the vehicle record is persisted atomically with the visit (AGGR-INV-002)

Scenario 2: Partial vehicle data rejected
  Given I provide plate_number without vehicle_type (or vice versa)
  When  I try to save the visit
  Then  the system rejects the request with 400 Bad Request (AGGR-INV-002: no partial vehicle data)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Could Have |
| Target sprint | Sprint 3 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8087`) |
| Dependencies | HU-ACC-001 |
| Requirement (FR) | FR13 |
| Affected service(s) | access-control-service |

---

### HU-ACC-003 — Log a visitor's exit {#HU-ACC-003}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to log a visitor's exit
> **so that** the front-desk record shows the complete visit, from entry to exit

**Acceptance Criteria:**

```gherkin
Scenario 1: Log exit for an open visit
  Given a visit was logged with exit_at = NULL
  When  I record the visitor's exit with exit_at >= entry_at
  Then  exit_at is saved and a VisitEnded event is published via outbox

Scenario 2: Attempt to log exit twice or before entry_at
  Given a visit already has exit_at recorded (or submitted exit_at < entry_at)
  When  I try to log an exit
  Then  the system rejects the request with 409 Conflict or 422 Unprocessable Entity (chk_visits_exit)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8087`) |
| Dependencies | HU-ACC-001 |
| Requirement (FR) | FR14 |
| Affected service(s) | access-control-service |

---

### HU-ACC-004 — Log incoming correspondence {#HU-ACC-004}

**Epic:** EP-007

> **As** a Security Guard
> **I want** to log a package or letter received for a unit
> **so that** the occupant knows they have something pending pickup

**Acceptance Criteria:**

```gherkin
Scenario 1: Log correspondence for a valid unit
  Given a valid destination unit
  When  I log an incoming package for it
  Then  it is saved with status PENDING, a 4-digit numeric pickup_pin is auto-generated for unit occupants, and a CorrespondenceReceived event is published via outbox

Scenario 2: Log correspondence for a non-existent unit
  Given I provide a unitId that does not exist
  When  I try to log the package
  Then  the system rejects the request with 400/422 validation error (INV-001)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Must Have |
| Target sprint | Sprint 3 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8087`) |
| Dependencies | HU-UNIT-001 |
| Requirement (FR) | FR15 |
| Affected service(s) | access-control-service |

---

### HU-ACC-005 — Check pending correspondence and confirm PIN handover {#HU-ACC-005}

**Epic:** EP-007

> **As** a Person
> **I want** to check my own unit's pending correspondence and confirm pickup with a 4-digit PIN
> **so that** I know what is waiting for me and packages are never handed to the wrong person

**Acceptance Criteria:**

```gherkin
Scenario 1: View pending correspondence
  Given I have an ACTIVE occupancy and correspondence logged for my unit with status PENDING
  When  I open my correspondence view
  Then  I see only correspondence for my own unit (correspondence:read:own) along with its 4-digit pickup_pin

Scenario 2: Collect package with matching 4-digit PIN
  Given I have a PENDING package with an issued 4-digit pickup_pin
  When  I provide the PIN to the Security Guard and the guard submits it
  Then  the correspondence transitions to DELIVERED with delivered_at timestamp and delivered_to reference, publishing CorrespondenceDelivered (INV-003, BR-14)

Scenario 3: Wrong PIN or already delivered package is rejected
  Given the Security Guard submits an incorrect PIN (or the package is already DELIVERED)
  When  handover confirmation is requested
  Then  the system rejects the transition with 422/409 error (INV-002, INV-003)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 2 |
| Priority | Should Have |
| Target sprint | Sprint 3 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8087`) |
| Dependencies | HU-ACC-004 |
| Requirement (FR) | FR16 |
| Affected service(s) | access-control-service |

---

### HU-ACC-006 — Pre-authorize a visitor {#HU-ACC-006}

**Epic:** EP-007

> **As** a Person (Resident or Commercial Owner/Tenant)
> **I want** to pre-authorize the entry of a visitor or service contractor in advance
> **so that** the visitor experiences seamless entry at the gate without phone call interruptions

**Acceptance Criteria:**

```gherkin
Scenario 1: Pre-authorize a guest
  Given I am logged in as Person with an ACTIVE occupancy
  When  I register a visitor pre-authorization with visitor_name, valid_date, and optional vehicle_plate
  Then  the pre-authorization is saved with status PENDING

Scenario 2: Guard validates pre-authorized visitor
  Given an incoming visitor with a valid PENDING pre-authorization for today's date
  When  the Security Guard verifies the plate or visitor name and confirms entry
  Then  the pre-authorization status transitions to USED
  And   a Visit entry record is automatically logged (AGGR-INV-004)

Scenario 3: Expired or already used pre-authorization is rejected
  Given a pre-authorization with status EXPIRED or USED
  When  the Security Guard attempts to redeem it for entry
  Then  the system rejects the redemption with 409 Conflict
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Should Have |
| Target sprint | Sprint 4 |
| Assigned to | Julián Losada |
| Status | In Progress (Phase 1 Repo `:8087`) |
| Dependencies | HU-UNIT-001, HU-PPL-001 |
| Requirement (FR) | FR22 |
| Affected service(s) | access-control-service |

---

### HU-FIN-001 — Create an extraordinary expense proposal {#HU-FIN-001}

**Epic:** EP-008

> **As** an Administrator
> **I want** to create a proposal for an extraordinary expense with voting deadline and quorum
> **so that** the Board of Trustees can review and decide on it formally

**Acceptance Criteria:**

```gherkin
Scenario 1: Create a valid proposal
  Given a title, description, amount > 0, required_votes > 0, and a future voting_deadline
  When  I submit the proposal
  Then  it is saved with status UNDER_REVIEW and an ExpenseProposalCreated event is published via outbox

Scenario 2: Amount is zero or negative
  Given I enter an amount <= 0
  When  I submit the proposal
  Then  the system rejects it with 400/422 validation error (chk_proposals_amount, AGGR-INV-003)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 3 |
| Priority | Must Have |
| Target sprint | Sprint 5 |
| Assigned to | Félix Murcia |
| Status | In Progress (Phase 1 Repo `:8088`) |
| Dependencies | HU-IAM-001 |
| Requirement (FR) | FR17 |
| Affected service(s) | finance-approval-service |

---

### HU-FIN-002 — Approve or reject an expense proposal {#HU-FIN-002}

**Epic:** EP-008

> **As** a Board of Trustees member
> **I want** to approve or reject an extraordinary expense proposal with optional rationale comments
> **so that** there is a permanent, auditable record of the decision

**Acceptance Criteria:**

```gherkin
Scenario 1: Board member records append-only vote
  Given a proposal with status UNDER_REVIEW and I have not voted on it yet
  When  I submit decision (APPROVED or REJECTED) with optional comments
  Then  my vote is permanently appended to approvals (AGGR-INV-002, BR-15) and the corresponding tally increments

Scenario 2: Automatic majority approval on quorum
  Given a proposal with required_votes threshold reached before voting_deadline
  When  the decisive positive vote is recorded
  Then  the proposal status transitions to APPROVED and an ExpenseProposalApproved event is published via outbox (AGGR-INV-004)

Scenario 3: Duplicate vote or decision on closed proposal is rejected
  Given a proposal is already APPROVED/REJECTED, or I already voted on it
  When  I attempt to submit another vote
  Then  the system rejects it with 409 Conflict (uq_approvals_proposal_member, AGGR-INV-001)
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Must Have |
| Target sprint | Sprint 5 |
| Assigned to | Félix Murcia |
| Status | In Progress (Phase 1 Repo `:8088`) |
| Dependencies | HU-FIN-001 |
| Requirement (FR) | FR18 |
| Affected service(s) | finance-approval-service |

---

### HU-REP-001 — View requests and pending-fee reports {#HU-REP-001}

**Epic:** EP-009

> **As** an Administrator or Board of Trustees member
> **I want** to see real-time summary reports of maintenance requests by status and pending fee arrears
> **so that** I have accurate management visibility without slowing down transactional databases

**Acceptance Criteria:**

```gherkin
Scenario 1: View event-driven reports
  Given maintenance requests and administration fees events have been consumed idempotently into reports_db
  When  an Administrator or Board member opens the reports view (reports:read)
  Then  the system returns requests grouped by status (requests_by_status_view) and pending-fee arrears per unit (pending_fees_view)

Scenario 2: Unauthorized role attempts to access reports
  Given I am logged in as PERSON, SECURITY_GUARD, or MAINTENANCE_STAFF
  When  I request the reports endpoint
  Then  the system returns 403 Forbidden
```

**Definition of Done:**
- [ ] Code reviewed and approved
- [ ] Unit tests written (≥ 80% coverage, ≥ 90% domain)
- [ ] Acceptance criteria verified (manual or automated)
- [ ] API contract updated if applicable
- [ ] Verified in local Docker environment

| Field | Value |
|-------|-------|
| Story Points | 5 |
| Priority | Should Have |
| Target sprint | Sprint 5 |
| Assigned to | Félix Murcia |
| Status | In Progress (Phase 1 Repo `:8089`) |
| Dependencies | HU-MAINT-001, HU-BILL-001 |
| Requirement (FR) | FR19 |
| Affected service(s) | reports-service |

---

## Rules for writing HUs

1. **Specific role:** Never use "As a user" — name one of the 5 roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`).
2. **Real business value:** The `so that` clause states an operational outcome, not a restatement of the button/endpoint.
3. **Verifiable Gherkin (happy + edge):** Every HU must include at least one happy-path scenario and one error/edge scenario using canonical enum values from `06-data/models.md`.
4. **Right-sized:** Completable inside a single sprint (≤ 8 Story Points).

---

## Correlations

- Functional Requirements & Business Rules → `04-requirements/functional.md`
- FR → Service Mapping → `04-requirements/fr-to-service-mapping.md`
- Non-Functional Requirements → `04-requirements/non-functional.md`
- Full Traceability Matrix (RTM) → `04-requirements/traceability-matrix.md`
- HU Template → `04-requirements/_template-hu.md`
- Physical Data Schemas → `06-data/models.md`
