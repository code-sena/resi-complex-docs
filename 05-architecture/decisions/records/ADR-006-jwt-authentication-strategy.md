# ADR-006 — JWT Authentication & API Gateway Strategy

| Field | Value |
| :--- | :--- |
| **ID** | ADR-006 |
| **Date** | 2026-09-17 *(reconciled 2026-11-21)* |
| **Status** | Accepted |
| **Authors** | Fabián Trujillo — Tech Lead & Project Leader |
| **Reviewers** | Alexandra Garzón, Félix Murcia, Julián Losada — Development Team |
| **Supersedes / Related to** | Resolves `AT-001`; enforces `00-governance/security-policy.md` & `NFR-004` |

---

## Context

`resi-complex` enforces Role-Based Access Control (RBAC) across **5 canonical authentication roles** (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`) plus fine-grained **Ownership-Level Authorization** (`:own` / `:assigned`, architectural principle `P5`). Exposing 9 independent microservices directly to web/mobile clients duplicates JWT verification, fractures CORS configuration, leaves services exposed to unthrottled traffic, and increases attack surface.

**Known Constraints:**
- Access tokens must expire in **1 hour (3600 s)**; refresh tokens valid for **7 days**, single-use rotated with BCrypt hash persistence and token-family revocation on reuse detection (`00-governance/security-policy.md`, `NFR-004`).
- Pure Java / Spring Boot ecosystem (`01-context/scope.md`).
- Canonical Docker Gateway port is **`8080`** (with configurable `SERVER_PORT=8090` local developer fallback when `:8080` is occupied on Windows hosts).

---

## Decision

**We decided:** To deploy **`api-gateway` (`resi-complex-api-gateway`, Spring Boot / Spring Cloud Gateway, canonical port `:8080`, local fallback `:8090`)** as the single edge entry point paired with **`iam-service` (`resi-complex-iam-service`, port `:8081`, `iam_db`)** as the centralized JWT issuer:

1. **Token Issuance (`iam-service`):**
   - Validates credentials against BCrypt hashes (cost ≥ 12) and issues signed JWT Access Tokens (`1h` expiration) + opaque rotatable Refresh Tokens (`7d` expiration stored BCrypt-hashed in `iam_db.refresh_tokens`).
   - **Signing Algorithm:** **HS256** with a ≥ 256-bit secret (`JWT_SECRET` environment variable) in Phase 1 MVP, with a clean upgrade path to **RS256 + JWKS (`GET /api/v1/auth/jwks`)** for multi-team production deployments (`00-governance/security-policy.md`).
   - **Canonical JWT Claims:** `sub` (user UUID), `email`, `roles` (array with one of the 5 canonical roles), `permissions` (`[resource]:[action]` + `:own`/`:assigned`), `unitIds` (active unit UUIDs for `PERSON`), `iat`, `exp`, `jti`.
2. **Centralized Edge Verification (`api-gateway`):**
   - Enforces centralized CORS, per-client rate limiting (`GATEWAY_RPM_LIMIT`, default 240 RPM), `X-Correlation-Id` injection, and stateless JWT signature/expiration verification on all protected routes.
   - Public passthrough allowlist is strictly bounded to: `GET /health`, `POST /api/v1/auth/login`, `POST /api/v1/auth/register`, `POST /api/v1/auth/activate`, `POST /api/v1/auth/refresh`, and `GET /api/v1/auth/jwks`.
3. **Trusted Context Propagation:**
   - Strips any client-spoofed `X-User-*` headers at the edge and injects verified headers downstream:
     - `X-User-Id`: Authenticated user UUID (`JWT.sub`)
     - `X-User-Roles`: Verified canonical role (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`)
     - `X-User-Permissions`: Comma-separated RBAC permissions
     - `X-Correlation-Id`: Distributed trace correlation UUID
4. **Defense-in-Depth Ownership Authorization:**
   - Downstream microservices validate RBAC permissions and enforce `:own` / `:assigned` ownership invariants in their **Application / Use Case layer** (Principle `P5`), never relying solely on edge routing.

---

## Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: Spring Gateway + IAM JWT (CHOSEN)** | Single Java/Spring stack; centralized JWT validation, CORS, and rate limiting; clean header propagation; fast hexagonal testability | Gateway must be scaled horizontally in high-availability production | — *(Chosen)* |
| **Option B: Kong API Gateway** | Mature C/Lua plugin ecosystem | Introduces non-Java operational stack and extra PostgreSQL/declarative config overhead | Adds unnecessary multi-stack complexity to a Java/Spring team |
| **Option C: Direct Microservice Exposure** | Zero gateway hop | Duplicates JWT/CORS/rate-limit logic across 9 services; high misconfiguration risk | Violates Defense-in-Depth and `NFR-004` |

---

## Consequences

**Positive:**
- Single edge entry point (`:8080` Docker / `:8090` local override) shielding internal microservices.
- Consistent `ErrorResponse` (`401`/`403`/`429`) and `X-Correlation-Id` tracing on every request.
- Both `resi-complex-iam-service` (`:8081`) and `resi-complex-api-gateway` are implemented and verified with unit/integration tests.

**Negative / Trade-offs:**
- Requires network isolation in Docker/Kubernetes so internal services reject unproxied external traffic that could spoof `X-User-*` headers (`05-architecture/security-threat-model.md`).

---

## Risks

| Risk | Probability | Impact | Mitigation |
| :--- | :--- | :--- | :--- |
| Header spoofing (`X-User-Id`) if a downstream service is accidentally exposed publicly | Low | Critical | Bind internal services to Docker internal network only; strip inbound `X-User-*` headers at Gateway; optionally enable service-level JWT validation/mTLS in production |
| Compromised refresh token reused by attacker | Medium | High | Mandatory one-use rotation + automatic revocation of entire token family in `iam-service` (`BR-03`) |

---

## References

- Security Policy & 5 Canonical Roles → `00-governance/security-policy.md`
- API Authentication Specification → `07-api/authentication.md`
- STRIDE Security Threat Model → `05-architecture/security-threat-model.md`
- Gateway & IAM Service Specs → `09-microservices/services/01-api-gateway/`, `09-microservices/services/02-auth-service/`
