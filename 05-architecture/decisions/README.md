# ADRs — Architecture Decision Records

ADRs document important architectural decisions. Each file = one decision.

## How to create an ADR

1. Copy `_template-adr.md`
2. Name it `ADR-NNN-short-title.md` (e.g.: `ADR-003-message-broker.md`)
3. Fill it in completely — especially the evaluated alternatives
4. Once accepted, the status is **permanent** (it is not deleted, it is "Superseded" by another ADR)
5. Every ADR written from this point on must itself comply with `ADR-001-idioma-documentacion.md`
   (English, no exceptions) — the one narrow exception (`ADR-002`) applies only to raw
   user-research artifacts in `03-product/`, not to anything in this folder.

## Possible statuses

- `Proposed` — under discussion
- `Accepted` — approved by the team
- `Rejected` — evaluated and discarded (document why)
- `Superseded` — superseded by ADR-NNN (indicate which one)

## ADR register

| # | Title | Status | Date |
|---|-------|--------|------|
| ADR-001 | Documentation Language | Accepted | 2026-08-28 |
| ADR-002 | Language Exception for User-Research Artifacts | Accepted | 2026-09-16 |
| **ADR-003** | `05-architecture/decisions/records/ADR-003-architectural-style.md` | Decision record for Microservices + Event-Driven style |
| **ADR-004** | `05-architecture/decisions/records/ADR-004-database-per-service.md` | Decision record for Database per Service pattern |
| **ADR-005** | `05-architecture/decisions/records/ADR-005-message-broker-selection.md` | Decision record for Message Broker selection (RabbitMQ) |
| **ADR-006** | `05-architecture/decisions/records/ADR-006-jwt-authentication-strategy.md` | Decision record for JWT Auth & API Gateway Strategy |
| **ADR Register Update** | `05-architecture/decisions/README.md` | Updated table of Architecture Decision Records |
| **Infrastructure Def.** | `05-architecture/overview.md` & `05-architecture/deployment.md` | Resolutions for AT-001 (Gateway), AT-002 (Broker), AT-003 (Orchestration) |
| **Outbox Pattern Spec** | `05-architecture/pattern-guide.md` & `06-data/models.md` | Transactional Outbox SQL schema and Spring Boot implementation guide |
| **Standard Error Schema** | `07-api/contracts/openapi/_shared.yaml` | OpenAPI 3.0 YAML definition for unified `ErrorResponse` (RFC 7807) |


Architecture Decision Records (ADRs)

### 1.1 ADR-003: Microservices + Event-Driven Architectural Style

