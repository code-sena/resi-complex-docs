# Product Backlog — resi-complex

> **Document Status:** Complete — Canonical Prioritized Backlog (`22 FRs · 22 HUs · 9 Epics`)  
> **Synchronized with:** `04-requirements/functional.md`, `04-requirements/user-stories.md`, `04-requirements/traceability-matrix.md`  
> **Last Updated:** 2026-11-21

---

## 1. Backlog Summary by Epic

| Epic ID | Bounded Context / Service | FRs | HUs | Total SP | Delivery Cut | Owner |
|---------|---------------------------|-----|-----|----------|--------------|-------|
| **EP-01** | IAM (`iam-service` + `api-gateway`) | `FR01`, `FR20` | 2 | 13 | Cut 1 (Sprint 1) | Fabián Trujillo |
| **EP-02** | Units Management (`units-service`) — *Core* | `FR02`, `FR03`, `FR04` | 3 | 13 | Cut 1 (Sprint 1) | María Alexandra Garzón |
| **EP-03** | People Management (`people-service`) | `FR21` | 2 | 10 | Cut 1 (Sprint 1–2) | María Alexandra Garzón |
| **EP-04** | Maintenance (`maintenance-service`) | `FR05`, `FR06` | 2 | 13 | Cut 2 (Sprint 3) | María Alexandra Garzón |
| **EP-05** | Billing (`billing-service`) — *Core* | `FR08`, `FR09`, `FR10` | 2 | 13 | Cut 2 (Sprint 3) | Félix Murcia |
| **EP-06** | Access Control (`access-control-service`) | `FR12`–`FR16`, `FR22` | 6 | 24 | Cut 2 (Sprint 3–4) | Julián Losada |
| **EP-07** | Communications (`communications-service`) | `FR07`, `FR11` | 2 | 8 | Cut 2–3 (Sprint 3 & 5) | Julián Losada |
| **EP-08** | Financial Approval (`finance-approval-service`) | `FR17`, `FR18` | 2 | 13 | Cut 3 (Sprint 5) | Félix Murcia |
| **EP-09** | Reports (`reports-service`) | `FR19` | 1 | 8 | Cut 3 (Sprint 5) | Félix Murcia |
| **Total** | **9 Microservices + API Gateway** | **22 FRs** | **22 HUs** | **115 SP** | **3 Cuts** | **4 Devs** |

---

## 2. Prioritized Product Backlog (Ordered by MVP Value & Dependency)

**MoSCoW Key:** `M` = Must Have (MVP mandatory) · `S` = Should Have (MVP target) · `C` = Could Have (v2.0+)  
**Status Key:** ✅ Done · 🟡 In Progress (Phase 1 Repo Live) · 🔴 Ready (Spec & OpenAPI complete, code pending)

