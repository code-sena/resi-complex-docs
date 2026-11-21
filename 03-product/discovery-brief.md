# Discovery Brief — resi-complex

> **Document Status:** Complete (Phase 0 Discovery Synthesis)  
> **Language:** English (`ADR-001`; synthesizes raw Spanish research artifacts exempted under `ADR-002`)  
> **Last Updated:** 2026-11-21

---

## 1. Executive Summary

Discovery for **resi-complex** combined market/competitor technology watch (`resi-complex-tech-watch.md`), direct operational framing of small-to-medium Colombian mixed-use residential complexes (Ley 675 de 2001 — Propiedad Horizontal), and role-specific discovery instruments designed for all **5 authentication roles** (`admin-survey.md`, `board-of-trustees-survey.md`, `maintenance-survey.md`, `resident-merchant-survey.md`, `security-guard-survey.md`, and `interview-resi-complex.md`).

The central discovery finding is unambiguous: **Colombian PH administration platforms treat commercial units as an afterthought.** None of the four benchmarked competitors (ConjuntoApp, TUCO 360, Edifia, PH360) natively models mixed residential + commercial complexes with differentiated administration fee tariffs, segmented communications, and commercial-client front-desk logging out of the box. Meanwhile, small-to-medium complexes rely on notebooks, Excel spreadsheets, WhatsApp groups, and physical minutes — creating opacity for residents/merchants and zero auditable history for the Board of Trustees.

---

## 2. Research Scope & Instruments

| Instrument | Target Audience | Role (`00-governance/security-policy.md`) | File | Status |
|------------|-----------------|-------------------------------------------|------|--------|
| Technology Watch & Market Benchmark | 4 Colombian PH platforms + CAN Decision 486 IP landscape | Product / Architecture | `03-product/resi-complex-tech-watch.md` | ✅ Completed |
| Semi-structured Interview Script | Complex Administrator & Board of Trustees | `ADMINISTRATOR`, `BOARD` | `03-product/interview-resi-complex.md` | ✅ Applied & synthesized (Phase 1 Discovery) |
| Administrator Survey | Day-to-day complex administrators | `ADMINISTRATOR` | `03-product/admin-survey.md` | ✅ Applied & synthesized (Phase 1 Discovery) |
| Board of Trustees Survey | Financial oversight committee members | `BOARD` | `03-product/board-of-trustees-survey.md` | ✅ Applied & synthesized (Phase 1 Discovery) |
| Resident & Merchant Survey | Apartment occupants & commercial unit owners/tenants | `PERSON` | `03-product/resident-merchant-survey.md` | ✅ Applied & synthesized (Phase 1 Discovery) |
| Maintenance Staff Survey | Building repair & upkeep personnel | `MAINTENANCE_STAFF` | `03-product/maintenance-survey.md` | ✅ Applied & synthesized (Phase 1 Discovery) |
| Security Guard Survey | Front-desk / gatehouse (*portería*) operators | `SECURITY_GUARD` | `03-product/security-guard-survey.md` | ✅ Applied & synthesized (Phase 1 Discovery) |

---

## 3. User Personas & Core Pain Points

| Persona | Canonical Role | Primary Job-to-be-Done | Key Pain Points Observed | Mapped Requirements |
|---------|----------------|------------------------|--------------------------|---------------------|
| **Laura (Complex Administrator)** | `ADMINISTRATOR` | Keep unit registry, differentiated fees, maintenance assignments, announcements, and expense proposals under control without spreadsheet errors | Manual Excel recalculation for residential vs. commercial fees; lost maintenance requests in WhatsApp; informal expense approvals with no audit trail | `FR01–FR04`, `FR06`, `FR08–FR09`, `FR11`, `FR17`, `FR19–FR21` |
| **Roberto (Board Member)** | `BOARD` | Audit extraordinary expense proposals, cast transparent votes, and inspect portfolio arrears | Decisions buried in physical minutes or scattered emails; difficulty verifying quorum and arrears trends before approving repairs | `FR18`, `FR19` |
| **Camila / Andrés (Resident & Commercial Tenant)** | `PERSON` (`RESIDENT`, `COMMERCIAL_OWNER`, `COMMERCIAL_TENANT`) | Check unit administration fees, report maintenance issues, read relevant notices, pre-authorize visitors, and collect packages | Commercial tenants receive residential-only spam and vice versa; no visibility into repair status; packages misplaced without pickup verification | `FR05`, `FR07`, `FR10`, `FR11`, `FR16`, `FR22` |
| **Mateo (Maintenance Technician)** | `MAINTENANCE_STAFF` | Receive prioritized work orders on mobile and update repair progress | Verbal or WhatsApp instructions get lost; urgent issues (e.g., leaks, elevator outage) blend into routine requests | `FR06`, `NFR-009` |
| **Diego (Security Guard)** | `SECURITY_GUARD` | Quickly log visitor/vehicle entry-exit and front-desk parcels from a desk or mobile device | Paper logbooks are slow, illegible, impossible to query remotely, and prone to handover disputes for deliveries | `FR12–FR15`, `FR22`, `NFR-009` |

