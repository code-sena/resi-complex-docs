# Technical Decisions — Units Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-units-service`.

---

### DEC-UNIT-001: Strict 1:1 Cardinality for Commercial Establishments
* **Status:** Accepted
* **Context:** The ubiquitous language in `01-context/glossary.md` specifies that a commercial unit has an establishment, but they are not the same entity. A commercial unit represents physical square meters and nomenclature; an establishment represents the commercial activity and business hours.
* **Decision:** Implement a distinct `commercial_establishments` table linked to `units` via a unique foreign key constraint `uq_comm_est_unit_id (unit_id)`. Additionally, the domain service enforces that only units with `unit_type = 'COMMERCIAL'` may hold an establishment (`AGGR-INV-001` / `BR-04`).
* **Consequences:** Clear boundary separation, strong relational enforcement in MySQL, and prevention of orphaned or multiple establishments per commercial premise.

---

### DEC-UNIT-002: Soft Delete Guarded by Financial Invariant (`BR-05`)
* **Status:** Accepted
* **Context:** Physical condominium units cannot simply vanish from history, as their historical fees and occupancy records must remain auditable. Furthermore, Colombian condominium regulations require settling outstanding debts before administrative decommissioning.
* **Decision:** Implement soft deletion via `deleted_at TIMESTAMP NULL`. Before setting `deleted_at`, `units-service` queries `billing-service` (`GET /api/v1/units/{id}/fees/balance`). If active or overdue fees exist (`PENDING`, `PARTIALLY_PAID`, `OVERDUE`), the request is rejected with `409 Conflict`.
* **Consequences:** Preservation of historical records across all services; immediate protection against wiping units with unpaid administration balances.

---

### DEC-UNIT-003: Transactional Outbox Pattern for Inter-Service Synchronization
* **Status:** Accepted
* **Context:** Direct dual-write (writing to MySQL and publishing to RabbitMQ in the same method) causes dual-write inconsistencies if the broker fails or network drops.
* **Decision:** Persist events into an `outbox_events` table inside the same local database transaction that commits unit modifications. A decoupled worker processes pending records and publishes them to the RabbitMQ exchange.
* **Consequences:** At-least-once delivery guaranteed; consumers (`billing-service`, `access-control-service`) must process events idempotently.

---

### DEC-UNIT-004: Schema Validation with Hibernate `validate` Mode
* **Status:** Accepted
* **Context:** Production environments require strict governance over database alterations. Automatic DDL generation (`ddl-auto=update`) can cause schema drift and data corruption.
* **Decision:** Configure `spring.jpa.hibernate.ddl-auto=validate` and manage all database migrations through versioned Flyway scripts (`V1__init_units_schema.sql`).
* **Consequences:** Spring Boot application startup fails immediately if Java entities diverge from the Flyway-managed MySQL schema.
