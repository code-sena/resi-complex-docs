# Runbook — Communications Service (`resi-complex-communications-service`)

> Operational procedures, health monitoring, and incident troubleshooting.

---

## 1. Service Health & Verification

### Health Check Endpoint
```bash
curl -i http://localhost:8082/api/v1/announcements
```
* **Expected Result:** HTTP `200 OK` with JSON array.

### Swagger Documentation
Verify interactive OpenAPI documentation at:
```
http://localhost:8082/swagger-ui/index.html
```

---

## 2. Common Operational Incidents & Resolutions

### Incident A: `SchemaManagementException: Schema-validation: missing table [announcements]`
* **Cause:** Hibernate failed startup because the `announcements` table does not exist in `communications_db`.
* **Remediation:**
  1. Verify MySQL is running on port 3306.
  2. Verify Flyway ran migration `V1__init_communications.sql`.
  3. Ensure database exists:
     ```sql
     CREATE DATABASE communications_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
     ```
  4. Restart the Spring Boot service to trigger Flyway.

---

### Incident B: `FlywayValidateException: Validate failed: Migrations have failed validation`
* **Cause:** An already-applied migration script was modified locally, causing a checksum mismatch.
* **Remediation:**
  1. In local development:
     ```sql
     DROP DATABASE communications_db;
     CREATE DATABASE communications_db;
     ```
  2. Re-run `./mvnw spring-boot:run` to reapply migrations cleanly from scratch.

---

### Incident C: High Latency on Announcements Feed
* **Cause:** Missing index on `target_scope` or large volume of announcements without pagination.
* **Remediation:**
  1. Verify indexes exist:
     ```sql
     SHOW INDEX FROM announcements;
     ```
  2. Ensure `idx_announcements_scope` and `idx_announcements_deleted` are present.

---

## 3. Maintenance Procedures

### Applying a New Migration Script
1. Create `src/main/resources/db/migration/V2__add_priority_column.sql`.
2. Commit the file to Git.
3. On deployment, Spring Boot executes the migration before accepting HTTP traffic.
