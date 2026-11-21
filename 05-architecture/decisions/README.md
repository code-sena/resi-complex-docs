# ADRs — Architecture Decision Records

> ADRs document important architectural decisions. **One file = one decision.**  
> Every ADR in this folder complies with `ADR-001` (English).

---

## How to create a new ADR

1. Copy `_template-adr.md` to `records/ADR-NNN-short-title.md` (e.g., `records/ADR-007-caching-strategy.md`).
2. Complete every section — especially **Evaluated Alternatives**, **Consequences**, and **Risks**.
3. Submit via Pull Request to `dev` for Tech Lead + Team review (`00-governance/git-conventions.md`).
4. Once `Accepted`, the record is **immutable** (never deleted; future changes create a new ADR that marks the old one `Superseded`).
5. Register the new ADR in the table below.

---

## Possible statuses

- `Proposed` — under technical review
- `Accepted` — approved and enforced across the system
- `Rejected` — evaluated and discarded (with rationale preserved)
- `Superseded` — replaced by a newer `ADR-NNN`

---

## ADR Register

| ID | Title | Status | Date | File |
|----|-------|--------|------|------|
| **ADR-001** | Documentation Language (English) | ✅ Accepted | 2026-08-28 | [ADR-001-idioma-documentacion.md](./records/ADR-001-idioma-documentacion.md) |
| **ADR-002** | Language Exception for Raw User-Research Artifacts | ✅ Accepted | 2026-09-16 | [ADR-002-language-exception-user-research.md](./records/ADR-002-language-exception-user-research.md) |
| **ADR-003** | Microservices + Event-Driven Architectural Style | ✅ Accepted | 2026-09-17 | [ADR-003-architectural-style.md](./records/ADR-003-architectural-style.md) |
| **ADR-004** | Database per Service Pattern (MySQL 8.0+) | ✅ Accepted | 2026-09-17 | [ADR-004-database-per-service.md](./records/ADR-004-database-per-service.md) |
| **ADR-005** | Message Broker Selection (RabbitMQ) & Transactional Outbox | ✅ Accepted | 2026-09-17 | [ADR-005-message-broker-selection.md](./records/ADR-005-message-broker-selection.md) |
| **ADR-006** | JWT Authentication & API Gateway Strategy | ✅ Accepted | 2026-09-17 | [ADR-006-jwt-authentication-strategy.md](./records/ADR-006-jwt-authentication-strategy.md) |

---

## Supporting Technical Reference

- [architecture-decisions-and-infrastructure.md](./records/architecture-decisions-and-infrastructure.md) — Consolidated infrastructure topology & Spring Boot Transactional Outbox implementation reference supporting `ADR-003`–`ADR-006`.

---

## Correlations

- System Architecture Overview → `05-architecture/overview.md`
- Deployment Topology → `05-architecture/deployment.md`
- Cross-Cutting Concerns → `05-architecture/cross-cutting.md`
- Security Threat Model → `05-architecture/security-threat-model.md`
