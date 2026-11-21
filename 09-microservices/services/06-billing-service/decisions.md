# Technical Decisions — Billing Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-billing-service`.

---

### DEC-BILL-001: Differentiated Tariff Resolution by Unit Type
* **Status:** Accepted
* **Context:** Mixed-use complexes require higher contribution coefficients for commercial premises (`COMMERCIAL`) than for housing apartments (`RESIDENTIAL`) under Ley 675 de 2001.
* **Decision:** Resolve default tariffs in `BillingService` based on `unitType` (`250000.00` for `RESIDENTIAL`, `450000.00` for `COMMERCIAL`) when no explicit override amount is supplied, and persist `unit_type` in the `fees` record.
* **Consequences:** Guarantees regulatory compliance and prevents retroactive distortion of historical invoices.

---

### DEC-BILL-002: Idempotent Billing Period Constraint
* **Status:** Accepted
* **Context:** Administrators running monthly batch billing must be prevented from accidentally invoicing the same unit twice in the same month.
* **Decision:** Verify `existsByUnitIdAndBillingMonthAndBillingYear` before saving a new fee, returning HTTP `409 Conflict` if a record already exists.
* **Consequences:** Prevents duplicate billing and protects account statement accuracy.

---

### DEC-BILL-003: Dynamic Late Fee Calculation on Read
* **Status:** Accepted
* **Context:** Storing daily changing late fee surcharges via nightly database batch updates introduces unnecessary write contention.
* **Decision:** Compute `overdue`, `daysOverdue`, `lateFeeAmount` (0.1% daily rate on overdue days), and `totalAmount` deterministically in `BillingService` whenever a fee is queried.
* **Consequences:** Always reflects accurate up-to-the-minute arrears without background cron mutations.
