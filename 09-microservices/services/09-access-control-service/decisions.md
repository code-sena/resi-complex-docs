# Technical Decisions — Access Control Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-access-control-service`.

---

### DEC-ACC-001: Front Desk Bounded Context Scope
* **Status:** Accepted
* **Context:** Visitor admissions and package deliveries both occur physically at the entrance gate (portería) and are managed by the same security guard role (`SECURITY_GUARD`).
* **Decision:** Encapsulate both `Visit` and `Correspondence` aggregates within the `access-control-service` bounded context, while maintaining independent database tables and repositories.
* **Consequences:** Reduces network overhead and simplifies frontend development for the guard touch console (`/front-desk/*`).

---

### DEC-ACC-002: Dual Visitor Categorization
* **Status:** Accepted
* **Context:** Competitor platforms treat all visitors identically. In a mixed complex, commercial clients (restaurant diners, store shoppers) have high turnover and different parking/access rules compared to residential family guests.
* **Decision:** Enforce `visit_type` (`PERSONAL_VISITOR` vs. `COMMERCIAL_CLIENT`) as a mandatory field on admission.
* **Consequences:** Enables downstream services to segment commercial parking limits and generate differentiated traffic reports without retrofitting models.

---

### DEC-ACC-003: Sub-second Active Visits Querying
* **Status:** Accepted
* **Context:** Guards need instantaneous listing of who is currently inside the complex when admitting or releasing vehicles.
* **Decision:** Create dedicated index `idx_visits_status` on `status` and expose a streamlined `/visits/active` endpoint returning only non-exited visitors.
* **Consequences:** Lookups take `< 10ms` even as the historical `visits` table grows into tens of thousands of records.
