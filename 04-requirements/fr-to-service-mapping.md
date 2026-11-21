# Functional Requirements → Services Mapping

> Canonical mapping from functional requirements (`04-requirements/functional.md`, `FR01`–`FR22`)
> and user stories (`04-requirements/user-stories.md`) to the 9 microservices of `resi-complex`.
> Service names align strictly with `02-domain/domain-map.md`, `05-architecture/overview.md`,
> and `06-data/models.md`.

---

## FR → HU → Bounded Context → Service Matrix

| FR | Associated HU(s) | Description | Bounded Context | Service (`Port`) | OpenAPI Contract | Owner | Target Cut / Sprint |
|----|------------------|-------------|-----------------|------------------|------------------|-------|---------------------|
| **FR01** | `HU-IAM-001` | User login with RBAC (5 roles), JWT (1h) and refresh token rotation (7d) | IAM | `iam-service` (`:8081`) | `auth-service.yaml` | Fabián Trujillo | Cut 1 · Sprint 1 |
| **FR20** | `HU-IAM-002` | Admin staff account creation & secure user activation flow | IAM | `iam-service` (`:8081`) | `auth-service.yaml` | Fabián Trujillo | Cut 1 · Sprint 1 |
| **FR02** | `HU-UNIT-001` | Register residential or commercial unit | Units Management | `units-service` (`:8083`) | `units-service.yaml` | María Alexandra Garzón | Cut 1 · Sprint 1 |
| **FR03** | `HU-UNIT-002` | Edit or soft-delete unit (blocked if unpaid fees exist) | Units Management | `units-service` (`:8083`) | `units-service.yaml` | María Alexandra Garzón | Cut 1 · Sprint 1 |
| **FR04** | `HU-UNIT-003` | Register 1:1 commercial establishment data | Units Management | `units-service` (`:8083`) | `units-service.yaml` | María Alexandra Garzón | Cut 1 · Sprint 1 |
| **FR21** | `HU-PPL-001`, `HU-PPL-002` | Register Person with Unit Occupancy & deactivate tenancy on lease end | People Management | `people-service` (`:8085`) | `people-service.yaml` | María Alexandra Garzón | Cut 1 · Sprint 1–2 |
| **FR05** | `HU-MAINT-001` | Create maintenance request (private unit or common area) | Maintenance | `maintenance-service` (`:8086`) | `maintenance-service.yaml` | María Alexandra Garzón | Cut 2 · Sprint 3 |
| **FR06** | `HU-MAINT-002` | Assign, track, verify, or reject maintenance request lifecycle | Maintenance | `maintenance-service` (`:8086`) | `maintenance-service.yaml` | María Alexandra Garzón | Cut 2 · Sprint 3 |
| **FR07** | `HU-MAINT-003` | Notify requester in-app on maintenance status change | Communications | `communications-service` (`:8082`) | `communications-service.yaml` | Julián Losada | Cut 2 · Sprint 3 |
| **FR08** | `HU-BILL-001` | Generate monthly administration fee per unit | Billing | `billing-service` (`:8084`) | `billing-service.yaml` | Félix Murcia | Cut 2 · Sprint 3 |
| **FR09** | `HU-BILL-001` | Apply unit-type differentiated rate frozen at generation | Billing | `billing-service` (`:8084`) | `billing-service.yaml` | Félix Murcia | Cut 2 · Sprint 3 |
| **FR10** | `HU-BILL-002` | Check own unit fee status, submit payment vouchers & partial payments | Billing | `billing-service` (`:8084`) | `billing-service.yaml` | Félix Murcia | Cut 2 · Sprint 3 |
| **FR12** | `HU-ACC-001` | Log visitor entry into residential perimeter | Access Control | `access-control-service` (`:8087`) | `access-control-service.yaml` | Julián Losada | Cut 2 · Sprint 3 |
| **FR13** | `HU-ACC-002` | Register optional visitor vehicle with visit | Access Control | `access-control-service` (`:8087`) | `access-control-service.yaml` | Julián Losada | Cut 2 · Sprint 3 |
| **FR14** | `HU-ACC-003` | Log visitor exit timestamp | Access Control | `access-control-service` (`:8087`) | `access-control-service.yaml` | Julián Losada | Cut 2 · Sprint 3 |
| **FR15** | `HU-ACC-004` | Log incoming correspondence with 4-digit pickup PIN | Access Control | `access-control-service` (`:8087`) | `access-control-service.yaml` | Julián Losada | Cut 2 · Sprint 3 |
| **FR16** | `HU-ACC-005` | Check pending correspondence & confirm PIN handover | Access Control | `access-control-service` (`:8087`) | `access-control-service.yaml` | Julián Losada | Cut 2 · Sprint 3 |
| **FR22** | `HU-ACC-006` | Pre-authorize visitor in advance & convert to Visit on entry | Access Control | `access-control-service` (`:8087`) | `access-control-service.yaml` | Julián Losada | Cut 2 · Sprint 4 |
| **FR11** | `HU-COMM-001` | Publish segmented announcements & emergency in-app broadcasts | Communications | `communications-service` (`:8082`) | `communications-service.yaml` | Julián Losada | Cut 3 · Sprint 5 |
| **FR17** | `HU-FIN-001` | Create extraordinary expense proposal with quorum & deadline | Financial Approval | `finance-approval-service` (`:8088`) | `finance-service.yaml` | Félix Murcia | Cut 3 · Sprint 5 |
| **FR18** | `HU-FIN-002` | Record append-only Board votes & auto-resolve on quorum | Financial Approval | `finance-approval-service` (`:8088`) | `finance-service.yaml` | Félix Murcia | Cut 3 · Sprint 5 |
| **FR19** | `HU-REP-001` | Event-driven summary reports (requests by status & fee arrears) | Reports | `reports-service` (`:8089`) | `reports-service.yaml` | Félix Murcia | Cut 3 · Sprint 5 |

---

## Implementation Dependencies & Delivery Order

1. **Critical inter-service dependencies:**
   - `iam-service` (`:8081`) + `api-gateway` (`:8080`, local fallback `:8090`) are **foundational blockers** for end-to-end authenticated flows across all other services (both Phase 1 repos live).
   - `units-service` (`:8083`) is upstream of `people-service` (`:8085`), `billing-service` (`:8084`), `maintenance-service` (`:8086`), and `access-control-service` (`:8087`) (`unit_id` validation / `UnitRegistered` event).
   - `people-service` (`:8085`) is upstream of `:own` ownership checks (`unit_occupancies.status = 'ACTIVE'`) in `billing-service`, `maintenance-service`, and `access-control-service`.
   - `reports-service` (`:8089`) consumes domain events published via `outbox` from all operational services.

2. **Roadmap delivery cuts (aligned with `03-product/vision.md` & `03-product/roadmap.md`):**
   - **Cut 1 — H1 (Sprints 1–2 · MVP Core):** `iam-service`, `api-gateway`, `units-service`, `people-service` *(7 HUs)*.
   - **Cut 2 — H2 (Sprints 3–4 · Daily Operations):** `maintenance-service`, `billing-service`, `access-control-service` + reactive maintenance notifications in `communications-service` *(11 HUs)*.
   - **Cut 3 — H3 (Sprint 5+ · Governance & Visibility):** `communications-service` (announcements), `finance-approval-service`, `reports-service` *(4 HUs)*.

---

**Last synchronized:** 2026-11-21