```markdown
# ADR-003 — Microservices + Event-Driven Architectural Style

| Field | Value |
| :--- | :--- |
| **ID** | ADR-003 |
| **Date** | 2026-09-17 |
| **Status** | Accepted |
| **Authors** | Fabian Trujillo — Project Leader |
| **Reviewers** | Tech Lead, Development Team |
| **Supersedes** | N/A |

---

#### Context
The `resi-complex` system manages mixed-use residential complexes in Colombia, containing both residential housing units and commercial establishments. The system comprises 9 distinct business domains: Identity & Access Management (IAM), Units Management, People Management, Maintenance, Billing, Communications, Access Control, Financial Approval, and Reports.

Core domain analysis reveals that Billing and Units Management carry the primary competitive differentiator (differentiated fee rates for residential vs. commercial units). Furthermore, operational flows require decoupled, asynchronous reactions (e.g., notifying residents when fees are generated or when urgent maintenance requests are created).

**Known Constraints:**
- Strict domain boundaries identified during DDD analysis (9 Bounded Contexts).
- Java 17+ and Spring Boot 3.x stack requirement.
- Formative project delivery constraints requiring independent testability and clear team task allocation.

---

#### Decision
**We decided:** To adopt a **Microservices + Event-Driven Architectural Style**, deploying each of the 9 Bounded Contexts as an autonomous, independently deployable microservice communicating synchronously via REST APIs for direct queries and asynchronously via Domain Events for state changes and reactive business policies.

**Justification:** 
1. **Domain Alignment:** Directly mirrors the 9 Bounded Contexts defined in `02-domain/domain-map.md`.
2. **Independent Investment:** Allows heavy development and testing focus on Core Domains (`billing-service`, `units-service`) without risking regressions in generic domains (`iam-service`, `reports-service`).
3. **Asynchronous Decoupling:** Naturally supports the 16 domain events and reactive policies defined in `02-domain/domain-events.md`.

---

#### Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: Microservices + Event-Driven (CHOSEN)** | Clear domain isolation; independent scaling and deployment; matches domain events model natively. | Operational overhead of multiple services and event broker management. | — (Chosen) |
| **Option B: Monolithic Architecture** | Simple single-process deployment; trivial local execution. | Tightly couples core and generic domains; shared database risks domain boundary leakage; fails to demonstrate microservice competencies. | Violates DDD domain separation; hampers independent service scaling and team parallelization. |
| **Option C: Modular Monolith** | In-process module boundaries with lower deployment complexity. | Shared database temptation; lacks physical enforcement of network boundaries and asynchronous messaging patterns. | Does not satisfy requirement for independent deployment and event-driven microservice demonstration. |

---

#### Consequences
**Positive:**
- Enforces strict encapsulation between the 9 Bounded Contexts.
- Enables asynchronous execution of cross-service policies (e.g., fee notifications, maintenance alerts).
- Facilitates parallel development across team members using clear contract boundaries.

**Negative / Trade-offs:**
- Requires managing an API Gateway, Message Broker, and multiple service configurations.
- Introduces eventual consistency between services (handled via reactive domain event consumers).

**Impact on the System:**
- Affected Services: All 9 microservices (`iam-service`, `units-service`, `people-service`, `maintenance-service`, `billing-service`, `communications-service`, `access-control-service`, `finance-approval-service`, `reports-service`).
- Documents Updated: `05-architecture/overview.md`, `09-microservices/service-catalog.md`.

---

#### Risks

| Risk | Probability | Impact | Mitigation |
| :--- | :--- | :--- | :--- |
| Distributed system complexity overwhelming local setup | Medium | High | Provide a unified `docker-compose.yml` for local environment orchestration. |
| Event loss during service failures | Medium | High | Implement the Transactional Outbox Pattern in producing services. |

---

#### References
- DDD Bounded Contexts → `02-domain/domain-map.md`
- Domain Events Catalog → `02-domain/domain-events.md`
```

---

### 1.2 ADR-004: Database per Service Pattern

