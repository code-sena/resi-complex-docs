# Runbook — People Service (`resi-complex-people-service`)

> Operational procedures, diagnostics, health verification, and incident management for `resi-complex-people-service`.

---

## 1. Service Overview

| Metric | Target / Value |
|---|---|
| **Port** | 8084 (HTTP) |
| **Health Check** | `GET /health` |
| **Database** | MySQL 8.0 (`people_db`) |
| **Message Broker** | RabbitMQ (`resicomplex.events` topic exchange) |
| **Log Format** | JSON structured logging (Logstash / SLF4J + Logback) |

---

## 2. Health Check & Diagnostic Verification

### Standard Health Endpoint
```bash
curl -i http://localhost:8084/health
```

Expected healthy response (`200 OK`):
```json
{
  "status": "ok",
  "timestamp": "2026-11-20T23:00:00Z",
  "dependencies": {
    "database": "ok"
  }
}
```

If MySQL connectivity fails, the endpoint returns `503 Service Unavailable`.

---

## 3. Incident Management & Common Alerts

### Incident 1: Document Conflict on Registration (`DOCUMENT_ALREADY_EXISTS`)
* **Symptom:** Registration fails with HTTP 409 Conflict when creating a person profile.
* **Diagnosis:** Check if a profile with the given `document_id` already exists in `people_db.person`:
  ```sql
  SELECT id, full_name, person_type, deleted_at FROM people_db.person WHERE document_id = '1020304050';
  ```
* **Resolution:** If the person was previously soft-deleted, perform a restore operation. If the document belongs to an existing active resident, assign a new `unit_occupancies` record to link them to the second unit rather than creating a duplicate person.

### Incident 2: Unit Validation Timeout (`UnitsServiceUnavailable`)
* **Symptom:** Creating a person or occupancy fails with HTTP 502/504 or fallback error when attempting to validate `unit_id`.
* **Diagnosis:** Verify whether `units-service` is reachable on its internal port:
  ```bash
  curl -i http://localhost:8083/health
  ```
* **Resolution:** If `units-service` is restarting or network latency is elevated, check container status and DNS routing in Docker Compose.

### Incident 3: Outbox Event Backlog Alert (`PeopleOutboxPendingHigh`)
* **Symptom:** More than 500 records in `outbox_events` with `status = 'PENDING'`.
* **Impact:** Downstream services do not receive `OccupancyDeactivated` or `PersonRegistered` notifications.
* **Mitigation:**
  1. Inspect RabbitMQ broker health:
     ```bash
     docker exec -it resi-complex-rabbitmq rabbitmqctl status
     ```
  2. Inspect Spring Boot error logs for AMQP connection issues:
     ```bash
     grep -i "AmqpException" /var/log/resi-complex/people-service.log
     ```
  3. Restart the container if the scheduled outbox thread has hung:
     ```bash
     docker restart resi-complex-people-service
     ```

---

## 4. Database Migrations (Flyway)

Migrations are located in `src/main/resources/db/migration/`:
- `V1__create_people_and_occupancies.sql`
- `V2__create_outbox_events.sql`

To verify applied migration history:
```sql
SELECT installed_rank, version, description, execution_time, success
FROM people_db.flyway_schema_history
ORDER BY installed_rank DESC;
```

---

## 5. Backup & Disaster Recovery

Run automated daily logical backup:
```bash
mysqldump -u root -p --single-transaction --quick --databases people_db > people_db_backup_$(date +%F).sql
```
Restore command:
```bash
mysql -u root -p people_db < people_db_backup_YYYY-MM-DD.sql
```
