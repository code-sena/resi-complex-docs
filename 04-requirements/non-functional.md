# Non-Functional Requirements (NFR)

> NFRs define the **measurable quality attributes** of `resi-complex` — not what the system
> does, but how fast, securely, reliably, accessibly, and maintainably it operates.
> Stack baseline: **Java 25 + Spring Boot 4.0.8 + MySQL 8 (Database per Service) + RabbitMQ**.

---

## How to write a measurable NFR

| Bad (unverifiable) | Good (measurable & testable) |
|--------------------|------------------------------|
| "The system must be fast" | "P95 latency of `POST /api/v1/fees/generate` must be < 300ms under 50 RPS" |
| "The system must be secure" | "All private endpoints require a valid RS256/HS256 JWT expiring in 1h; passwords hashed with BCrypt cost ≥ 12" |
| "The system must work on mobile" | "Core screens pass WCAG 2.1 AA (axe-core) and remain usable on 360px mobile viewports" |
| "The system must be available" | "Production availability SLO: 99.9% monthly (max 44 min downtime/month)" |

---

## NFR-001: Performance

| Attribute | Metric | Test condition |
|-----------|--------|----------------|
| P95 latency — critical endpoints | < 300 ms | Under **50 RPS** sustained load (single-complex scale) |
| P99 latency — critical endpoints | < 500 ms | Under **50 RPS** sustained load |
| P95 latency — non-critical endpoints | < 1000 ms | Normal operational load (20 RPS) |
| Minimum sustained throughput | **50 RPS** (burst peak **150 RPS**) | Without HTTP 5xx degradation |
| Service cold startup time | < 30 seconds | Spring Boot container startup with Flyway `validate` |

**Defined critical endpoints:**
- `POST /api/v1/auth/login` (`iam-service`) — gates access for all 5 roles
- `POST /api/v1/fees/generate` (`billing-service`) — monthly batch generation per unit (`FR08`–`FR09`)
- `POST /api/v1/maintenance-requests` (`maintenance-service`) — real-time reporting by occupants (`FR05`)
- `POST /api/v1/visits` (`access-control-service`) — front-desk entry logging by Security Guards (`FR12`)

**Load testing tools:** k6 or Apache JMeter executed in CI/staging prior to release.

---

## NFR-002: Availability

| Environment | SLO | Maintenance window | Max downtime/month |
|-------------|-----|--------------------|--------------------|
| Production (target) | 99.9% | Sundays 02:00–04:00 COT | 44 minutes |
| Staging (target) | 95.0% | Unrestricted | 36 hours |

> **Scope status:** `01-context/scope.md` confirms the Local environment for the formative delivery; Staging/Production SLOs define the target architectural contract once deployed.

**Error Budget policy:** If > 50% of the monthly 44-minute error budget is consumed before mid-month, feature deployments freeze in favor of stability fixes.

**Health checks (Spring Boot Actuator):**
- `GET /actuator/health/liveness` (`GET /health`) — HTTP 200 if JVM process is alive
- `GET /actuator/health/readiness` (`GET /health/ready`) — HTTP 200 only if MySQL 8 and RabbitMQ connections are healthy

---

## NFR-003: Scalability

