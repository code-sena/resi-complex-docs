# Technical Decisions — People Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-people-service`.

---

### DEC-PPL-001: Separation of Person Identity from Unit Occupancy
* **Status:** Accepted
* **Context:** In condominium management, a person (resident or commercial owner) may relocate to another apartment, own multiple units, or terminate a lease while their administrative and payment history must remain preserved.
* **Decision:** Decouple the person's identity into a `person` table and manage their physical residency or ownership in a separate `unit_occupancies` table with start/end validity dates and status (`ACTIVE` / `INACTIVE`).
* **Consequences:** Highly flexible model supporting historical tenancy tracking, co-ownership, and multi-unit leases without duplicate person profiles.

---

### DEC-PPL-002: Immediate Privilege Revocation with Audit Retention (`HU-PPL-002`)
* **Status:** Accepted
* **Context:** When an occupant terminates a lease or moves out, their ability to pre-authorize visitors or collect packages must cease immediately to protect physical building security. However, legal regulations require retaining past records.
* **Decision:** Deactivation updates `unit_occupancies.status = 'INACTIVE'` and sets `end_date = CURRENT_DATE` without performing SQL `DELETE`. The service emits `OccupancyDeactivated`, signaling `iam-service` and `access-control-service` to revoke `:own` claims immediately.
* **Consequences:** Security breach prevention and instant access cutoff while ensuring full compliance with audit and legal discovery requirements.

---

### DEC-PPL-003: Logical Referencing Across Database Boundaries
* **Status:** Accepted
* **Context:** SENA ADSO architecture rules mandate strict Database per Service isolation. Cross-database foreign keys between `people_db` and `units_db` violate architectural boundaries.
* **Decision:** Store `unit_id` as a logical `CHAR(36)` reference in `people_db`. Validation of unit existence is carried out synchronously via HTTP against `units-service` (`GET /api/v1/units/{id}`) during registration.
* **Consequences:** Total autonomy of each service database; schemas can evolve independently.

---

### DEC-PPL-004: Transactional Outbox for Distributed Consistency
* **Status:** Accepted
* **Context:** Direct dual-writing to MySQL and RabbitMQ introduces the risk of phantom states if either system fails mid-operation.
* **Decision:** Persist event payloads into `outbox_events` inside the local ACID transaction. An asynchronous worker dispatches events to RabbitMQ with at-least-once delivery guarantees.
* **Consequences:** Zero event loss and complete decoupling between HTTP response time and message broker latency.
