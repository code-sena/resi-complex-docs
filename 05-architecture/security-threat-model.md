# Security Threat Model (STRIDE) — resi-complex

> **Document Status:** Complete — Proactive STRIDE threat analysis across all 9 microservices, API Gateway, MySQL 8 databases, and RabbitMQ  
> **Aligned with:** `00-governance/security-policy.md`, `00-governance/security-rules.md`, `04-requirements/non-functional.md` (`NFR-004`), `ADR-004`, `ADR-006`, Colombian **Ley 1581 de 2012**  
> **Last Updated:** 2026-11-21

---

## 1. Trust Boundaries & Crown-Jewel Assets

| Asset / Data Class | Owning Service & DB | Sensitivity | Regulatory / Business Impact |
|--------------------|---------------------|-------------|------------------------------|
| User credentials & refresh token hashes | `iam-service` (`iam_db`) | **Critical** | Full account takeover across all 5 roles |
| Resident & commercial owner/tenant PII (document ID, phone, email, occupancies) | `people-service` (`people_db`) | **High** | Colombian **Ley 1581 de 2012** (Habeas Data) liability |
| Administration fees, tariff snapshots & payment records | `billing-service` (`billing_db`) | **High** | Financial fraud, altered arrears or unauthorized payment confirmation |
| Board extraordinary expense votes & quorum audit ledger | `finance-approval-service` (`finance_approval_db`) | **High** | Tampered governance decisions / repudiation during assembly audits |
| Visitor/vehicle logs, pre-authorizations & parcel pickup PINs | `access-control-service` (`access_control_db`) | **Medium–High** | Physical perimeter breach or stolen resident packages |
| Inter-service AMQP domain events | RabbitMQ (`resicomplex.events`) | **Medium** | Forged events corrupting `reports_db` projections or triggering fake notifications |

**Trust Boundaries:**
1. **TB-1 (Internet / Browser → `api-gateway` `:8080`):** Untrusted external traffic. Terminates TLS (staging/prod), enforces CORS, rate limits, and JWT verification.
2. **TB-2 (`api-gateway` → Internal Microservices `:8081–:8089`):** Private Docker/cluster network. Gateway strips inbound `X-User-*` headers and injects verified identity context.
3. **TB-3 (Microservices → Dedicated MySQL 8 Schemas & RabbitMQ):** Internal data boundary enforced by per-service credentials and strict Database per Service isolation (`ADR-004`).

---

## 2. STRIDE Threat Matrix & Mitigations

