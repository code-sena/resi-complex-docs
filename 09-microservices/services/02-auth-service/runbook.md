# Operational Runbook — `iam-service`

> **Criticality: P0 (Identity Authority).** A P0 outage in `iam-service` prevents all logins, account activations, and refresh-token renewals across `resi-complex`.

---

## 1. Health & Diagnostics

```bash
# 1. Check service liveness & iam_db connectivity
curl -i http://127.0.0.1:8081/health

# 2. Verify MySQL 8 iam_db status and Flyway migration history
mysql -u resi_dev -p iam_db -e "SELECT version, description, success FROM flyway_schema_history;"
```

| Symptom | Likely Cause | Immediate Action |
|---------|--------------|------------------|
| Startup crash (`Schema-validation: missing table`) | `iam_db` not created or Flyway migration failed under `ddl-auto=validate` | Create `iam_db`, inspect Flyway logs, verify `DB_URL` credentials |
| `423 Locked` reported by legitimate user | 5 consecutive failed logins (`BR-02`, 15-minute lockout) | Confirm user identity; wait 15 min or reset lockout via audited Admin action |
| Mass `401 Unauthorized` on `/api/v1/auth/refresh` | Token family revoked due to replay attack (`BR-03`) or rotated `JWT_SECRET` | Inspect `iam_db.refresh_tokens` revocation logs & `X-Correlation-Id`; coordinate synchronized `JWT_SECRET` rotation with `api-gateway` |
| High latency on `/api/v1/auth/login` | BCrypt CPU saturation or MySQL connection pool exhaustion | Inspect JVM CPU metrics and HikariCP active connections (`NFR-005`) |

---

## 2. Emergency Secret Rotation (`JWT_SECRET`)

1. Generate a new cryptographically random ≥ 256-bit secret.
2. Update `JWT_SECRET` simultaneously in `iam-service` and `api-gateway` secret stores.
3. Rolling-restart `iam-service` and `api-gateway` (active access tokens expire within ≤ 1 hour; refresh tokens remain valid unless revoked in `iam_db.refresh_tokens`).

---

## 3. Recovery & Escalation

- **Target RTO:** < 2 minutes (`NFR-008`) · **Target RPO:** 0 (MySQL binary-log backups).
- **Escalation:** Tech Lead & IAM Owner (`@Trujillo7915`) → Security Lead (`@julianlp-hub`).
