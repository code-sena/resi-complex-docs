# Operational Runbook — `maintenance-service`

> **Criticality: P2 (Daily Operations).** Outages halt new repair reporting and technician status updates.

---

## 1. Health & Diagnostics

```bash
curl -i http://127.0.0.1:8086/health
mysql -u resi_dev -p maintenance_db -e "SELECT status, COUNT(*) FROM maintenance_requests WHERE deleted_at IS NULL GROUP BY status;"
mysql -u resi_dev -p maintenance_db -e "SELECT status, COUNT(*) FROM outbox GROUP BY status;"
```

| Symptom | Likely Cause | Immediate Action |
|---------|--------------|------------------|
| Resident not notified after status change (`FR07`) | Outbox relay stalled or RabbitMQ unreachable | Inspect `maintenance_db.outbox` rows in `PENDING`/`FAILED`; verify RabbitMQ `:5672` health & `communications-service` queue bindings |
| `422 Unprocessable Entity` on status update | Technician attempted direct jump to `RESOLVED` (`BR-09`) or invalid transition | Direct technician to set `WORK_COMPLETED`; requester completes closure via `/verify` |
| `403 Forbidden` for `MAINTENANCE_STAFF` | Technician attempting to view/update unassigned ticket (`:assigned`) | Confirm `assigned_to` matches authenticated technician UUID |

---

## 2. Recovery & Escalation

- **Target RTO:** < 2 minutes (`NFR-008`).
- **Escalation:** Service Owner (`@AlexandraVargass`) → Tech Lead (`@Trujillo7915`).
