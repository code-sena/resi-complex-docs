# Traceability Matrix (RTM)

> End-to-end Requirements Traceability Matrix connecting **Product Objectives (`03-product/vision.md`) →
> Functional Requirements (`FR01`–`FR22`) → Business Rules (`BR-01`–`BR-16`) → User Stories (`22 HUs`) →
> Test Classes → Microservices → Delivery Status**.

---

## How to use this matrix

```
Vision KR → FR / NFR → BR / Invariant → HU (Gherkin) → Automated Test → Microservice
```

- Every FR must trace to at least one HU and one automated test class.
- Every NFR must trace to a measurable metric and automated/manual verification tool.
- Status reflects actual team delivery progress as of **2026-11-21**.

---

## 1. FR → HU → BR → Test → Service Matrix (22 FRs · 22 HUs)

| FR ID | Vision KR | FR Description | Business Rules | HU(s) | Automated Test Class(es) | Service | Owner | Status |
|-------|-----------|----------------|----------------|-------|--------------------------|---------|-------|--------|
| **FR01** | KR1, KR4 | Authenticate users (5 roles), 1h JWT + 7d refresh rotation | BR-02, BR-03 | `HU-IAM-001` | `AuthenticationServiceTest.java` | `iam-service` | Fabián Trujillo | 🟡 In Progress (Phase 1 Repo `:8081` — 3/3 tests, Flyway + Hexagonal) |
| **FR20** | KR1 | Admin staff account creation & secure user activation | BR-01, BR-02 | `HU-IAM-002` | `UserProvisioningServiceTest.java` | `iam-service` | Fabián Trujillo | 🟡 In Progress (Phase 1 Repo `:8081` — activation flow implemented) |
| **FR02** | KR1 | Register unit (`RESIDENTIAL` / `COMMERCIAL`) | BR-04 | `HU-UNIT-001` | `UnitServiceTest.java` | `units-service` | María Alexandra Garzón | 🔴 Pending (Repo initialized `:8083`, OpenAPI complete) |
| **FR03** | KR1 | Edit and soft-delete unit (blocked with unpaid fees) | BR-05 | `HU-UNIT-002` | `UnitServiceTest.java` | `units-service` | María Alexandra Garzón | 🔴 Pending (Repo initialized `:8083`, OpenAPI complete) |
| **FR04** | KR1 | Register 1:1 commercial establishment data | BR-04 | `HU-UNIT-003` | `CommercialEstablishmentServiceTest.java` | `units-service` | María Alexandra Garzón | 🔴 Pending (Repo initialized `:8083`, OpenAPI complete) |
| **FR21** | KR1 | Register Person with Unit Occupancy & deactivate tenancy | BR-06, BR-07, BR-16 | `HU-PPL-001`, `HU-PPL-002` | `PersonServiceTest.java`, `UnitOccupancyServiceTest.java` | `people-service` | María Alexandra Garzón | 🔴 Pending (Spec & OpenAPI complete `:8085`) |
| **FR05** | KR1, KR4 | Create maintenance request (private unit / common area) | BR-08 | `HU-MAINT-001` | `MaintenanceRequestServiceTest.java` | `maintenance-service` | María Alexandra Garzón | 🔴 Pending (Spec & OpenAPI complete `:8086`) |
| **FR06** | KR1 | Assign, track, verify, or reject maintenance request | BR-09 | `HU-MAINT-002` | `MaintenanceRequestServiceTest.java` | `maintenance-service` | María Alexandra Garzón | 🔴 Pending (Spec & OpenAPI complete `:8086`) |
| **FR07** | KR1 | Notify requester in-app on maintenance status change | BR-09 | `HU-MAINT-003` | `MaintenanceNotificationServiceTest.java` | `communications-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8082`) |
| **FR08** | KR1, KR4 | Generate monthly administration fee per unit | BR-10 | `HU-BILL-001` | `FeeGenerationServiceTest.java` | `billing-service` | Félix Murcia | 🟡 In Progress (Phase 1 Repo `:8084`) |
| **FR09** | KR1 | Apply unit-type differentiated rate fixed at generation | BR-10 | `HU-BILL-001` | `FeeGenerationServiceTest.java` | `billing-service` | Félix Murcia | 🟡 In Progress (Phase 1 Repo `:8084`) |
| **FR10** | KR1 | Check own fee status, submit vouchers & partial payments | BR-11, BR-12 | `HU-BILL-002` | `FeeQueryServiceTest.java`, `PaymentServiceTest.java` | `billing-service` | Félix Murcia | 🟡 In Progress (Phase 1 Repo `:8084`) |
| **FR11** | KR1 | Publish segmented announcements & emergency in-app broadcasts | BR-04 | `HU-COMM-001` | `AnnouncementServiceTest.java` | `communications-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8082`) |
| **FR12** | KR1, KR4 | Log visitor entry (`PERSONAL` / `COMMERCIAL_CLIENT`) | BR-13 | `HU-ACC-001` | `VisitServiceTest.java` | `access-control-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8087`) |
| **FR13** | KR1 | Register optional complete visitor vehicle data | BR-13 | `HU-ACC-002` | `VisitServiceTest.java` | `access-control-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8087`) |
| **FR14** | KR1 | Log visitor exit timestamp | BR-13 | `HU-ACC-003` | `VisitServiceTest.java` | `access-control-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8087`) |
| **FR15** | KR1 | Log incoming correspondence with 4-digit pickup PIN | BR-14 | `HU-ACC-004` | `CorrespondenceServiceTest.java` | `access-control-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8087`) |
| **FR16** | KR1 | View own pending correspondence & handover via PIN | BR-07, BR-14 | `HU-ACC-005` | `CorrespondenceServiceTest.java` | `access-control-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8087`) |
| **FR22** | KR1 | Pre-authorize visitor in advance & convert to Visit | BR-07 | `HU-ACC-006` | `VisitorPreauthorizationServiceTest.java` | `access-control-service` | Julián Losada | 🟡 In Progress (Phase 1 Repo `:8087`) |
| **FR17** | KR1 | Create extraordinary expense proposal with deadline & quorum | BR-15 | `HU-FIN-001` | `ExpenseProposalServiceTest.java` | `finance-approval-service` | Félix Murcia | 🟡 In Progress (Phase 1 Repo `:8088`) |
| **FR18** | KR1 | Record append-only Board votes & auto-resolve on quorum | BR-15 | `HU-FIN-002` | `ExpenseProposalServiceTest.java` | `finance-approval-service` | Félix Murcia | 🟡 In Progress (Phase 1 Repo `:8088`) |
| **FR19** | KR1 | Event-driven summary reports (requests & pending fees) | BR-16 | `HU-REP-001` | `ReportsServiceTest.java` | `reports-service` | Félix Murcia | 🟡 In Progress (Phase 1 Repo `:8089`) |

