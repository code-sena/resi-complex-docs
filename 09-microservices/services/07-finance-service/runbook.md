# Runbook — Finance Approval Service (`resi-complex-finance-approval-service`)

> Operational procedures, health monitoring, and incident troubleshooting.

---

## 1. Service Health & Verification

### Health Check Endpoint
```bash
curl -i http://localhost:8088/health
```
* **Expected Result:** HTTP `200 OK` with `{"status":"ok","service":"finance-approval-service","db":"connected"}`.

### Swagger Documentation
Verify interactive OpenAPI documentation at:
```
http://localhost:8088/swagger-ui/index.html
```

---

## 2. Common Operational Incidents & Resolutions

### Incident A: `SchemaManagementException: Schema-validation: missing table [expense_proposals]`
* **Cause:** Hibernate failed startup validation because `finance_db` tables were not created.
* **Remediation:**
  1. Verify MySQL is running on port `3306`.
  2. Ensure `finance_db` exists:
     ```sql
     CREATE DATABASE IF NOT EXISTS finance_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
     ```
  3. Restart the service so Flyway executes `V1__init_finance.sql`.

---

### Incident B: `409 Conflict` When Casting a Board Vote
* **Cause:** Either the Board member (`boardMemberId`) has already voted on the proposal, or the proposal has already reached quorum (`APPROVED` / `REJECTED`).
* **Remediation:**
  1. Inspect proposal status and existing votes:
     ```bash
     curl http://localhost:8088/api/v1/expense-proposals/{proposalId}
     ```
