# ADR-003 — Microservices + Event-Driven Architectural Style

| Field | Value |
| :--- | :--- |
| **ID** | ADR-003 |
| **Date** | 2026-09-17 *(reconciled 2026-11-21)* |
| **Status** | Accepted |
| **Authors** | Fabián Trujillo — Tech Lead & Project Leader |
| **Reviewers** | Alexandra Garzón, Félix Murcia, Julián Losada — Development Team |
| **Supersedes / Related to** | Complements `ADR-004`, `ADR-005`, `ADR-006` |

---

## Context

The `resi-complex` system manages mixed-use residential complexes in Colombia, containing both residential housing units and commercial establishments. Domain-Driven Design analysis (`02-domain/domain-map.md`) decomposed the system into **9 distinct Bounded Contexts**: Identity & Access Management (IAM), Units Management, People Management, Maintenance, Billing, Communications, Access Control, Financial Approval, and Reports.

Core domain analysis establishes that **Billing** and **Units Management** carry the primary competitive differentiator (native residential vs. commercial tariff differentiation and establishment management), whereas **IAM** and **Reports** are generic domains. Additionally, operational workflows rely on 16 decoupled domain events and 6 reactive cross-context policies (`02-domain/domain-events.md`), such as alerting residents when fees are generated or urgent maintenance requests change status.

**Known Constraints:**
- Confirmed stack: **Java 25 + Spring Boot 4.0.8** (LTS baseline Java 17+ / Spring Boot 3.x+), Hexagonal Architecture (`_stacks/java-spring.md`), MySQL 8.0+.
- Strict independent testability, parallel team ownership across 4 SENA ADSO 3239188 developers, and clear contract boundaries (`07-api/contracts/openapi/`).

---

## Decision

**We decided:** To adopt a **Microservices + Event-Driven Architectural Style**, deploying each of the **9 Bounded Contexts** as an autonomous, independently deployable Spring Boot microservice structured internally with **Hexagonal Architecture (Ports & Adapters)**, placed behind a single **API Gateway (`:8080`)**, communicating:
- **Synchronously via REST/JSON** (OpenAPI 3.0.3 contracts) for direct request/response queries and commands.
- **Asynchronously via Domain Events** (RabbitMQ + Transactional Outbox) for cross-context state propagation, reactive notifications, and CQRS read projections (`reports-service`).

**Justification:**
1. **1:1 Bounded Context Alignment:** Mirrors `02-domain/domain-map.md` directly without cross-context leakage.
2. **Targeted Engineering Investment:** Concentrates deep invariant testing on Core Domains (`billing-service`, `units-service`) without coupling them to generic services (`iam-service`, `reports-service`).
3. **Parallel Team Velocity:** Each teammate owns autonomous GitHub repositories under `https://github.com/resi-complex` governed by shared OpenAPI contracts.
4. **Native Event-Driven Support:** Directly executes the 16 domain events and reactive policies specified in `02-domain/domain-events.md`.

---

## Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: Microservices + Event-Driven (CHOSEN)** | Strict domain isolation; independent deployment & scaling; native event-driven decoupling; clear per-developer repo ownership | Operational overhead of Gateway, RabbitMQ, and 9 service configs | — *(Chosen)* |
| **Option B: Layered Monolith** | Trivial local startup; single deployable artifact | Shared database encourages cross-domain SQL joins; tight coupling between Core and Generic domains | Violates DDD boundaries and prevents independent service deployment |
| **Option C: Modular Monolith** | Lower operational overhead; in-process events | No physical enforcement of network/database boundaries; weaker demonstration of distributed microservice competencies | Does not meet project requirement for autonomous deployable services |

---

## Consequences

**Positive:**
- Strict encapsulation of the 9 Bounded Contexts verified by ArchUnit fitness tests (`NFR-006`).
- Independent release cadence across the 10 repositories (`api-gateway` + 9 microservices).
- Resilient, decoupled asynchronous workflows (`communications-service`, `reports-service`).

**Negative / Trade-offs:**
- Requires unified local orchestration via Docker Compose (`05-architecture/deployment.md`, `10-devops/local-setup.md`).
- Introduces eventual consistency across contexts, mitigated via Transactional Outbox and Idempotent Consumers (`ADR-005`, `06-data/models.md`).

**Impact on the System:**
- Affected Services: All 9 microservices (`iam-service` `:8081`, `communications-service` `:8082`, `units-service` `:8083`, `billing-service` `:8084`, `people-service` `:8085`, `maintenance-service` `:8086`, `access-control-service` `:8087`, `finance-approval-service` `:8088`, `reports-service` `:8089`) plus `api-gateway` (`:8080`).
- Updated Documents: `05-architecture/overview.md`, `09-microservices/service-catalog.md`.

---

## Risks

| Risk | Probability | Impact | Mitigation |
| :--- | :--- | :--- | :--- |
| Local orchestration complexity slows onboarding | Medium | High | Maintain single-command Docker Compose setup (`05-architecture/deployment.md`, `10-devops/local-setup.md`) |
| Event loss during service crashes | Medium | High | Mandatory Transactional `outbox` table + `processed_events` deduplication (`06-data/models.md`) |
| Contract drift between repos | Medium | High | Enforce OpenAPI linting (`npx @redocly/cli lint`) and contract tests in CI (`07-api/`, `11-quality/`) |

---

## References

- DDD Bounded Contexts → `02-domain/domain-map.md`
- Domain Events Catalog → `02-domain/domain-events.md`
- System Architecture Overview → `05-architecture/overview.md`
- Hexagonal Architecture Guide → `05-architecture/hexagonal-architecture.md`
