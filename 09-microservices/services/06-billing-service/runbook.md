# Runbook — Billing Service (`resi-complex-billing-service`)

> Operational procedures, health monitoring, and incident troubleshooting.

---

## 1. Service Health & Verification

### Health Check Endpoint
```bash
curl -i http://localhost:8084/health
```
* **Expected Result:** HTTP `200 OK` with `{"status":"ok","service":"billing-service","db":"connected"}`.

### Swagger Documentation
Verify interactive OpenAPI documentation at:
```
http://localhost:8084/swagger-ui/index.html
```

---

## 2. Common Operational Incidents & Resolutions

### Incident A: `SchemaManagementException: Schema-validation: missing table [fees]`
* **Cause:** Hibernate failed startup validation because `fees` table is missing in `billing_db`.
* **Remediation:**
  1. Verify MySQL is running on port `3306`.
  2. Ensure `billing_db` exists:
     ```sql
     CREATE DATABASE IF NOT EXISTS billing_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
     ```
  3. Restart the service so Flyway executes `V1__init_billing.sql`.

---

### Incident B: `409 Conflict` When Generating Monthly Fee
* **Cause:** A fee for the target `(unitId, billingMonth, billingYear)` already exists in `billing_db`.
* **Remediation:**
  1. Query the unit's existing fees:
     ```bash
     curl http://localhost:8084/api/v1/fees/unit/{unitId}
     ```
  2. Verify the existing fee's ID and status instead of generating a duplicate invoice.
