# Operational Runbook — `api-gateway`

> **Criticality: P0 (Edge Perimeter).** If `api-gateway` is unreachable, all external web/API traffic is blocked.

---

## 1. Health & Diagnostics

```bash
# Check gateway liveness and upstream map (use :8090 if running bare-metal Windows fallback)
curl -i http://127.0.0.1:8080/health

# Check port binding on Windows / Linux
netstat -ano | findstr :8080
```

| Symptom | Likely Cause | Immediate Action |
|---------|--------------|------------------|
| `401 Unauthorized` across all protected routes | `JWT_SECRET` mismatch between `api-gateway` and `iam-service`, or clock skew | Verify `JWT_SECRET` matches `iam-service` env var; check container NTP sync; inspect `X-Correlation-Id` logs |
| `429 Too Many Requests` spike | Brute-force client or misconfigured polling UI | Identify offending IP in JSON logs; temporarily tune `GATEWAY_RPM_LIMIT` or block IP at reverse proxy |
| `502 Bad Gateway` / `503 Service Unavailable` on one prefix | Target downstream microservice crashed or wrong `*_SERVICE_URL` port | Check target service `/health` (`:8081`–`:8089`); confirm canonical port mapping in `application.properties` |
| Gateway fails to start on Windows local dev | Port `8080` occupied by local Windows service | Run with `SERVER_PORT=8090` (default fallback in `application.properties`) |

---

## 2. Recovery & Rollback

1. **Restart container:** `docker compose restart api-gateway` (target RTO < 30 s, stateless).
2. **Verify smoke route:** `curl http://127.0.0.1:8080/health` returns `200 OK` and `POST /api/v1/auth/login` reaches `iam-service` (`:8081`).
3. **Escalation:** Tech Lead (`@Trujillo7915`) → Security/DevOps Lead (`@julianlp-hub`).
