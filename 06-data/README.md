# 06 — Data

> **What is this?** How the system stores, structures, validates, and migrates data across
> its 9 dedicated MySQL 8 databases (`ADR-004`).

## Fundamental principle in microservices

> **Each microservice owns its own data (Database per Service).**

No service may access another service's database directly. Cross-service data is obtained
synchronously via REST API or asynchronously via Domain Events (`02-domain/domain-events.md`).

---

## What is here

### `models.md` ⭐ — Complete
Full MySQL 8 (`InnoDB`, `utf8mb4`) data models for all **9 microservices**, covering:
- **18 domain & read-projection tables** (`users`, `refresh_tokens`, `units`, `commercial_establishments`, `person`, `unit_occupancies`, `maintenance_requests`, `administration_fees`, `payments`, `announcements`, `visits`, `visitor_vehicles`, `visitor_preauthorizations`, `correspondence`, `expense_proposals`, `approvals`, `requests_by_status_view`, `pending_fees_view`)
- **2 cross-cutting event-resilience schemas** (`outbox` for the 7 publishing services and `processed_events` for the 4 consuming services)
- Explicit table-level `CONSTRAINT fk_*`, `uq_*`, `chk_*`, and `idx_*` definitions
- Inline data dictionaries, modeling decisions, 1NF–3NF/BCNF normalization assessment, Flyway + Expand-and-Contract migration strategy, safe seed-data policy, and full cross-service `erDiagram`

### Consolidated sections inside `models.md`
The following framework artifacts are consolidated inline inside `models.md` for this MVP scope and only need splitting into separate files if cross-service table counts expand significantly:
- **`data-dictionary.md`** → Per-service *Data dictionary* subsections in `models.md`
- **`modeling-conventions.md`** → *Data modeling principles* (`snake_case`, `CHAR(36)` UUIDs, UTC `TIMESTAMP`, `deleted_at` soft delete vs. append-only ledgers, table-level MySQL FKs)
- **`normalization-assessment.md`** → *Normalization Assessment* section (1NF–3NF/BCNF verification + CQRS read-model and `unit_type_at_generation` snapshot justification)
- **`migration-strategy.md`** → *Migration & Seed Data Strategy* section (Flyway `V{n}__` / `S{n}__` naming, forward-only rules, Expand & Contract zero-downtime protocol)

---

## Status

| Artifact | Status | Details |
|----------|--------|---------|
| `models.md` | ✅ Complete | 9 services (`iam-service` through `reports-service`), 9 MySQL 8 databases (`*_db`), 18 domain tables + `outbox` & `processed_events`, full `erDiagram`, Flyway + Expand-and-Contract strategy |
| `data-dictionary.md`, `modeling-conventions.md`, `normalization-assessment.md`, `migration-strategy.md` | ✅ Merged in `models.md` | Maintained inline without pending placeholders; extract into separate files only if schemas outgrow `models.md` |

---

## Correlations with other sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `02-domain/entities-and-rules.md` → domain entities & invariants | DB tables & `CHECK` constraints |
| `02-domain/domain-events.md` → 16 events & idempotent delivery | `outbox` & `processed_events` tables |
| `05-architecture/` (`ADR-004`, `ADR-005`, `ADR-006`) | Engine choice, Outbox schema & `refresh_tokens` |
| `models.md` | `07-api/contracts/` → exposed API schemas |
| `models.md` | `08-uml/` → ER diagrams |
| `models.md` | `09-microservices/services/NN/data-model.md` |

---

## Key data architecture decisions

### Why SQL (MySQL 8) across all 9 services?
Every entity in `02-domain/entities-and-rules.md` has a fixed relational schema requiring ACID guarantees (e.g., fee + payment ledger integrity, transactional `outbox` commit). Neither document stores (MongoDB), time-series engines (InfluxDB), nor full-text search engines (Elasticsearch) are needed in MVP scope.

### How is cross-service consistency guaranteed?
Without shared databases or distributed 2PC locks, consistency is **eventual**:
- **Transactional Outbox (`outbox`):** Guarantees at-least-once event publishing atomically with state changes (`05-architecture/decisions/records/architecture-decisions-and-infrastructure.md` §3.1).
- **Idempotent Consumers (`processed_events`):** Prevents duplicate side effects upon message redelivery (`02-domain/domain-events.md`).
- **Compensating flows (Saga):** Available in `05-architecture/pattern-guide.md` if future multi-step workflows require rollback.

---

## Questions this section answers

- **What data does each microservice own?** → `models.md`, one `## Service:` section per microservice (18 domain tables + event-resilience tables across 9 dedicated `*_db` MySQL 8 schemas).
- **Why MySQL 8 for all 9?** → Relational integrity, ACID transactions, and stack lock (`01-context/scope.md`, `ADR-004`).
- **How do schemas evolve safely?** → Flyway forward-only versioned migrations + 4-step Expand & Contract for breaking changes.
- **How are cross-service references enforced?** → Logical `CHAR(36)` references validated via REST/events, never cross-database SQL `FOREIGN KEY` constraints.