| Scenario | Expected behavior |
|----------|-------------------|
| Gradual load growth | Horizontal scaling triggered when CPU > 70% |
| Traffic burst (monthly billing day / emergency alert) | Scale-out completes in < 2 minutes |
| Load reduction | Graceful scale-down draining in-flight requests |
| Horizontal scaling limit | Up to 3 stateless instances per service (single-complex scale, `01-context/scope.md` assumption #3) |

**Strategy:** Stateless horizontal scaling — service instances keep zero in-memory session state. Access tokens are stateless JWTs (`ADR-006`), refresh tokens persist in `iam_db.refresh_tokens`, and domain state persists in dedicated MySQL 8 schemas (`ADR-004`). Redis is not required in the MVP scope (`06-data/README.md`).

---

## NFR-004: Security

### Authentication & Authorization
- All private endpoints require a valid JWT (`Authorization: Bearer <token>`) validated at the API Gateway and/or service layer (`ADR-006`)
- Access token expiration: **1 hour**; refresh token expiration: **7 days** with mandatory one-use rotation and token-family reuse revocation (`00-governance/security-policy.md`)
- Strict RBAC for the 5 authentication roles plus service-layer ownership verification (`:own`, `:assigned`) backed by `unit_occupancies.status = 'ACTIVE'`

### Data Protection & OWASP Top 10
- HTTPS mandatory (TLS 1.2+) outside local development
- Passwords and refresh tokens hashed with **BCrypt (cost ≥ 12)**; secrets injected solely via environment variables / secret manager (never committed to Git)
- 100% parameterized queries (JPA / Prepared Statements); RFC 7807 `ErrorResponse` with zero stack-trace leakage in production
- Static security checks (SCA + SAST + Secret Scanning) on every release
- **Regulatory compliance:** Colombian **Ley 1581 de 2012** (Habeas Data / personal data protection) across `people-service`, `iam-service`, and `access-control-service`

---

## NFR-005: Observability

| Pillar | Requirement | Tool (Java / Spring Boot stack) |
|--------|-------------|----------------------------------|
| **Logs** | Structured JSON format with `timestamp`, `service`, `level`, `correlationId`, `userId` (no `System.out.println` in prod) | SLF4J + Logback (`logstash-logback-encoder`) |
| **Metrics** | RED metrics (Rate, Errors, Duration) + JVM/HikariCP/Outbox backlog gauges | Micrometer + Prometheus + Grafana |
| **Traces** | End-to-end distributed tracing across REST and RabbitMQ spans | OpenTelemetry Java Agent + Jaeger |
| **Alerts** | Alert within < 5 minutes upon SLO breach or HTTP 5xx spike | Alertmanager / Slack |

**Correlation ID:** Every incoming request carries or generates an `X-Correlation-Id` UUID propagated across all HTTP headers, RabbitMQ message headers, logs, and traces.

---

## NFR-006: Maintainability & Architectural Fitness

| Metric | Target | Verification |
|--------|--------|--------------|
| Automated test coverage | ≥ 80% lines & branches (≥ 90% in `domain/`) | JaCoCo Quality Gate in CI |
| Cyclomatic complexity (McCabe) | ≤ 10 per method; methods < 50 lines; files < 800 lines | Checkstyle / PMD / SonarQube |
| Architectural Fitness Functions | Zero hexagonal boundary violations (`domain` imports zero `org.springframework.*` / `jakarta.persistence.*`) | ArchUnit in `mvn test` |
| Onboarding time | New developer boots local environment in < 1 hour | `10-devops/local-setup.md` |
| CI build duration | < 5 minutes per microservice | GitHub Actions |

---

## NFR-007: Portability

- Every microservice is packaged as a reproducible multi-stage OCI/Docker image (Java 25 JRE runtime)
- Zero host-OS dependencies; all environment differences (`DB_URL`, `JWT_SECRET`, `RABBITMQ_HOST`, `SERVER_PORT`) are externalized via 12-Factor environment variables with fail-fast validation at startup
- Local multi-service orchestration supported via unified `docker-compose.yml`

---

## NFR-008: Disaster Recovery (DR)

| Scenario | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) | Scope |
|----------|-------------------------------|--------------------------------|-------|
| Single service container crash | < 2 minutes (automatic restart) | 0 (stateless + transactional `outbox`) | MVP / Local & Prod |
| Primary MySQL database failure | < 5 minutes (restore / failover) | < 1 minute (automated encrypted backup / binlog) | Production target |
| Availability zone loss | < 15 minutes | < 5 minutes | Post-formative (Production) |
| Full region disaster | < 4 hours | < 1 hour | Post-formative (Production) |

---

## NFR-009: Usability, Mobile Responsiveness & Accessibility

| Attribute | Metric / Standard | Verification |
|-----------|-------------------|--------------|
| Accessibility conformance | **WCAG 2.1 Level AA** (semantic HTML, ARIA labels, keyboard navigation, contrast ≥ 4.5:1) | Automated `axe-core` / Lighthouse accessibility audit (≥ 90 score) |
| Mobile responsiveness | All operational flows (`/front-desk/*`, `/requests/*`, `/fees`, `/correspondence`) fully usable on **360×800 px** viewports without horizontal scroll | Responsive browser QA across mobile (360px), tablet (768px), desktop (1280px+) |
| Core Web Vitals (frontend) | `LCP < 2.5s`, `INP < 200ms`, `CLS < 0.1` | Lighthouse / Chrome DevTools |
| Normal UI response perception | Page / form action feedback < 2 seconds (`vision.md` KR4) | E2E smoke test |

> *Reconciliation note:* Earlier drafts in `01-context/scope.md` and `03-product/vision.md` referenced mobile usability as `NFR03`; **`NFR-009`** is now the canonical identifier for usability, responsiveness, and WCAG 2.1 AA accessibility (`NFR-003` remains Scalability).

---

## NFR Priority Matrix

| NFR ID | Attribute | Priority | Validated in CI? | Owner |
|--------|-----------|----------|------------------|-------|
| `NFR-001` | Performance | P1 | Planned (k6 load suite) | Félix Murcia |
| `NFR-002` | Availability | P1 | Planned (Actuator health probes) | Fabián Trujillo |
| `NFR-003` | Scalability | P2 | Manual / post-staging | Fabián Trujillo |
| `NFR-004` | Security | P1 | Planned (SCA + SAST + Secret Scan) | Fabián Trujillo |
| `NFR-005` | Observability | P1 | Planned (JSON logs + Micrometer) | Julián Losada |
| `NFR-006` | Maintainability & Fitness | P1 | Planned (JaCoCo ≥ 80% + ArchUnit) | Fabián Trujillo |
| `NFR-007` | Portability | P2 | Planned (Docker build check) | Julián Losada |
| `NFR-008` | Disaster Recovery | P2 | Planned (backup restore drill) | María Alexandra Garzón |
| `NFR-009` | Usability & Accessibility (WCAG 2.1 AA) | P1 | Planned (`axe-core` + responsive QA) | María Alexandra Garzón |

---

## Correlations

- Functional Requirements & Business Rules → `04-requirements/functional.md`
- Traceability Matrix (RTM) → `04-requirements/traceability-matrix.md`
- Security Policy (RBAC, JWT, BCrypt) → `00-governance/security-policy.md`
- Design System & Responsive Routes → `12-ux-ui/design-system.md`, `12-ux-ui/navigation-map.md`
- Detailed SLOs / Observability → `13-operations/observability.md`
