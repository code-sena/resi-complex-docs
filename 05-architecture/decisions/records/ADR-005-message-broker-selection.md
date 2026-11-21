# ADR-005 — Message Broker Selection (RabbitMQ) & Transactional Outbox

| Field | Value |
| :--- | :--- |
| **ID** | ADR-005 |
| **Date** | 2026-09-17 *(reconciled 2026-11-21)* |
| **Status** | Accepted |
| **Authors** | Fabián Trujillo — Tech Lead & Project Leader |
| **Reviewers** | Alexandra Garzón, Félix Murcia, Julián Losada — Development Team |
| **Supersedes / Related to** | Resolves `AT-002`; complements `ADR-003` & `02-domain/domain-events.md` |

---

## Context

The `resi-complex` domain specifies **16 Domain Events** and **6 reactive cross-context policies** (`02-domain/domain-events.md`), including `UnitRegistered`, `PersonRegistered`, `MaintenanceRequestCreated`, `MaintenanceRequestStatusUpdated`, `FeeGenerated`, `FeePaid`, `CorrespondenceReceived`, `ExpenseProposalCreated/Approved/Rejected`, and `AnnouncementPublished`.

To deliver these events reliably without distributed two-phase commit (2PC) locks, the system requires a lightweight, Spring-native message broker paired with reliable publishing and idempotent consumption.

**Known Constraints:**
- Hierarchical topic routing (`units.unit.registered`, `billing.fee.generated`, `maintenance.request.status-updated`, etc.).
- Low CPU/RAM footprint for local Docker Compose execution on student/developer laptops.
- First-class Java / Spring Boot support (`spring-boot-starter-amqp`).
- Guaranteed at-least-once delivery with Dead Letter Queue (DLQ) inspection.

---

## Decision

**We decided:** To adopt **RabbitMQ 3.12+ (AMQP 0-9-1, Management Plugin enabled)** as the official message broker, paired with the **Transactional Outbox Pattern** and **Idempotent Consumers**:
1. **Exchange Topology:** Single durable Topic Exchange named **`resicomplex.events`**; routing keys follow `{context}.{entity}.{event-past-tense}` (`02-domain/domain-events.md`).
2. **Ports:** AMQP `5672`, Management UI `15672`.
3. **Reliable Publishing (`outbox`):** Every publishing service (`units`, `people`, `maintenance`, `billing`, `communications`, `access-control`, `finance-approval`, plus `iam` for `UserRegistered`/`UserActivated`) persists events in its local MySQL `outbox` table inside the same ACID transaction as its aggregate mutation; a relay worker publishes `PENDING` records to RabbitMQ and marks them `PUBLISHED`.
4. **Idempotent Consumption (`processed_events`):** Every consuming service (`billing`, `communications`, `access-control`, `reports`) records consumed `event_id` values in its local `processed_events` table inside the same local transaction as its projection/notification update, preventing duplicate side effects on redelivery.
5. **Failure Handling:** Failed consumer deliveries retry with exponential backoff and route to dedicated Dead Letter Exchanges/Queues (`*.dlq`) after max attempts.

**Justification:**
1. **Exact Routing Fit:** AMQP topic exchanges natively match our hierarchical routing keys and fan-out subscriptions (`communications-service` and `reports-service`).
2. **Lightweight Local Footprint:** Minimal RAM/CPU footprint in Docker Compose compared to Apache Kafka.
3. **Built-in Operational Tooling:** Native Dead Letter Queues, message TTL, acknowledgments, and web management dashboard (`http://localhost:15672`).
4. **Zero-Friction Spring Integration:** Native `RabbitTemplate` and `@RabbitListener` integration with Spring Boot.

---

## Evaluated Alternatives

| Alternative | Pros | Cons | Reason for Discarding |
| :--- | :--- | :--- | :--- |
| **Option A: RabbitMQ 3.12+ + Transactional Outbox (CHOSEN)** | Lightweight; native topic routing & DLQs; rich web UI; seamless Spring AMQP support | Lower maximum throughput than Kafka (irrelevant at single-complex scale < 500 RPS) | — *(Chosen)* |
| **Option B: Apache Kafka** | High throughput; durable log replay | Heavy JVM memory footprint; complex local broker/KRaft setup for 10-container dev laptops | Over-engineered for residential complex event volumes |
| **Option C: Redis Streams** | Fast; simple container | Lacks native AMQP topic exchanges, DLQ routing, and built-in management UI | Insufficient enterprise messaging semantics for audit/DLQ workflows |

---

## Consequences

**Positive:**
- Guaranteed at-least-once event delivery with zero dual-write inconsistency between MySQL and RabbitMQ.
- Clean local setup using `rabbitmq:3.12-management` container.
- Clear visibility of queues, bindings, and dead-lettered messages via Management UI.

**Negative / Trade-offs:**
- Strict global ordering across queues is not guaranteed (mitigated by `occurredAt` timestamps and version checks in event envelopes).
- Requires maintaining the `outbox` and `processed_events` tables in Flyway migrations (`06-data/models.md`).

---

## Risks

| Risk | Probability | Impact | Mitigation |
| :--- | :--- | :--- | :--- |
| Duplicate event delivery causes double notifications or skewed report counters | Medium | High | Mandatory `processed_events` primary-key deduplication in every consumer (`06-data/models.md`) |
| Poison-pill message blocks consumer queue | Low | Medium | Configure max retry count (3 attempts with exponential backoff) + Dead Letter Queue alerting |

---

## References

- Domain Events & Routing Keys → `02-domain/domain-events.md`
- Outbox & `processed_events` DDL → `06-data/models.md`
- Outbox Spring Boot Reference Implementation → `05-architecture/decisions/records/architecture-decisions-and-infrastructure.md` §3
- Resolved Technical Debt → `05-architecture/overview.md` (`AT-002`)
