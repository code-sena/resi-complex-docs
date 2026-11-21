# Runbook — Units Service (`resi-complex-units-service`)

> Operational procedures, diagnostics, health verification, and incident management for `resi-complex-units-service`.

---

## 1. Service Overview

| Metric | Target / Value |
|---|---|
| **Port** | 8083 (HTTP) |
| **Health Check** | `GET /health` |
| **Database** | MySQL 8.0 (`units_db`) |
| **Message Broker** | RabbitMQ (`resicomplex.events` topic exchange) |
| **Log Format** | JSON structured logging (Logstash / SLF4J + Logback) |

---

## 2. Health Check & Diagnostic Verification

### Standard Health Endpoint
```bash
curl -i http://localhost:8083/health
```

Expected healthy response (`200 OK`):
```json
{
  "status": "ok",
  "timestamp": "2026-11-20T22:45:00Z",
  "dependencies": {
    "database": "ok"
  }
}
```

If MySQL connectivity fails, the endpoint returns `503 Service Unavailable`.

---

## 3. Incident Management & Common Alerts

### Incident 1: Outbox Event Backlog Alert (`UnitsOutboxPendingHigh`)
* **Symptom:** `outbox_events` table has more than 500 rows with `status = 'PENDING'` older than 2 minutes.
* **Impact:** Downstream services (`billing-service`, `access-control-service`) do not receive unit updates or new units.
* **Root Causes:**
  1. RabbitMQ broker down or network partitioned.
  2. Outbox scheduled publisher crashed or hung.
* **Mitigation Steps:**
  1. Check RabbitMQ cluster health:
     ```bash
     docker exec -it resi-complex-rabbitmq rabbitmqctl status
     ```
  2. Verify service logs for connection errors:
     ```bash
     grep -i "AmqpConnectException" /var/log/resi-complex/units-service.log
     ```
  3. Restart the outbox publisher thread or restart the container:
     ```bash
     docker restart resi-complex-units-service
     ```

### Incident 2: Soft Deletion Blocked with 409 Conflict (`UnitHasOutstandingFees`)
* **Symptom:** Administrators report inability to delete or decommission a unit.
* **Diagnosis:** Check if `billing-service` is returning active debt:
  ```bash
  curl -H "Authorization: Bearer <ADMIN_TOKEN>" http://localhost:8085/api/v1/units/{unitId}/fees/balance
  ```
* **Resolution:** In accordance with business rule `BR-05`, the administrator or accountant must settle, cancel, or credit the outstanding administration fees in `billing-service` before the unit can be soft-deleted.

### Incident 3: MySQL Connection Pool Starvation
* **Symptom:** High API latency and `CannotAcquireConnectionException` in application logs.
* **Mitigation:**
  1. Inspect active MySQL connections:
     ```sql
     SHOW PROCESSLIST;
     SELECT count(*) FROM information_schema.processlist WHERE db = 'units_db';
     ```
  2. Increase HikariCP max pool size via environment variable `SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE=30` and redeploy.

---

## 4. Database Migrations (Flyway)

Migrations are located in `src/main/resources/db/migration/`:
- `V1__create_units_and_establishments.sql`
- `V2__create_outbox_events.sql`

To verify applied migration history:
```sql
SELECT installed_rank, version, description, execution_time, success
FROM units_db.flyway_schema_history
ORDER BY installed_rank DESC;
```

---

## 5. Backup & Disaster Recovery

Run automated daily logical backup:
```bash
mysqldump -u root -p --single-transaction --quick --databases units_db > units_db_backup_$(date +%F).sql
```
Restore command:
```bash
mysql -u root -p units_db < units_db_backup_YYYY-MM-DD.sql
```