---

## 4. Hypothesis Validation Register

| Hypothesis | Verdict | Evidence / Rationale | Resulting Product Decision |
|------------|---------|----------------------|----------------------------|
| **H1:** Mixed residential + commercial complexes need native differentiated tariffs and segmented communications | **✅ Validated** | 0 of 4 Colombian PH competitors benchmarked in `resi-complex-tech-watch.md` offer native residential/commercial differentiation; Excel fails on dual-tariff tracking | `units-service` and `billing-service` elevated to **Core Domains** (`02-domain/domain-map.md`); `unit_type` frozen at fee generation (`BR-10`, `FR08–FR09`) |
| **H2:** Maintenance requests require a verified closure loop rather than unilateral technician closure | **✅ Validated** | Residents reported premature ticket closure in informal WhatsApp workflows | Added `WORK_COMPLETED` verification gate (`PENDING → ASSIGNED → IN_PROGRESS → WORK_COMPLETED ⇄ IN_PROGRESS → RESOLVED`, `FR06`, `BR-09`) |
| **H3:** Lease turnover causes data-privacy leaks if tenancy is coupled directly to user accounts | **✅ Validated** | Ley 1581 de 2012 compliance + tenant turnover requires instant revocation of unit-scoped visibility without deleting historical ledgers | Separated `person` from `unit_occupancies` (`FR21`, `BR-06`, `BR-07`) and enforced `:own` ownership checks against `ACTIVE` occupancies |
| **H4:** Front-desk parcel handovers need lightweight verification without hardware scanners | **✅ Validated** | Security guards need fast confirmation without barcode/QR hardware investment | Added 4-digit `pickup_pin` generation and verification (`FR15–FR16`, `BR-14`) plus digital visitor pre-authorization (`FR22`), deferring QR camera hardware to v2.0 |
| **H5:** Blockchain or AI document analysis adds immediate value for small-to-medium complexes | **❌ Invalidated (Deferred)** | Technology watch showed only 1/4 competitors offers document AI and 0/4 require blockchain; append-only SQL ledgers satisfy auditability | Explicitly excluded from MVP (`01-context/scope.md`); append-only `approvals` table enforces Board immutability (`FR18`, `BR-15`) |

---

## 5. Feasibility Summary (Technical, Operational, Legal)

- **Technical Feasibility:** Proven via Java 25 + Spring Boot 4.0.8 microservices (`ADR-003`), MySQL 8 dedicated per service (`ADR-004`), RabbitMQ 3.12+ with Transactional Outbox (`ADR-005`), and Spring Cloud Gateway JWT verification (`ADR-006`). Seven of ten service repositories are already running in Phase 1.
- **Operational & Economic Feasibility:** Runs on commodity Docker Compose infrastructure (`05-architecture/deployment.md`) with zero mandatory paid third-party SaaS dependencies (no SMTP, SMS, WhatsApp Business, or payment gateway fees in MVP).
- **Legal & Regulatory Feasibility:** Designed for Colombian **Ley 675 de 2001** (Propiedad Horizontal governance) and **Ley 1581 de 2012** (Habeas Data — PII isolated in `people_db`, RBAC + `:own` ownership checks, BCrypt password/token hashing). Software IP protected under **CAN Decision 486 / Copyright** (`resi-complex-tech-watch.md`).

---

## 6. Governance & Quality Agreements (DoR / DoD)

- **Definition of Ready (DoR):** Every backlog item links to an `FR` (`04-requirements/functional.md`), `BR` invariants, Given/When/Then acceptance criteria (`04-requirements/user-stories.md`), data schema (`06-data/models.md`), and OpenAPI contract (`07-api/contracts/openapi/`).
- **Definition of Done (DoD):** Clean Hexagonal implementation passing unit + Testcontainers integration tests with **≥ 80% JaCoCo coverage** (`≥ 90%` domain), **0 ArchUnit violations** (`NFR-006`), `npx @redocly/cli lint` with **0 errors**, peer code review, and updated SDD documentation (`00-governance/definition-of-done.md`).

---

## 7. Correlations

- Raw research evidence → `03-product/resi-complex-tech-watch.md`, `03-product/interview-resi-complex.md`, `03-product/*-survey.md`
- Problem framing & North Star → `03-product/problem-framing.md`
- Vision & OKRs → `03-product/vision.md`
- Delivery plan & backlog → `03-product/roadmap.md`, `03-product/product-backlog.md`
- Formal requirements → `04-requirements/functional.md`, `04-requirements/user-stories.md`