---

## 2. NFR → Validation Matrix (9 NFRs)

| NFR ID | Description | Validation Method | Tool | Owner | Status |
|--------|-------------|-------------------|------|-------|--------|
| **NFR-001** | Performance: P95 < 300ms / P99 < 500ms at 50 RPS (150 RPS peak) | Load test against critical endpoints | k6 / JMeter | Félix Murcia | 🔴 Pending |
| **NFR-002** | Availability: 99.9% production target; liveness & readiness probes | Actuator health endpoints + SLO dashboards | Spring Actuator + Grafana | Fabián Trujillo | 🟡 Partial (`/health` live in 7/10 repos) |
| **NFR-003** | Scalability: stateless horizontal scaling up to 3 instances (no Redis in MVP) | Statelessness audit + spike load test | k6 + Docker | Fabián Trujillo | 🟡 Partial (stateless JWT verified at Gateway) |
| **NFR-004** | Security: 1h JWT, 7d rotatable BCrypt refresh tokens, RBAC `:own`, OWASP, Ley 1581 | Security contract tests + SCA/SAST/Secret scan | OWASP ZAP + Trivy/Semgrep + Gitleaks | Fabián Trujillo | 🟡 Partial (`iam-service` + `api-gateway` live; STRIDE threat model documented) |
| **NFR-005** | Observability: JSON logs + `X-Correlation-Id`, RED metrics, OpenTelemetry traces | CI smoke test + tracing inspection | Logback + Prometheus + OpenTelemetry | Julián Losada | 🔴 Pending |
| **NFR-006** | Maintainability: ≥ 80% coverage (≥ 90% domain), McCabe ≤ 10, zero hexagonal leaks | CI Quality Gate + Fitness Functions | JaCoCo + Checkstyle + ArchUnit | Fabián Trujillo | 🔴 Pending |
| **NFR-007** | Portability: OCI/Docker images (Java 25), 12-Factor env config, `docker-compose` | Clean-room container startup verification | Docker Compose | Julián Losada | 🟡 Partial (`deployment.md` & `local-setup.md` reconciled) |
| **NFR-008** | Disaster Recovery: service restart < 2m (RPO 0); MySQL restore < 5m | Automated backup & restore drill | MySQL backup scripts | María Alexandra Garzón | 🔴 Pending |
| **NFR-009** | Usability & Accessibility: WCAG 2.1 AA, 360px mobile viewport, Core Web Vitals | Automated a11y + responsive browser QA | `axe-core` + Lighthouse | María Alexandra Garzón | 🟡 Partial (Stitch UI Kit & 5 screens complete) |

