# ADR-004 — Database per Service Pattern (MySQL 8.0+)

| Field | Value |
| :--- | :--- |
| **ID** | ADR-004 |
| **Date** | 2026-09-17 *(reconciled 2026-11-21)* |
| **Status** | Accepted |
| **Authors** | Fabián Trujillo — Tech Lead & Project Leader |
| **Reviewers** | Alexandra Garzón, Félix Murcia, Julián Losada — Development Team |
| **Supersedes / Related to** | Complements `ADR-003` and `06-data/models.md` |

---

## Context

In a microservices architecture, allowing multiple services to share a single database schema creates tight data-layer coupling, breaks independent schema evolution, and enables developers to bypass domain invariants via cross-context SQL `JOIN`s.

**Known Constraints:**
- **MySQL 8.0+** (`InnoDB`, `utf8mb4_unicode_ci`) is the confirmed relational database engine for `resi-complex` (`01-context/scope.md`).
- **Colombian Ley 1581 de 2012 (Habeas Data)** requires strict isolation of personal identifiable information (`person`, `unit_occupancies`) inside `people-service`.
- Academic/engineering standard mandates **Flyway** forward-only versioned migrations (`V{n}__*.sql`) with `spring.jpa.hibernate.ddl-auto=validate` (no automatic schema mutation by Hibernate).

---

## Decision

**We decided:** To enforce the **Database per Service** pattern using **MySQL 8.0+**. Each of the 9 microservices owns a dedicated logical database schema — direct cross-database SQL access is strictly prohibited:

| Service | Canonical Database | Port | Core Tables |
| :--- | :--- | :--- | :--- |
| `iam-service` | `iam_db` | `8081` | `users`, `refresh_tokens`, `outbox` |
| `communications-service` | `communications_db` | `8082` | `announcements`, `outbox`, `processed_events` |
| `units-service` | `units_db` | `8083` | `units`, `commercial_establishments`, `outbox` |
| `billing-service` | `billing_db` | `8084` | `administration_fees`, `payments`, `outbox`, `processed_events` |
| `people-service` | `people_db` | `8085` | `person`, `unit_occupancies`, `outbox` |
| `maintenance-service` | `maintenance_db` | `8086` | `maintenance_requests`, `outbox` |
| `access-control-service` | `access_control_db` | `8087` | `visits`, `visitor_vehicles`, `visitor_preauthorizations`, `correspondence`, `outbox`, `processed_events` |
| `finance-approval-service` | `finance_approval_db` *(Phase-1 local alias: `finance_db`)* | `8088` | `expense_proposals`, `approvals`, `outbox` |
| `reports-service` | `reports_db` | `8089` | `requests_by_status_view`, `pending_fees_view`, `processed_events` |

Cross-service entity references are stored strictly as **logical `CHAR(36)` UUID references** validated via REST APIs or Domain Events — never via cross-database SQL `FOREIGN KEY` constraints.

**Justification:**
1. **Autonomous Schema Evolution:** Each service manages its own Flyway migrations and Expand-and-Contract rollouts independently.
2. **Domain Boundary Enforcement:** Prevents accidental SQL joins across Bounded Contexts; all invariants stay inside the owning Aggregate.
3. **Privacy & Security Isolation:** Isolates resident PII (`people_db`) and credential/token hashes (`iam_db`) from operational and reporting services.
4. **ACID Transactional Integrity:** Relational guarantees protect fee/payment ledgers, append-only Board votes (`approvals`), and atomic Aggregate + `outbox` commits.

---

## Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: Database per Service on MySQL 8 (CHOSEN)** | Full domain isolation; independent Flyway migrations; ACID transactional outbox; Ley 1581 PII isolation | No cross-service SQL joins; requires CQRS read models (`reports_db`) and eventual consistency | — *(Chosen)* |
| **Option B: Shared Single Database** | Simple cross-table joins; single migration pipeline | Destroys microservice autonomy; schema changes cause cascading failures | Anti-pattern in microservices; breaks `ADR-003` |
| **Option C: Polyglot Persistence (PostgreSQL + MongoDB + Redis)** | Specialized engines per workload | Unnecessary operational complexity for single-complex MVP; contradicts SENA stack lock (`MySQL 8`) | Relational MySQL 8 covers all 18 domain/read tables cleanly in MVP |

---

## Consequences

**Positive:**
- Independent Flyway migrations and `ddl-auto=validate` enforcement in every repository.
- Zero cross-schema coupling; schema refactors in `units_db` cannot break `billing_db`.
- Local development can run a single MySQL 8 Docker container hosting 9 isolated logical schemas (`*_db`) with separate per-service credentials, scaling seamlessly to independent managed DB instances in production.

**Negative / Trade-offs:**
- Cross-domain analytics require event-driven read projections in `reports_db` (`FR19`).
- Referential integrity across service boundaries is enforced at application/event level rather than DB foreign keys.

---

## Risks

| Risk | Probability | Impact | Mitigation |
| :--- | :--- | :--- | :--- |
| Developer accidentally connects service A to service B's schema | Low | High | Separate MySQL credentials per database; ArchUnit & PR review checks |
| Breaking schema change causes downtime | Medium | Medium | Enforce forward-only Flyway scripts + 4-step Expand & Contract (`06-data/models.md`) |

---

## References

- Complete DDL, Data Dictionaries & ER Diagram → `06-data/models.md`
- Microservices Pattern Guide → `05-architecture/pattern-guide.md`
- Local Database Topology → `05-architecture/deployment.md`, `10-devops/local-setup.md`
