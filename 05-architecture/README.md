# 05 — Architecture

> **What is this?** The system's structural design decisions: how `resi-complex` is organized into
> 9 microservices + API Gateway, why each technology was chosen (`ADR-001`–`ADR-006`), how
> cross-cutting concerns and security threats are controlled, and how containers are deployed.

---

## Why this section exists

A system's architecture is the set of decisions that are hard to change later. Documenting them ensures:
1. **Fast onboarding:** Any new developer understands the topology, ports, and boundaries immediately.
2. **No repeated debates:** Trade-offs and rejected alternatives are permanently recorded in ADRs.
3. **Auditability:** SENA reviewers and engineers can trace every `NFR` (`04-requirements/non-functional.md`) directly to its architectural mechanism.

For **`resi-complex`**, this section translates the 9 Bounded Contexts in `02-domain/domain-map.md` into 9 autonomous microservices structured with **Hexagonal Architecture (Ports & Adapters)** on **Java 25 + Spring Boot 4.0.8**, **MySQL 8.0+ (Database per Service)**, **RabbitMQ 3.12+ (Transactional Outbox)**, and **Spring Cloud Gateway**.

---

## Status

| File | Status | Description |
|------|--------|-------------|
| `overview.md` ⭐ | ✅ Complete | C4 Context & Container diagrams, reconciled 10-service port/DB catalog (`:8080`, `:8081–:8089`), architectural principles P1–P5, resolved technical debt (`AT-001`–`AT-004`) |
| `hexagonal-architecture.md` | ✅ Complete | Java + Spring Boot Ports & Adapters blueprint with `MaintenanceRequest` worked example and ArchUnit enforcement rules |
| `pattern-guide.md` | ✅ Complete | GoF & distributed microservices pattern catalog with updated project adoption matrix (`ADR-003`–`ADR-006`) |
| `deployment.md` ⭐ | ✅ Complete | Local Docker Compose topology, canonical port/DB allocation, 12-Factor packaging (`NFR-007`), and Kubernetes production evolution path |
| `cross-cutting.md` | ✅ Complete | System-wide standards: JWT + `X-User-*` propagation, `:own`/`:assigned` guards, RFC 7807 errors, JSON logging + `X-Correlation-Id`, Actuator health, Resilience4j, Outbox idempotency |
| `security-threat-model.md` | ✅ Complete | Proactive **STRIDE** threat model across Gateway, all 9 services, MySQL 8, and RabbitMQ, aligned with `NFR-004` and Ley 1581 de 2012 |
| `decisions/README.md` ⭐ | ✅ Complete | Clean ADR register indexing `ADR-001` through `ADR-006` |
| `decisions/records/ADR-001`–`ADR-006` | ✅ Accepted | All 6 formal ADRs materialized as standalone records (Language, Research Exception, Microservices+EDA, Database per Service, RabbitMQ+Outbox, JWT+Gateway) |
| `decisions/records/architecture-decisions-and-infrastructure.md` | ✅ Complete | Consolidated infrastructure & Spring Boot Transactional Outbox reference guide |

---

## ADR Summary (`decisions/records/`)

| ID | Decision | Status |
|----|----------|--------|
| **ADR-001** | Documentation & code in English | ✅ Accepted |
| **ADR-002** | Bounded Spanish exception for raw user-research artifacts (`03-product/`) | ✅ Accepted |
| **ADR-003** | Microservices + Event-Driven Architectural Style (9 Bounded Contexts) | ✅ Accepted |
| **ADR-004** | Database per Service on MySQL 8.0+ (`*_db` + Flyway + `ddl-auto=validate`) | ✅ Accepted |
| **ADR-005** | RabbitMQ 3.12+ (`resicomplex.events`) + Transactional Outbox & `processed_events` | ✅ Accepted |
| **ADR-006** | Spring Cloud Gateway (`:8080` / local `:8090`) + IAM JWT (`:8081`, 5 canonical roles) | ✅ Accepted |

---

## Correlations with other sections

| This section is fed by... | And feeds... |
|---------------------------|--------------|
| `02-domain/domain-map.md` & `domain-events.md` | `09-microservices/service-catalog.md` & `services/01-*`..`10-*` |
| `04-requirements/non-functional.md` (`NFR-001`–`NFR-009`) | `overview.md`, `cross-cutting.md`, `security-threat-model.md`, `deployment.md` |
| `00-governance/security-policy.md` (5 roles + `:own`/`:assigned`) | `ADR-006`, `security-threat-model.md`, `07-api/authentication.md` |
| `ADR-004` & `ADR-005` | `06-data/models.md` (9 MySQL schemas, `outbox`, `processed_events`) |
| `deployment.md` | `10-devops/local-setup.md` & `10-devops/environments.md` |

---

## Questions this section answers

- **How is the system organized?** → 9 microservices + API Gateway, 1:1 with Bounded Contexts (`overview.md` §4, `ADR-003`).
- **Why was each key technology chosen?** → Recorded with evaluated alternatives in `ADR-001` through `ADR-006`.
- **How are security threats and cross-cutting concerns enforced?** → `security-threat-model.md` (STRIDE) and `cross-cutting.md`.
- **How is the system deployed locally and evolved to production?** → `deployment.md` (Docker Compose MVP → Kubernetes post-formative).