```markdown
# ADR-004 — Database per Service Pattern

| Field | Value |
| :--- | :--- |
| **ID** | ADR-004 |
| **Date** | 2026-09-17 |
| **Status** | Accepted |
| **Authors** | Fabian Trujillo — Project Leader |
| **Reviewers** | Tech Lead, Development Team |
| **Supersedes** | N/A |

---

#### Context
In microservice architectures, data management is a critical design choice. Allowing multiple microservices to access a single shared database creates tight coupling at the data layer, causes schema refactoring friction, and allows developers to bypass domain APIs using SQL `JOIN`s across service boundaries.

**Known Constraints:**
- MySQL 8.0+ is the locked database engine for `resi-complex`.
- Strict data privacy constraints under Colombian Ley 1581 de 2012 (Habeas Data) require isolating sensitive personal data in `people-service`.

---

#### Decision
**We decided:** To enforce the **Database per Service** pattern. Each of the 9 microservices owns its own dedicated MySQL database instance/schema (`iam_db`, `units_db`, `people_db`, `maintenance_db`, `billing_db`, `communications_db`, `access_control_db`, `finance_db`, `reports_db`). Direct database access or cross-database SQL queries between microservices are strictly prohibited.

**Justification:**
1. **Autonomy & Independence:** Services can change schema and deploy migrations independently using Flyway.
2. **Domain Boundary Integrity:** Forces inter-service data exchange to occur strictly via REST APIs or Domain Events.
3. **Security & Compliance:** Restricts access to sensitive resident data (`people_db`) to the `people-service` container only.

---

#### Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: Database per Service (CHOSEN)** | Perfect isolation; independent schema migrations; zero database-level coupling. | Cannot perform cross-service SQL `JOIN`s; requires eventual consistency. | — (Chosen) |
| **Option B: Shared Single Database** | Easy cross-table `JOIN`s; single database management. | Complete coupling; schema changes in one service break others; violates microservice isolation. | Anti-pattern in microservices; destroys service autonomy. |
| **Option C: Shared Database Server with Logical Schemas** | Reduced database container footprint. | Shared CPU/RAM resources; risk of accidental cross-schema permissions. | Acceptable in local dev via logical MySQL databases, but logically treated as distinct DBs. |

---

#### Consequences
**Positive:**
- Schema changes in `units-service` (e.g., adding commercial fields) never break `billing-service`.
- Data model for each service remains clean and focused solely on its aggregate root.

**Negative / Trade-offs:**
- Cross-domain reporting requires an event-driven read model (`reports-service` subscribes to domain events to build projections).

**Impact on the System:**
- Affected Services: All 9 microservices.
- Documents Updated: `06-data/models.md`, `05-architecture/pattern-guide.md`.

---

#### References
- Data Models per Service → `06-data/models.md`
- Microservices Pattern Guide → `05-architecture/pattern-guide.md`
```

---

### 1.3 ADR-005: Message Broker Selection (RabbitMQ)

```markdown
# ADR-005 — Message Broker Selection (RabbitMQ)

| Field | Value |
| :--- | :--- |
| **ID** | ADR-005 |
| **Date** | 2026-09-17 |
| **Status** | Accepted |
| **Authors** | Fabian Trujillo — Project Leader |
| **Reviewers** | Tech Lead, Development Team |
| **Supersedes** | N/A (Resolves AT-002) |

---

#### Context
The `resi-complex` architecture relies on 16 domain events (e.g., `UnitRegistered`, `FeeGenerated`, `MaintenanceRequestCreated`, `CorrespondenceReceived`) to trigger asynchronous cross-context policies. To carry these event payloads reliably between microservices, a dedicated Message Broker is required.

**Known Constraints:**
- Must support publish-subscribe messaging with topic-based routing (`units.unit.registered`, `billing.fee.generated`, etc.).
- Must run efficiently in local Docker development environments with minimal resource consumption.
- Native integration with Java 17+ / Spring Boot 3.x (`spring-boot-starter-amqp`).

---

#### Decision
**We decided:** To select **RabbitMQ** (version 3.12+ with Management Plugin) using AMQP 0-9-1 as the official Message Broker for `resi-complex`. Events will be published to a single Topic Exchange named `resicomplex.events` with routing keys matching the domain event channels.

**Justification:**
1. **Flexible Topic Routing:** AMQP topic exchanges perfectly match the hierarchical routing keys used in our domain events catalog.
2. **Lightweight Footprint:** Low memory and CPU usage in Docker Compose compared to Kafka clusters.
3. **Built-in Queue Management:** Features native Dead Letter Queues (DLQ), message TTL, and a web management dashboard (`http://localhost:15672`).
4. **Spring Boot Ecosystem:** Flawless integration via `Spring AMQP` (`RabbitTemplate`, `@RabbitListener`).

---

#### Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: RabbitMQ (CHOSEN)** | Lightweight; native AMQP topic routing; DLQ support; low resource usage; excelente Spring integration. | Lower throughput limits compared to Kafka (irrelevant at residential complex scale). | — (Chosen) |
| **Option B: Apache Kafka** | Massive throughput; log replay capabilities. | High resource consumption (JVM + ZooKeeper/KRaft); steep setup complexity for local dev. | Over-engineered for single complex scale (< 500 RPS). |
| **Option C: Redis Streams** | Extremely fast; zero additional container if Redis is already used for caching. | Lacks native DLQ, sophisticated routing exchanges, and message acknowledgment management. | Lacks enterprise message broker features needed for auditability. |

---

#### Consequences
**Positive:**
- Fast local setup via standard `rabbitmq:3-management` Docker image.
- Out-of-the-box Dead Letter Exchanges for handling failed message retries.
- Web UI provides immediate visual debugging of queues and message rates.

**Negative / Trade-offs:**
- Message ordering across different queues is not guaranteed (mitigated by using `occurredAt` and `version` timestamps in event metadata).

**Impact on the System:**
- Affected Services: `units-service`, `people-service`, `maintenance-service`, `billing-service`, `communications-service`, `access-control-service`, `finance-approval-service`, `reports-service`.
- Documents Updated: `05-architecture/overview.md`, `02-domain/domain-events.md`.

---

#### References
- Domain Events Catalog & Topics → `02-domain/domain-events.md`
- Technical Debt AT-002 → `05-architecture/overview.md`
```

---

### 1.4 ADR-006: JWT-Based Authentication & API Gateway Strategy

```markdown
# ADR-006 — JWT Authentication & API Gateway Strategy

| Field | Value |
| :--- | :--- |
| **ID** | ADR-006 |
| **Date** | 2026-09-17 |
| **Status** | Accepted |
| **Authors** | Fabian Trujillo — Project Leader |
| **Reviewers** | Tech Lead, Development Team |
| **Supersedes** | N/A (Resolves AT-001) |

---

#### Context
`resi-complex` enforces Role-Based Access Control (RBAC) across 5 authentication roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`) and Ownership-Level Authorization (`:own` / `:assigned`). Exposing 9 microservices directly to clients creates security vulnerabilities, CORS configuration nightmare, and duplicate JWT verification logic in every service.

**Known Constraints:**
- JWT access tokens must expire in **1 hour**; refresh tokens valid for **7 days** (per `NFR-004`).
- All incoming HTTP traffic must pass through a single entry point (API Gateway).
- Java 17+ and Spring Boot stack compatibility.

---

#### Decision
**We decided:** To deploy **Spring Cloud Gateway** (Port 8080) as the central API Gateway. 
1. **Authentication:** `iam-service` issues signed JWT tokens upon login (`POST /auth/login`).
2. **Centralized Gateway Validation:** Spring Cloud Gateway validates incoming JWT signature and expiration.
3. **Context Propagation:** Upon successful validation, the Gateway extracts JWT claims and forwards them to downstream microservices via trusted HTTP headers:
   - `X-User-Id`: User's UUID
   - `X-User-Roles`: User's primary role
   - `X-User-Permissions`: Comma-separated list of granted permissions
4. **Ownership Authorization:** Downstream services evaluate ownership (`:own`, `:assigned`) using the `X-User-Id` header against resource ownership.

---

#### Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: Spring Cloud Gateway + IAM JWT (CHOSEN)** | Single tech stack (Java/Spring); native Reactive Netty performance; seamless JWT claim propagation headers; centralized CORS. | Requires learning Spring WebFlux reactive routing syntax. | — (Chosen) |
| **Option B: Kong API Gateway (Community)** | High performance C/Lua gateway; rich plugin ecosystem. | Separate tech stack; heavier plugin configuration; extra operational learning curve for Java team. | Adds unnecessary multi-language complexity to a pure Java/Spring project. |
| **Option C: Direct Microservice Exposure** | No Gateway container needed. | Duplicate JWT verification in 9 services; CORS nightmare; security risk if a service omits auth. | Violates security best practices and `NFR-004`. |

---

#### Consequences
**Positive:**
- Centralized CORS, Rate Limiting, and SSL termination at Port 8080.
- Downstream microservices do not need complex JWT verification libraries—they read trusted `X-User-*` headers.
- Internal microservices are hidden behind the Gateway network alias in Docker.

**Negative / Trade-offs:**
- API Gateway becomes a single point of failure (mitigated in production via horizontal scaling).

**Impact on the System:**
- Affected Services: `api-gateway`, `iam-service`, and all 8 downstream services.
- Documents Updated: `05-architecture/overview.md`, `04-requirements/non-functional.md`.
#### References
- Security Policy & Permission Suffixes → `00-governance/security-policy.md`
- Non-Functional Security Requirements → `04-requirements/non-functional.md`
```

