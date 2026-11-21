# Data Model — `api-gateway`

> **Persistence Strategy:** **Stateless (No Database).** Per `ADR-004` and `ADR-006`, `api-gateway` owns zero domain tables and never connects to any MySQL database.

---

## Ephemeral In-Memory / Cache State

| State Structure | Storage (MVP) | Storage (Multi-Replica Production) | TTL / Lifecycle | Purpose |
|-----------------|---------------|------------------------------------|-----------------|---------|
| Route definitions (`ServiceRouteDefinition`) | Externalized `application.properties` / env vars | ConfigMap / Spring Cloud Config | Process lifetime | Maps `/api/v1/*` prefixes to the 9 internal microservices (`:8081`–`:8089`) |
| Rate-limit counters (`clientIp → requestTimestamps`) | In-memory concurrent sliding window | Shared Redis cluster (`NFR-003`) | 60 seconds | Enforces `GATEWAY_RPM_LIMIT` (default `240 RPM`) and returns `429 Too Many Requests` |
| JWKS public-key cache *(when RS256 enabled)* | In-memory cache refreshed from `iam-service` `/api/v1/auth/jwks` | In-memory cache with ETag | 15 minutes | Enables stateless asymmetric JWT verification without synchronous IAM lookups |

## Architectural Rationale

Keeping `api-gateway` strictly stateless guarantees sub-millisecond edge overhead (`NFR-001`), eliminates database single-points-of-failure at the perimeter, and enables instant horizontal scaling (`NFR-003`).
