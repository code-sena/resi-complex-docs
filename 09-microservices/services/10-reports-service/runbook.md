# Runbook — Reports Service (`resi-complex-reports-service`)

> Operational procedures, health monitoring, and incident troubleshooting.

---

## 1. Service Health & Verification

### Health Check Endpoint
```bash
curl -i http://localhost:8089/health
```
* **Expected Result:** HTTP `200 OK` with `{"status":"ok","service":"reports-service","db":"connected"}`.

### Swagger Documentation
Verify interactive OpenAPI documentation at:
```
http://localhost:8089/swagger-ui/index.html
```

---

## 2. Common Operational Incidents & Resolutions

### Incident A: `SchemaManagementException: Schema-validation: missing table [requests_by_status_view]`
* **Cause:** Hibernate failed startup validation because `reports_db` tables were not created.
* **Remediation:**
  1. Verify MySQL is running on port `3306`.
  2. Ensure `reports_db` exists:
     ```sql
     CREATE DATABASE IF NOT EXISTS reports_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
     ```
  3. Restart the service so Flyway executes `V1__init_reports.sql`.
