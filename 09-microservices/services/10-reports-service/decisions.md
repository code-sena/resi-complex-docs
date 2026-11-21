# Technical Decisions — Reports Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-reports-service`.

---

### DEC-REP-001: CQRS Read Projections in MySQL (`reports_db`)
* **Status:** Accepted
* **Context:** Executive dashboards require combined indicators from Billing and Maintenance without violating the Database-per-Service isolation rule (`05-architecture/overview.md`).
* **Decision:** Maintain dedicated read-projection tables (`pending_fees_view` and `requests_by_status_view`) in `reports_db`.
* **Consequences:** Dashboard endpoints respond in `< 15ms` and remain available even if upstream transactional services experience load spikes.

---

### DEC-REP-002: Critical Arrears Threshold (`overdueMonths > 2`)
* **Status:** Accepted
* **Context:** Administrators and Board members need an early filter to identify units requiring formal collection follow-up.
* **Decision:** Flag units with `overdueMonths > 2` in `criticalOverdueUnits` and `unitsWithCriticalOverdue` inside `FeeStatsResponse`.
* **Consequences:** Directly powers the `/reports` KPI card and critical arrears table in the frontend.