---

## 3. Inverse Traceability: HU → FR → Cut / Sprint (22 HUs)

| HU | Title | FR(s) | Service | Cut / Sprint | Status |
|----|-------|-------|---------|--------------|--------|
| `HU-IAM-001` | User login with role-based access | FR01 | `iam-service` | Cut 1 · Sprint 1 | 🟡 In Progress (`:8081`) |
| `HU-IAM-002` | Administrator creates staff accounts & user activation | FR20 | `iam-service` | Cut 1 · Sprint 1 | 🟡 In Progress (`:8081`) |
| `HU-PPL-001` | Register a Person and assign Unit Occupancy | FR21 | `people-service` | Cut 1 · Sprint 1 | 🔴 Pending |
| `HU-PPL-002` | Deactivate tenant occupancy upon lease termination | FR21 | `people-service` | Cut 1 · Sprint 1 | 🔴 Pending |
| `HU-UNIT-001` | Register a unit | FR02 | `units-service` | Cut 1 · Sprint 1 | 🔴 Pending |
| `HU-UNIT-002` | Edit and delete a unit | FR03 | `units-service` | Cut 1 · Sprint 1 | 🔴 Pending |
| `HU-UNIT-003` | Register commercial establishment data | FR04 | `units-service` | Cut 1 · Sprint 1 | 🔴 Pending |
| `HU-MAINT-001` | Create a maintenance request | FR05 | `maintenance-service` | Cut 2 · Sprint 3 | 🔴 Pending |
| `HU-MAINT-002` | Assign and track a maintenance request | FR06 | `maintenance-service` | Cut 2 · Sprint 3 | 🔴 Pending |
| `HU-MAINT-003` | Notify a maintenance status change | FR07 | `communications-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-BILL-001` | Generate the monthly differentiated fee | FR08, FR09 | `billing-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-BILL-002` | Check fee payment status & submit partial payments | FR10 | `billing-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-ACC-001` | Log a visitor's entry | FR12 | `access-control-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-ACC-002` | Register a visitor's vehicle | FR13 | `access-control-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-ACC-003` | Log a visitor's exit | FR14 | `access-control-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-ACC-004` | Log incoming correspondence | FR15 | `access-control-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-ACC-005` | Check pending correspondence & PIN handover | FR16 | `access-control-service` | Cut 2 · Sprint 3 | 🟡 In Progress |
| `HU-ACC-006` | Pre-authorize a visitor | FR22 | `access-control-service` | Cut 2 · Sprint 4 | 🟡 In Progress |
| `HU-COMM-001` | Publish a segmented announcement & emergency broadcast | FR11 | `communications-service` | Cut 3 · Sprint 5 | 🟡 In Progress |
| `HU-FIN-001` | Create an extraordinary expense proposal | FR17 | `finance-approval-service` | Cut 3 · Sprint 5 | 🟡 In Progress |
| `HU-FIN-002` | Approve or reject an expense proposal | FR18 | `finance-approval-service` | Cut 3 · Sprint 5 | 🟡 In Progress |
| `HU-REP-001` | View requests and pending-fee reports | FR19 | `reports-service` | Cut 3 · Sprint 5 | 🟡 In Progress |