```markdown
#### ADR Register

| # | Title | Status | Date |
| :--- | :--- | :--- | :--- |
| **ADR-001** | Documentation Language (English) | Accepted | 2026-08-28 |
| **ADR-002** | Language Exception for User-Research Artifacts | Accepted | 2026-09-16 |
| **ADR-003** | Microservices + Event-Driven Architectural Style | Accepted | 2026-09-17 |
| **ADR-004** | Database per Service Pattern | Accepted | 2026-09-17 |
| **ADR-005** | Message Broker Selection (RabbitMQ) | Accepted | 2026-09-17 |
| **ADR-006** | JWT Authentication & API Gateway Strategy | Accepted | 2026-09-17 |
```

---



> ADR-001 and ADR-002 are **governance/documentation** decisions, not architecture-style
> decisions. They live in this folder because ADR-002 explicitly references the path
> `05-architecture/decisions/records/ADR-001-idioma-documentacion.md`, and because they set the
> language rule every future ADR in this repository — including the architecture-specific ones
> below — must follow.

---

## Open gap: no architecture-style ADRs exist yet

`05-architecture/overview.md` and `05-architecture/pattern-guide.md` already describe and
justify several architectural decisions in prose (microservices + event-driven style, database
per service, REST + domain events, JWT via `iam-service`), but **none of them has been formalized
as its own ADR yet.** This is a real, tracked gap — not an oversight to paper over — flagged
explicitly so the next contributor (or the instructor, during the Architecture Review gate in
`00-sdd-guide.md`) knows exactly what to write next:

| # | Candidate title | What it would formalize | Where the justification already lives |
|---|-----------------|--------------------------|-----------------------------------------|
| ADR-003 | Microservices + Event-Driven Architectural Style | Why resi-complex is deployed as 9 services instead of a modular monolith | `05-architecture/overview.md` §1 |
| ADR-004 | Database per Service (MySQL, dedicated per context) | Why no service shares a schema with another | `02-domain/domain-map.md` (per-context table); `05-architecture/pattern-guide.md` |
| ADR-005 | Message Broker Selection (Kafka vs. RabbitMQ vs. Redis Streams) | Which broker carries the 16 events already specified in `02-domain/domain-events.md` | `05-architecture/overview.md` AT-002 (currently unresolved) |
| ADR-006 | JWT-Based Authentication Strategy | Why JWT + refresh tokens over server-side sessions | `01-context/overview.md`; `04-requirements/non-functional.md` NFR-004 |

**Recommended action:** write ADR-003 and ADR-004 before `06-data/models.md` is finalized (both
decisions are already load-bearing assumptions in every service's data model); write ADR-005
before any service starts publishing events in Sprint 3 (`03-product/vision.md` H2); write
ADR-006 whenever `07-api/` documents the authentication flow in OpenAPI.

---

## Correlations

- Architectural style, C4 diagrams, service catalog → `05-architecture/overview.md`
- Pattern-by-pattern adoption rationale → `05-architecture/pattern-guide.md`
- Team documentation conventions this folder must follow → `00-governance/documentation-rules.md`
- Domain term glossary (source of truth for terminology used across ADRs) → `01-context/glossary.md`