| ID | STRIDE Category | Target Component | Threat Scenario | Likelihood | Impact | Mandatory Mitigation | Verification / Policy |
|----|-----------------|------------------|-----------------|------------|--------|----------------------|-----------------------|
| **TH-01** | **S — Spoofing** | `iam-service` / `api-gateway` | Credential stuffing, brute-force login, or forged JWT token | High | Critical | BCrypt password hashing (cost ≥ 12); account lockout after 5 failed attempts (`15 min`, `BR-02`); strict JWT signature verification (`HS256` ≥ 256-bit secret / `RS256` JWKS); 1h access TTL; generic `401` login errors (no account enumeration) | `AuthenticationServiceTest`, OWASP ZAP, `ADR-006` |
| **TH-02** | **S — Spoofing** | Internal Microservices (`:8081–:8089`) | Attacker sends spoofed `X-User-Id` / `X-User-Roles: ADMINISTRATOR` headers directly to a service or through the gateway | Medium | Critical | `api-gateway` strips all client-supplied `X-User-*` headers before verification; internal ports bound exclusively to isolated Docker/cluster network (never publicly exposed); mTLS/service-level JWT verification in production | Gateway filter tests (`GatewayRoutingApplicationServiceTest`), network policy audit |
| **TH-03** | **S — Spoofing / Replay** | `iam-service` (`refresh_tokens`) | Stolen refresh token replayed after legitimate rotation | Medium | High | Mandatory one-use refresh token rotation (`7d` TTL, BCrypt-hashed in `iam_db.refresh_tokens`); reuse of a rotated/revoked token triggers immediate revocation of the entire token family (`BR-03`) | `AuthenticationServiceTest`, `00-governance/security-policy.md` |
| **TH-04** | **T — Tampering** | All Microservices (MySQL 8) | SQL Injection via search/filter parameters (`unitNumber`, `documentId`, `status`) | Medium | Critical | 100% parameterized queries via Spring Data JPA / Prepared Statements; Bean Validation (`@Valid`) allowlist schemas at every controller boundary; zero string concatenation in SQL | Semgrep SAST, PR code review (`00-governance/security-rules.md`) |
| **TH-05** | **T — Tampering** | `finance-approval-service` (`approvals`) | Administrator or Board member alters or deletes a cast vote after quorum resolution | Low | High | Append-only `approvals` table (no `UPDATE`/`DELETE` endpoints or repository mutators); unique constraint `(proposal_id, board_member_id)`; quorum auto-locks proposal state (`BR-15`) | `ExpenseProposalServiceTest`, `06-data/models.md` |
| **TH-06** | **T — Tampering** | `billing-service` (`administration_fees`) | Retroactive unit type change alters already-billed fees or overpayment corrupts balance | Medium | High | `unit_type_at_generation` snapshot frozen on fee creation (`BR-10`); MySQL `CHECK (total_amount > 0)` and payment sum validation (`BR-11`) inside ACID transaction | `FeeGenerationServiceTest`, `PaymentServiceTest` |
| **TH-07** | **R — Repudiation** | `finance-approval-service`, `access-control-service`, `maintenance-service` | Actor denies approving an expense, delivering a parcel, or updating a maintenance ticket | Medium | High | Immutable audit columns (`created_at`, `created_by`, `recorded_by`, `voted_at`, `delivered_at`); 4-digit `pickup_pin` verification for parcel delivery (`BR-14`); structured JSON audit logs carrying `X-Correlation-Id` and `userId` (`NFR-005`) | Structured logging tests, DB audit columns |
| **TH-08** | **I — Information Disclosure** | `people-service`, `billing-service`, `access-control-service` | Horizontal privilege escalation (IDOR): Resident A queries Resident B's fees, maintenance requests, or packages; or former tenant retains access after lease ends | High | High | Mandatory ownership-level authorization (`:own`, Principle `P5`) enforced in Application layer against `unit_occupancies.status = 'ACTIVE'`; deactivating occupancy (`HU-PPL-002`, `BR-07`) immediately revokes unit-scoped access; PII isolated in `people_db` (`ADR-004`) | Integration tests with cross-unit `403 Forbidden` assertions (`FR10`, `FR16`, `FR21`) |
| **TH-09** | **I — Information Disclosure** | All Services & Git Repos | Stack traces, SQL errors, PII in logs, or secrets committed in Git | Medium | High | Centralized RFC 7807 `ErrorResponse` hides internal exceptions (`500` generic message + `traceId`); zero PII/passwords/tokens in JSON logs; `Gitleaks` secret scanning in CI; `.env` excluded in `.gitignore` | `Gitleaks`, error handler unit tests, `00-governance/security-rules.md` |
| **TH-10** | **D — Denial of Service** | `api-gateway` & RabbitMQ | High-rate request flooding, large payload exhaustion, or poison-pill AMQP message loop | Medium | Medium | Per-client rate limiting at `api-gateway` (`240 RPM` default, tighter login limit); pagination caps (`limit <= 100`, `07-api/guidelines.md`); RabbitMQ max retry + Dead Letter Queues (`ADR-005`); Resilience4j timeouts & circuit breakers | `GatewayRoutingApplicationServiceTest`, k6 load test (`NFR-001`) |
| **TH-11** | **E — Elevation of Privilege** | `iam-service` & All Services | Self-registration escalates role to `ADMINISTRATOR` or `BOARD`; staff accesses admin-only endpoints | Medium | Critical | Public self-registration cannot assign privileged roles — `ADMINISTRATOR`, `BOARD`, `MAINTENANCE_STAFF`, and `SECURITY_GUARD` accounts are provisioned strictly by `ADMINISTRATOR` (`FR20`, `BR-01`); strict RBAC checks on every endpoint matching `07-api/authentication.md` | `UserProvisioningServiceTest`, OpenAPI security tests |

---

## 3. DevSecOps Verification Gates

Before promoting any release candidate (`dev` → `main` / MVP Sign-Off):
1. **SCA (Dependencies):** Trivy / OWASP Dependency-Check — **0 Critical/High CVEs** across all 10 Maven POMs.
2. **SAST (Source Code):** Semgrep / SpotBugs — **0 OWASP Top 10 High/Critical findings** (SQLi, XSS, hardcoded crypto).
3. **Secret Scanning:** Gitleaks across full Git commit history — **0 exposed credentials or JWT secrets**.
4. **Architecture Fitness:** ArchUnit verifying controllers never bypass use cases or leak JPA entities (`NFR-006`).

---

## 4. Correlations

- Security Policy & Rules → `00-governance/security-policy.md`, `00-governance/security-rules.md`
- API Authentication & Permissions Matrix → `07-api/authentication.md`
- Non-Functional Security Requirements → `04-requirements/non-functional.md` (`NFR-004`)
- Cross-Cutting Controls → `05-architecture/cross-cutting.md`
