# Functional Requirements & Business Rules (`FR01`–`FR22`)

> Formal catalog of all functional requirements (`FR01`–`FR22`), domain business rules
> (`BR-01`–`BR-16`), scope boundaries, and transition requirements for `resi-complex`.
> Written in English per `ADR-001` and aligned 1:1 with `02-domain/`, `04-requirements/user-stories.md`,
> `04-requirements/traceability-matrix.md`, and `06-data/models.md`.

---

## 1. Functional Requirements Catalog (MoSCoW)

| FR ID | Priority (MoSCoW) | Bounded Context | Responsible Service | Owner | Associated HU(s) | Functional Specification (Input → Processing → Output) |
|-------|-------------------|-----------------|---------------------|-------|------------------|--------------------------------------------------------|
| **FR01** | Must Have | IAM | `iam-service` | Fabián Trujillo | `HU-IAM-001` | Authenticate users via email + BCrypt password; issue 1h JWT (with single RBAC role claim) and 7-day rotatable `refresh_token`; deny invalid credentials with generic `401`. |
| **FR02** | Must Have | Units Management | `units-service` | María Alexandra Garzón | `HU-UNIT-001` | Register a physical unit (`unit_number`, `unit_type`: `RESIDENTIAL` or `COMMERCIAL`), enforce uniqueness, and publish `UnitRegistered` via `outbox`. |
| **FR03** | Should Have | Units Management | `units-service` | María Alexandra Garzón | `HU-UNIT-002` | Edit or soft-delete (`deleted_at`) an existing unit; block deletion when active unpaid/overdue fees exist; publish `UnitUpdated`. |
| **FR04** | Must Have | Units Management | `units-service` | María Alexandra Garzón | `HU-UNIT-003` | Register 1:1 commercial establishment data (`business_name`, `business_type`, `opening_time`, `closing_time`) exclusively for `COMMERCIAL` units; publish `CommercialEstablishmentRegistered`. |
| **FR05** | Must Have | Maintenance | `maintenance-service` | María Alexandra Garzón | `HU-MAINT-001` | Create a maintenance request for either `PRIVATE_UNIT` (requires `unit_id`) or `COMMON_AREA` (requires `common_area_name`) with `type`, `description`, and mandatory `priority`; save as `PENDING` and publish `MaintenanceRequestCreated`. |
| **FR06** | Must Have | Maintenance | `maintenance-service` | María Alexandra Garzón | `HU-MAINT-002` | Assign request to `MAINTENANCE_STAFF`, advance states (`PENDING` → `ASSIGNED` → `IN_PROGRESS` → `WORK_COMPLETED` → `RESOLVED`), allow resident rejection back to `IN_PROGRESS` with `rejection_notes` or approval with `resident_rating` (1–5); publish `MaintenanceRequestStatusUpdated`. |
| **FR07** | Should Have | Communications | `communications-service` | Julián Losada | `HU-MAINT-003` | Consume `MaintenanceRequestStatusUpdated` idempotently (`processed_events`) and generate an in-app notification for the requesting `Person`. |
| **FR08** | Must Have | Billing | `billing-service` | Félix Murcia | `HU-BILL-001` | Generate monthly `administration_fees` per active unit for a `YYYY-MM` period (idempotent per `(unit_id, period)`); publish `FeeGenerated`. |
| **FR09** | Must Have | Billing | `billing-service` | Félix Murcia | `HU-BILL-001` | Apply differentiated fee rates based on `unit_type` (`RESIDENTIAL` vs. `COMMERCIAL`) and freeze `unit_type_at_generation` permanently (`INV-002`). |
| **FR10** | Must Have | Billing | `billing-service` | Félix Murcia | `HU-BILL-002` | Allow active unit Owners and Tenants (`fees:read:own`) to inspect `total_amount`, `paid_amount`, and `balance`, submit partial/full payment vouchers (`PENDING_VERIFICATION`), and allow Administrator approval updating fee status (`PARTIALLY_PAID` / `PAID`) and publishing `FeePaid`. |
| **FR11** | Should Have | Communications | `communications-service` | Julián Losada | `HU-COMM-001` | Publish announcements segmented by `scope` (`ALL`, `RESIDENTIAL_ONLY`, `COMMERCIAL_ONLY`) and `priority` (`INFORMATIVE`, `IMPORTANT`, `EMERGENCY`), emitting `AnnouncementPublished` and urgent in-app broadcast alerts for `EMERGENCY`. |
| **FR12** | Must Have | Access Control | `access-control-service` | Julián Losada | `HU-ACC-001` | Log visitor entry at front desk (`PERSONAL` or `COMMERCIAL_CLIENT`) against a validated destination unit; publish `VisitRegistered`. Street-facing walk-in shop customers bypass residential gate logs. |
| **FR13** | Could Have | Access Control | `access-control-service` | Julián Losada | `HU-ACC-002` | Optionally attach complete vehicle data (`plate_number`, `vehicle_type`) to a visit record without allowing partial vehicle entries. |
| **FR14** | Should Have | Access Control | `access-control-service` | Julián Losada | `HU-ACC-003` | Record exit timestamp (`exit_at >= entry_at`) for an open visit and publish `VisitEnded`. |
| **FR15** | Must Have | Access Control | `access-control-service` | Julián Losada | `HU-ACC-004` | Log incoming correspondence for a valid unit as `PENDING`, auto-generate a 4-digit `pickup_pin` visible only to active unit occupants, and publish `CorrespondenceReceived`. |
| **FR16** | Should Have | Access Control | `access-control-service` | Julián Losada | `HU-ACC-005` | Allow active unit occupants (`correspondence:read:own`) to view pending packages with their 4-digit `pickup_pin` and allow the Security Guard to confirm handover (`DELIVERED`) strictly upon matching PIN, publishing `CorrespondenceDelivered`. |
| **FR17** | Must Have | Financial Approval | `finance-approval-service` | Félix Murcia | `HU-FIN-001` | Allow Administrator to submit an extraordinary expense proposal (`amount > 0`, `voting_deadline`, `required_votes > 0`) in `UNDER_REVIEW` status and publish `ExpenseProposalCreated`. |
| **FR18** | Must Have | Financial Approval | `finance-approval-service` | Félix Murcia | `HU-FIN-002` | Allow Board members to record append-only votes (`APPROVED` / `REJECTED`, optional `comments`, one vote per member per proposal) and auto-transition the proposal once majority/quorum threshold is reached, publishing `ExpenseProposalApproved` or `ExpenseProposalRejected`. |
| **FR19** | Should Have | Reports | `reports-service` | Félix Murcia | `HU-REP-001` | Maintain event-driven CQRS read projections (`requests_by_status_view`, `pending_fees_view`) via idempotent consumers (`processed_events`) and expose read-only summary reports (`reports:read`). |
| **FR20** | Must Have | IAM | `iam-service` | Fabián Trujillo | `HU-IAM-002` | Allow Administrator to provision staff accounts (`SECURITY_GUARD`, `MAINTENANCE_STAFF`, `BOARD`) and generate one-time activation invitations for `PERSON` accounts (`PENDING_ACTIVATION` → `ACTIVE`) without public self-registration. |
| **FR21** | Must Have | People Management | `people-service` | María Alexandra Garzón | `HU-PPL-001`, `HU-PPL-002` | Register `Person` profiles (`RESIDENT`, `COMMERCIAL_OWNER`, `COMMERCIAL_TENANT`) with active `unit_occupancies` (`OWNER` / `TENANT`), and deactivate tenancy (`INACTIVE` + `end_date`) upon lease termination to revoke `:own` unit access immediately while preserving audit history; publish `PersonRegistered`. |
| **FR22** | Should Have | Access Control | `access-control-service` | Julián Losada | `HU-ACC-006` | Allow active unit occupants to pre-authorize expected guests/contractors (`PENDING`) and allow Security Guards to verify and convert valid pre-authorizations (`USED`) directly into a logged `Visit`. |