| Rank | HU ID | FR(s) | Story Title | Service | MoSCoW | SP | Cut | Dependencies | Status |
|------|-------|-------|-------------|---------|:------:|:--:|-----|--------------|--------|
| 1 | `HU-IAM-001` | `FR01` | User login with RBAC (5 roles), 1h JWT & 7d refresh rotation | `iam-service` | **M** | 8 | Cut 1 | — | 🟡 In Progress (`:8081`) |
| 2 | `HU-IAM-002` | `FR20` | Admin staff account provisioning & user activation flow | `iam-service` | **M** | 5 | Cut 1 | `HU-IAM-001` | 🟡 In Progress (`:8081`) |
| 3 | `HU-UNIT-001` | `FR02` | Register physical unit (`RESIDENTIAL` or `COMMERCIAL`) | `units-service` | **M** | 5 | Cut 1 | `HU-IAM-001` | 🔴 Ready |
| 4 | `HU-UNIT-002` | `FR03` | Edit & soft-delete unit (blocked if unpaid fees exist — `BR-05`) | `units-service` | **M** | 5 | Cut 1 | `HU-UNIT-001` | 🔴 Ready |
| 5 | `HU-UNIT-003` | `FR04` | Register 1:1 commercial establishment data (`BR-04`) | `units-service` | **M** | 3 | Cut 1 | `HU-UNIT-001` | 🔴 Ready |
| 6 | `HU-PPL-001` | `FR21` | Register Person (`RESIDENT` / `COMMERCIAL_*`) & Unit Occupancy | `people-service` | **M** | 5 | Cut 1 | `HU-UNIT-001` | 🔴 Ready |
| 7 | `HU-PPL-002` | `FR21` | Deactivate tenant occupancy upon lease termination (`BR-07`) | `people-service` | **M** | 5 | Cut 1 | `HU-PPL-001` | 🔴 Ready |
| 8 | `HU-BILL-001` | `FR08`, `FR09` | Generate monthly differentiated administration fee per unit (`BR-10`) | `billing-service` | **M** | 8 | Cut 2 | `HU-UNIT-001` | 🟡 In Progress (`:8084`) |
| 9 | `HU-BILL-002` | `FR10` | Check own unit fee status, submit vouchers & partial payments (`BR-11`, `BR-12`) | `billing-service` | **M** | 5 | Cut 2 | `HU-BILL-001`, `HU-PPL-001` | 🟡 In Progress (`:8084`) |
| 10 | `HU-MAINT-001` | `FR05` | Create maintenance request (`PRIVATE_UNIT` or `COMMON_AREA`, `BR-08`) | `maintenance-service` | **M** | 5 | Cut 2 | `HU-PPL-001`, `HU-UNIT-001` | 🔴 Ready |
| 11 | `HU-MAINT-002` | `FR06` | Assign, progress, verify (`WORK_COMPLETED` → `RESOLVED`) or reject request (`BR-09`) | `maintenance-service` | **M** | 8 | Cut 2 | `HU-MAINT-001` | 🔴 Ready |
| 12 | `HU-MAINT-003` | `FR07` | Notify requester in-app on maintenance status change | `communications-service` | **M** | 3 | Cut 2 | `HU-MAINT-002`, RabbitMQ | 🟡 In Progress (`:8082`) |
| 13 | `HU-ACC-001` | `FR12` | Log visitor entry (`PERSONAL` / `COMMERCIAL_CLIENT`, `BR-13`) | `access-control-service` | **M** | 5 | Cut 2 | `HU-UNIT-001` | 🟡 In Progress (`:8087`) |
| 14 | `HU-ACC-002` | `FR13` | Register optional visitor vehicle data with visit | `access-control-service` | **M** | 3 | Cut 2 | `HU-ACC-001` | 🟡 In Progress (`:8087`) |
| 15 | `HU-ACC-003` | `FR14` | Log visitor exit timestamp | `access-control-service` | **M** | 3 | Cut 2 | `HU-ACC-001` | 🟡 In Progress (`:8087`) |
| 16 | `HU-ACC-004` | `FR15` | Log incoming correspondence with 4-digit pickup PIN (`BR-14`) | `access-control-service` | **M** | 5 | Cut 2 | `HU-UNIT-001` | 🟡 In Progress (`:8087`) |
| 17 | `HU-ACC-005` | `FR16` | View own pending correspondence & confirm PIN handover | `access-control-service` | **M** | 3 | Cut 2 | `HU-ACC-004`, `HU-PPL-001` | 🟡 In Progress (`:8087`) |
| 18 | `HU-ACC-006` | `FR22` | Pre-authorize visitor in advance & convert to Visit on entry | `access-control-service` | **S** | 5 | Cut 2 | `HU-ACC-001`, `HU-PPL-001` | 🟡 In Progress (`:8087`) |
| 19 | `HU-COMM-001` | `FR11` | Publish segmented announcements (`ALL` / `RESIDENTIAL_ONLY` / `COMMERCIAL_ONLY`) & emergency broadcasts | `communications-service` | **M** | 5 | Cut 3 | `HU-IAM-001` | 🟡 In Progress (`:8082`) |
| 20 | `HU-FIN-001` | `FR17` | Create extraordinary expense proposal with quorum & deadline (`BR-15`) | `finance-approval-service` | **M** | 5 | Cut 3 | `HU-IAM-001` | 🟡 In Progress (`:8088`) |
| 21 | `HU-FIN-002` | `FR18` | Record append-only Board votes & auto-resolve on quorum (`BR-15`) | `finance-approval-service` | **M** | 8 | Cut 3 | `HU-FIN-001` | 🟡 In Progress (`:8088`) |
| 22 | `HU-REP-001` | `FR19` | Event-driven CQRS executive reports (requests by status & fee arrears, `BR-16`) | `reports-service` | **M** | 8 | Cut 3 | RabbitMQ + Operational events | 🟡 In Progress (`:8089`) |

---

## 3. Technical Enabler Backlog (Cross-Cutting)

| Enabler ID | Description | Owner | Linked ADR / NFR | Status |
|------------|-------------|-------|------------------|--------|
| **TECH-01** | API Gateway centralized JWT verification, `X-User-*` headers, CORS & rate limiting | Fabián Trujillo | `ADR-006`, `NFR-004` | 🟡 Phase 1 Repo Live (`:8080` / `:8090`) |
| **TECH-02** | RabbitMQ Topic Exchange (`resicomplex.events`) + Transactional Outbox & Idempotent Consumers | Full Team | `ADR-005`, `06-data/models.md` | 🔴 Integration pending across repos |
| **TECH-03** | CI Quality Gate: JaCoCo ≥ 80% (`≥ 90%` domain), ArchUnit hexagonal fitness tests, Checkstyle | Full Team | `NFR-006`, `11-quality/` | 🔴 Pending CI rollout |
| **TECH-04** | Unified local `docker-compose.yml` + OpenTelemetry/Prometheus observability stack | Julián Losada | `ADR-004`, `NFR-005`, `NFR-007` | 🔴 Pending final composition |
| **TECH-05** | Responsive Web UI integration against OpenAPI contracts + `axe-core` WCAG 2.1 AA audit | María Alexandra Garzón / Full Team | `NFR-009`, `12-ux-ui/` | 🟡 Stitch UI Kit & 5 screens complete |

---

## 4. Correlations

- Full BDD acceptance criteria (Given/When/Then) → `04-requirements/user-stories.md`
- Functional & Business Rules catalog → `04-requirements/functional.md`
- End-to-end Traceability Matrix → `04-requirements/traceability-matrix.md`
- Horizon roadmap → `03-product/roadmap.md`
