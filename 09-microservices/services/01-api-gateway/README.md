# API Gateway (`resi-complex-api-gateway`)

> **Single Edge Entry Point & Security Perimeter** of `resi-complex` (`ADR-006`). Centralizes JWT signature/expiration verification, trusted `X-User-*` identity header propagation, CORS, per-client rate limiting, correlation ID injection, and reverse-proxy routing across all 9 internal microservices.

---

## Location in the Architecture

| Field | Value |
|---|---|
| Number in catalog | 01 |
| Service Name | `resi-complex-api-gateway` (`api-gateway`) |
| Canonical port | `8080` *(local Windows fallback: `8090` via `SERVER_PORT=8090`)* |
| Repository | `https://github.com/resi-complex/resi-complex-api-gateway.git` |
| DB engine | None (stateless; in-memory sliding-window rate limiter in MVP, Redis optional at multi-replica scale) |
| Architecture layer | Hexagonal Architecture (`GatewayRoutingUseCasePort`, `JwtVerifierPort`, `Hs256GatewayJwtVerifierAdapter`) |
| Tech stack | Java 25, Spring Boot 4.0.8 / Spring Cloud Gateway, Maven |
| Communicates with | All 9 internal microservices (`:8081`–`:8089`) over private network |
| Consumed by | Responsive Web UI (`12-ux-ui/`), Postman/Swagger dev clients |

---

## Responsibilities (What This Service DOES)

- **Centralized JWT Verification (`ADR-006`, `NFR-004`):** Validates `Authorization: Bearer <jwt>` signature (`HS256` ≥ 256-bit secret; RS256/JWKS upgrade ready) and `exp` timestamp before forwarding protected routes.
- **Anti-Spoofing & Context Propagation:** Strips any client-supplied `X-User-*` headers at the edge and injects verified `X-User-Id`, `X-User-Roles`, `X-User-Permissions`, and `X-Correlation-Id` headers downstream.
- **Public Route Allowlist:** Permits unauthenticated passthrough exclusively for `GET /health`, `POST /api/v1/auth/login`, `POST /api/v1/auth/register`, `POST /api/v1/auth/activate`, `POST /api/v1/auth/refresh`, and `GET /api/v1/auth/jwks`.
- **Rate Limiting & CORS:** Enforces per-client IP throttling (`GATEWAY_RPM_LIMIT`, default `240 RPM`, HTTP `429`) and centralized CORS policy.
- **Path-Based Routing:** Proxies `/api/v1/*` prefixes to the 9 canonical microservice ports (`:8081`–`:8089`).

## Out of Scope (What It Does NOT Do)

- **No business logic or database access:** Never queries MySQL schemas or evaluates domain invariants.
- **No `:own` / `:assigned` ownership evaluation:** Fine-grained ownership checks remain in each downstream service's Application layer (`P5`).
- **No credential or refresh token storage:** Delegated strictly to `iam-service` (`:8081`).

---

## Canonical Route Table

| Path Prefix | Downstream Service | Canonical URL Env Var | Canonical Port |
|---|---|---|---|
| `/api/v1/auth/**`, `/api/v1/users/**`, `/api/v1/roles/**` | `iam-service` | `IAM_SERVICE_URL` | `8081` |
| `/api/v1/announcements/**`, `/api/v1/notifications/**` | `communications-service` | `COMMUNICATIONS_SERVICE_URL` | `8082` |
| `/api/v1/units/**` | `units-service` | `UNITS_SERVICE_URL` | `8083` |
| `/api/v1/fees/**`, `/api/v1/payments/**` | `billing-service` | `BILLING_SERVICE_URL` | `8084` |
| `/api/v1/people/**`, `/api/v1/occupancies/**` | `people-service` | `PEOPLE_SERVICE_URL` | `8085` |
| `/api/v1/maintenance-requests/**` | `maintenance-service` | `MAINTENANCE_SERVICE_URL` | `8086` |
| `/api/v1/visits/**`, `/api/v1/correspondence/**`, `/api/v1/preauthorizations/**` | `access-control-service` | `ACCESS_CONTROL_SERVICE_URL` | `8087` |
| `/api/v1/expense-proposals/**` | `finance-approval-service` | `FINANCE_APPROVAL_SERVICE_URL` | `8088` |
| `/api/v1/reports/**` | `reports-service` | `REPORTS_SERVICE_URL` | `8089` |

---

## Local Startup

```bash
# Run locally (uses port 8090 by default on Windows if 8080 is busy; set SERVER_PORT=8080 in Docker)
mvn spring-boot:run

# Run automated routing, JWT & rate-limit test suite
mvn test

# Verify gateway liveness
curl http://127.0.0.1:8080/health
```

---

## Environment Variables

| Variable | Description | Default (Local) |
|---|---|---|
| `SERVER_PORT` | HTTP listen port (`8080` Docker canonical, `8090` Windows fallback) | `8090` |
| `JWT_SECRET` | Shared HS256 signing secret (≥ 256 bits; injected via Vault/env in prod) | Local dev secret |
| `GATEWAY_RPM_LIMIT` | Per-client requests per minute threshold | `240` |
| `IAM_SERVICE_URL` .. `REPORTS_SERVICE_URL` | Downstream microservice base URLs (`http://localhost:8081`..`8089`) | Local ports |

---

## Documentation Links

- OpenAPI Contract → [`07-api/contracts/openapi/api-gateway.yaml`](../../../07-api/contracts/openapi/api-gateway.yaml)
- Architecture Decision → [`ADR-006`](../../../05-architecture/decisions/records/ADR-006-jwt-authentication-strategy.md)
- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
