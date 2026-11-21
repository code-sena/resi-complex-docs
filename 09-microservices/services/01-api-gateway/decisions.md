# Local Technical Decisions — `api-gateway`

## Decision 1 — Pure Java / Spring Stack Over Kong or NGINX Lua
- **Context:** The entire `resi-complex` backend uses Java 25 + Spring Boot 4.0.8 (`01-context/scope.md`). Introducing Kong or custom OpenResty/Lua scripts would add a second operational runtime and configuration language.
- **Decision:** Implement `resi-complex-api-gateway` in Java/Spring with Hexagonal Architecture (`GatewayRoutingApplicationService`, `JwtVerifierPort`), formalizing `ADR-006`.
- **Consequences:** Single team skill set, fast JUnit 5 routing/security tests (4/4 passing), and seamless Docker packaging (`NFR-007`).

## Decision 2 — Configurable Local Port Fallback (`:8090`) vs. Canonical Docker Port (`:8080`)
- **Context:** Canonical architecture and Docker Compose expose the Gateway on `:8080`, but Windows developer workstations frequently have system services (`AgentService`) bound to `:8080`.
- **Decision:** Default `server.port=${SERVER_PORT:8090}` for bare-metal local runs and inject `SERVER_PORT=8080` in Docker Compose and production containers.
- **Consequences:** Zero port-collision friction on Windows workstations while preserving canonical `:8080` in containerized environments.

## Decision 3 — Strip Untrusted `X-User-*` Headers at the Edge
- **Context:** Downstream microservices trust `X-User-Id`, `X-User-Roles`, and `X-User-Permissions` propagated by the Gateway (`05-architecture/security-threat-model.md`, `TH-02`).
- **Decision:** Unconditionally remove any client-supplied `X-User-*` headers before JWT verification and re-inject only cryptographically verified claims.
- **Consequences:** Prevents header-injection privilege escalation from external callers.
