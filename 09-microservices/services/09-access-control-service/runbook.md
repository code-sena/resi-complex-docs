# Runbook — Access Control Service (`resi-complex-access-control-service`)

> Operational procedures, health monitoring, and incident troubleshooting.

---

## 1. Service Health & Verification

### Health Check Endpoint
```bash
curl -i http://localhost:8087/api/v1/visits/active
```
* **Expected Result:** HTTP `200 OK` with JSON array of active visitors.

### Swagger Documentation
Verify interactive OpenAPI documentation at:
```
http://localhost:8087/swagger-ui/index.html
```

---

## 2. Common Operational Incidents & Resolutions

### Incident A: `SchemaManagementException: missing table [visits]`
* **Cause:** Hibernate started before Flyway migrations created the tables.
* **Remediation:**
  1. Verify MySQL service is up on port 3306.
  2. Verify `access_control_db` exists:
     ```sql
     CREATE DATABASE access_control_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
     ```
  3. Ensure `spring.flyway.enabled=true` in `application.properties`.
  4. Restart the service to apply `V1__init_access.sql`.

---

### Incident B: Stale Active Visits (Visitor exit forgotten by guard)
* **Cause:** Guard forgot to mark departure when the visitor left during a shift change.
* **Remediation:**
  1. The security supervisor can force exit using the API:
     ```bash
     curl -X PATCH http://localhost:8087/api/v1/visits/{visitId}/exit
     ```
  2. A daily cron cleanup script can flag visits exceeding 24 hours for supervisor review.

---

### Incident C: Guard Console Lag under High Vehicle Flow
* **Cause:** Full table scans when searching by license plate.
* **Remediation:**
  1. Verify index `idx_visits_plate` exists:
     ```sql
     SHOW INDEX FROM visits WHERE Key_name = 'idx_visits_plate';
     ```
  2. Rebuild index if missing or fragmented:
     ```sql
     CREATE INDEX idx_visits_plate ON visits (vehicle_plate);
     ```
