# Cross-Cutting Concerns — resi-complex

> **Document Status:** Complete — System-wide standards enforced across `api-gateway` and all 9 microservices  
> **Aligned with:** `00-governance/security-policy.md`, `04-requirements/non-functional.md`, `05-architecture/overview.md`, `ADR-005`, `ADR-006`, `07-api/`  
> **Last Updated:** 2026-11-21

---

## 1. Authentication, RBAC & Ownership Propagation (`ADR-006`, `P5`)

1. **Edge Verification (`api-gateway`):**
   - Validates `Authorization: Bearer <jwt>` signature (`HS256` 256-bit secret in MVP; `RS256`/JWKS upgrade path) and `exp` timestamp (1-hour lifetime).
   - Strips untrusted client-supplied `X-User-*` headers and injects trusted downstream headers:
     - `X-User-Id`: Authenticated user UUID (`JWT.sub`)
     - `X-User-Roles`: Canonical role (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`)
     - `X-User-Permissions`: Comma-separated RBAC permissions
2. **Service-Layer Ownership Guards (`:own` / `:assigned`):**
   - `PERSON` users may access only records linked to an `ACTIVE` occupancy in `people-service` (`unitIds` claim / occupancy verification).
   - `MAINTENANCE_STAFF` users may view and update only requests where `assigned_to == X-User-Id`.
   - Enforced inside the **Application / Use Case layer**, never exclusively in controllers or UI.

---

## 2. Standard Error Handling (RFC 7807 / `_shared.yaml`)

Every microservice implements a centralized `@RestControllerAdvice` mapping domain and validation exceptions to the unified `ErrorResponse` schema (`07-api/contracts/openapi/_shared.yaml`):

```json
{
  "error": "DOMAIN_INVARIANT_VIOLATION",
  "message": "Cannot delete unit with active or overdue administration fees.",
  "details": [
    { "field": "unitId", "message": "Unpaid fee exists in billing-service (BR-05)" }
  ],
  "traceId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Mandatory Rules:**
- Zero internal stack traces, SQL fragments, or class names leaked in HTTP responses (`NFR-004`).
- Deterministic status mapping: `400` validation, `401` invalid/expired token, `403` RBAC/ownership violation, `404` missing entity, `409` uniqueness/state conflict, `422` domain invariant violation, `423` locked account, `429` rate limit exceeded, `503` downstream unavailable.

---

## 3. Structured Logging, Metrics & Distributed Tracing (`NFR-005`)

| Pillar | Standard | Configuration |
|--------|----------|---------------|
| **Structured JSON Logs** | Logback + `logstash-logback-encoder` | Mandatory fields: `timestamp` (UTC ISO-8601), `level`, `service`, `correlationId`, `userId` (UUID only — zero PII/passwords/tokens), `event`, `durationMs` |
| **Correlation Propagation** | `X-Correlation-Id` HTTP header + AMQP message header | Generated at `api-gateway` if absent; stored in SLF4J MDC; forwarded on all synchronous REST calls and RabbitMQ Outbox envelopes |
| **Metrics (RED)** | Spring Boot Actuator + Micrometer → Prometheus (`/actuator/prometheus`) | Request rate, error rate (4xx/5xx), latency histograms (P95/P99), JVM heap, HikariCP pool, RabbitMQ queue depth, Outbox pending lag |
| **Distributed Tracing** | OpenTelemetry Java Agent / Micrometer Tracing → Jaeger | W3C `traceparent` propagation across Gateway → Service → MySQL → RabbitMQ |

*Prohibited:* `System.out.println` or `console.log` in production code.

---

## 4. Health Checks & Resilience (`NFR-002`, `P3`)

- **Liveness Probe (`GET /health`):** Public, unauthenticated endpoint returning HTTP `200` when the JVM process is responsive (`status: "ok"`).
- **Readiness Probe (`GET /health/ready` / `/actuator/health/readiness`):** Verifies connectivity to dedicated MySQL 8 schema and RabbitMQ broker; returns `503` if degraded so traffic stops routing to unhealthy instances.
- **Resilience Policies (Resilience4j):**
  - *Timeouts:* Strict connect (`2s`) and read (`5s`) timeouts on inter-service HTTP clients (e.g., `units-service` → `billing-service` unpaid-fee check).
  - *Retry with Exponential Backoff & Jitter:* Up to 3 attempts on transient `5xx`/network errors for idempotent reads and Outbox publishing.
  - *Circuit Breaker:* Opens at ≥ 50% failure rate over a sliding window of 20 calls; half-open probe after 30s.
  - *Rate Limiting:* Centralized at `api-gateway` (`240 RPM` default per client IP; stricter throttling on `/api/v1/auth/login`).

---

## 5. Event Delivery & Idempotency (`ADR-005`, `06-data/models.md`)

- **Transactional Outbox (`outbox`):** Aggregate state change + outbox row commit atomically in MySQL 8; relay worker publishes to RabbitMQ exchange `resicomplex.events` with exponential retry.
- **Idempotent Consumers (`processed_events`):** Every consumer inserts `event_id` into `processed_events` in the same local DB transaction as its projection/notification write; duplicate deliveries are acknowledged and ignored safely.
- **Event Envelope Standard:** `eventId` (UUID), `eventType`, `version` (`"1.0"`), `occurredAt` (UTC ISO-8601), `correlationId`, `aggregateId`, `payload`.

---

## 6. Configuration & Secret Management (12-Factor)

- Validate required env vars at startup (**Fail Fast**).
- Never commit `.env` files or secrets to Git (`Gitleaks` secret scanning in CI per `NFR-004`).
- Maintain `.env.example` in every service repository with safe synthetic defaults only.

---

## 7. Correlations

- Architecture Principles → `05-architecture/overview.md` §5
- API Guidelines & Error Schema → `07-api/guidelines.md`, `07-api/contracts/openapi/_shared.yaml`
- Observability Runbook → `13-operations/observability.md`
- Threat Model → `05-architecture/security-threat-model.md`