---

## 2. Business Rules Catalog (`BR-01`–`BR-16`)

Technology-independent domain policies enforced in the Domain layer (`02-domain/entities-and-rules.md`) and mirrored in database constraints (`06-data/models.md`):

| Rule ID | Domain | Rule Definition | Enforced By |
|---------|--------|-----------------|-------------|
| **BR-01** | IAM | Public self-registration is prohibited; only an `ADMINISTRATOR` may provision staff or initiate resident account activation. | `iam-service` (`HU-IAM-002`) |
| **BR-02** | IAM | Every user account holds **exactly one** of the 5 RBAC roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`). | `users.chk_users_role`, `security-policy.md` |
| **BR-03** | IAM | Refresh tokens are single-use with mandatory rotation; reuse of any revoked refresh token immediately revokes all active tokens in that `token_family_id`. | `iam-service`, `refresh_tokens` |
| **BR-04** | Units | Every unit is strictly `RESIDENTIAL` or `COMMERCIAL`; a `commercial_establishments` record exists 1:1 **only** for `COMMERCIAL` units. | `units-service`, `uq_comm_est_unit_id` |
| **BR-05** | Units | A unit with pending, partially paid, or overdue administration fees cannot be deleted. | `units-service` (`HU-UNIT-002`) |
| **BR-06** | People | `RESIDENT`, `COMMERCIAL_OWNER`, and `COMMERCIAL_TENANT` are `Person` subtypes — never separate authentication roles. | `people-service`, `chk_person_type` |
| **BR-07** | People / Security | Ownership-level authorization (`:own`) requires an `ACTIVE` `unit_occupancies` record; deactivating a tenancy revokes unit access immediately without deleting historical records. | `people-service`, `unit_occupancies` |
| **BR-08** | Maintenance | A maintenance request requires `unit_id` when `location_type = PRIVATE_UNIT`, or `common_area_name` when `location_type = COMMON_AREA`, and always requires a valid `priority`. | `chk_maintreq_location_ref`, `INV-001`, `INV-003` |
| **BR-09** | Maintenance | Valid lifecycle transitions are `PENDING → ASSIGNED → IN_PROGRESS → WORK_COMPLETED → RESOLVED` (with resident rejection `WORK_COMPLETED → IN_PROGRESS` requiring `rejection_notes`); `RESOLVED` is terminal. | `maintenance-service`, `INV-002` |
| **BR-10** | Billing | At most one administration fee may be generated per `(unit_id, period)` with `total_amount > 0`; `unit_type_at_generation` is immutable once created. | `uq_fee_unit_period`, `chk_fees_total_amount`, `INV-002` |
| **BR-11** | Billing | Approved payments accumulate into `paid_amount` where `0 <= paid_amount <= total_amount`; when `balance = total_amount - paid_amount` reaches `0`, status transitions terminally to `PAID`. | `chk_fees_paid_amount`, `INV-003`, `INV-005` |
| **BR-12** | Billing / Access | Both registered `OWNER` and active `TENANT` of a unit share `fees:read:own` visibility and may submit payment vouchers. | `billing-service`, `INV-004` |
| **BR-13** | Access Control | Street-facing external customers entering commercial shops directly from the public street are not logged as residential visitors. | `access-control-service`, `AGGR-INV-003` |
| **BR-14** | Access Control | Correspondence delivery (`PENDING → DELIVERED`) is terminal and requires matching the auto-generated 4-digit `pickup_pin` (`INV-003`). | `correspondence.chk_correspondence_pin` |
| **BR-15** | Finance Approval | Board votes (`approvals`) are append-only, immutable, and limited to one vote per Board member per proposal; decisions on `APPROVED` or `REJECTED` proposals are forbidden. | `uq_approvals_proposal_member`, `AGGR-INV-001`, `AGGR-INV-002` |
| **BR-16** | Data Protection | All personal data storage and access comply with Colombian **Ley 1581 de 2012** (Habeas Data), soft-delete auditability, and least-privilege `:own` / `:assigned` isolation. | All services, `NFR-004` |

---

## 3. Transition, Legacy Migration & Scope Boundaries

### Transition from Manual Operations
- **Legacy state:** Mixed residential complexes currently operate with paper front-desk logbooks, Excel fee sheets, and unstructured WhatsApp groups (`01-context/overview.md`).
- **Migration strategy (Initial Onboarding):**
  1. Bootstrap initial `ADMINISTRATOR` account in `iam-service` via environment-supplied BCrypt hash (never hardcoded credentials).
  2. Bulk-load existing units (`units-service`) and active residents/owners/tenants (`people-service`) before the first billing cycle cut-over.
  3. Run a 1-billing-cycle **dual-run verification** comparing existing Excel fee balances against generated `administration_fees` before retiring manual spreadsheets.
  4. **Incremental evolution (Strangler Fig Principle):** Future post-MVP capabilities (online payment gateways, external SMTP/SMS providers, hardware RFID gates) are designed to plug in as new hexagonal secondary adapters without rewriting core domain aggregates.

### Explicit MVP Exclusions (Out of Scope)
Per `01-context/scope.md`, the following remain **excluded** from the MVP delivery:
- External third-party integrations (payment gateways such as Stripe/PayU, external SMTP/SMS/WhatsApp providers — all MVP notifications and activation links operate in-app / via local provisioning).
- Full NIIF accounting, physical access hardware (RFID/electronic locks), electronic assembly voting, common-area booking calendar, AI document assistant, and multi-complex SaaS tenancy.
- *(Scope alignment: digital visitor pre-authorization `FR22` / `HU-ACC-006` is included in Sprint 4 scope and synchronized with `01-context/scope.md`; only physical QR/barcode scanning remains out of scope.)*

---

## Correlations

- Detailed BDD User Stories → `04-requirements/user-stories.md`
- FR-to-Service development order → `04-requirements/fr-to-service-mapping.md`
- Measurable quality criteria → `04-requirements/non-functional.md`
- Full Traceability Matrix (RTM) → `04-requirements/traceability-matrix.md`
- Domain entities & invariants → `02-domain/entities-and-rules.md`
- Physical MySQL 8 schemas → `06-data/models.md`