---

## 4. Status Legend

| Status | Meaning |
|--------|---------|
| ✅ Done | Implemented, ≥ 80% test coverage passing, contract verified, merged |
| 🟡 In Progress | Phase 1 microservice repository pushed on GitHub (`https://github.com/resi-complex`); pending full TDD suite, Outbox/RabbitMQ integration & gateway E2E verification |
| 🔴 Pending | Specification, data model, and OpenAPI contract complete; code implementation not yet pushed |
| ⏸ Blocked | Waiting on external or cross-service dependency |

---

## 5. Identified Gaps (Active Tracking)

| Gap ID | Gap Type | Description | Required Action | Owner | Status |
|--------|----------|-------------|-----------------|-------|--------|
| **GAP-01** | Remaining Cut 1–2 code | `iam-service` (`:8081`) and `api-gateway` (`:8080`/`:8090`) are now implemented & pushed; remaining 3 code repositories (`units-service`, `people-service`, `maintenance-service`) are pending implementation | Implement & push `units-service` (`:8083`), `people-service` (`:8085`), and `maintenance-service` (`:8086`) on `github.com/resi-complex` | María Alexandra Garzón | 🟡 Active (7/10 repos pushed) |
| **GAP-02** | OpenAPI contracts (`07-api/`) | All 10 OpenAPI 3.0.3 contracts (`api-gateway` + 9 microservices) and `_shared.yaml` are authored, aligned to the 5 canonical roles, registered in `redocly.yaml`, and linted with 0 errors | Maintain contract-first parity on every API PR | Tech Lead & All Devs | ✅ Resolved (2026-11-21) |
| **GAP-03** | Automated test coverage | None of the 7 Phase 1 repos currently enforces the ≥ 80% JaCoCo + ArchUnit Quality Gate in CI | Add unit, Testcontainers (MySQL 8) integration, contract, and ArchUnit fitness tests per `NFR-006` | All Devs | 🟡 Active |
| **GAP-04** | Port reconciliation | Canonical ports reconciled across `05-architecture/overview.md`, `07-api/contracts/openapi/`, `09-microservices/service-catalog.md`, and `10-devops/local-setup.md` (`:8080` gateway, `:8081` iam, `:8082` communications, `:8083` units, `:8084` billing, `:8085` people, `:8086` maintenance, `:8087` access-control, `:8088` finance-approval, `:8089` reports) | Keep `resi-complex-api-gateway/application.properties` aligned to canonical `:8083` (`units`) and `:8085` (`people`) | Fabián Trujillo | ✅ Resolved in docs (2026-11-21) |

---

## Maintenance Rules

1. Every new FR/HU must be added simultaneously to `functional.md`, `user-stories.md`, `fr-to-service-mapping.md`, and this matrix.
2. When a microservice achieves passing tests (≥ 80% JaCoCo + ArchUnit) and OpenAPI validation, update its row from 🟡/🔴 to ✅.
3. Review active gaps at every Sprint Planning and before SENA milestone reviews.
