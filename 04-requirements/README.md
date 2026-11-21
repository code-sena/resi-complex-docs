# 04 — Requirements

> **What is this?** The formal, traceable specification of what `resi-complex` must do
> (Functional Requirements + Business Rules + BDD User Stories) and how well it must
> operate (Non-Functional Requirements), linked end-to-end through the Traceability Matrix (RTM).

---

## Why this section exists

Requirements are the contract between the development team, the SENA instructor, and stakeholders.
Without them:
- There is no objective way to verify completeness
- Scope changes have no baseline for impact analysis
- Automated tests have no acceptance oracle

---

## What is here

| File | Priority | Status | Contents |
|------|----------|--------|----------|
| `functional.md` | ⭐ | ✅ Complete | Canonical catalog of **22 Functional Requirements (`FR01`–`FR22`)** with MoSCoW prioritization, **16 Business Rules (`BR-01`–`BR-16`)**, legacy transition strategy, and MVP exclusions |
| `fr-to-service-mapping.md` | ⭐ | ✅ Complete | Canonical `FR01`–`FR22` → HU → Bounded Context → Microservice → Owner mapping and Cut 1 / Cut 2 / Cut 3 delivery dependencies |
| `user-stories.md` | ⭐ | ✅ Complete | **22 BDD User Stories (`HU-IAM-001` through `HU-REP-001`)** organized across 3 Cuts (7 + 11 + 4), each with happy-path and negative/edge Gherkin scenarios, DoD, story points, and real owners |
| `non-functional.md` | ⭐ | ✅ Complete | **9 measurable NFRs (`NFR-001`–`NFR-009`)** aligned to Java 25 + Spring Boot 4.0.8 + MySQL 8 + RabbitMQ, including WCAG 2.1 AA usability/accessibility (`NFR-009`) and ArchUnit fitness targets |
| `traceability-matrix.md` | ⭐ | ✅ Complete | End-to-end RTM linking **22 FRs**, **22 HUs**, **9 NFRs**, Vision KRs, test classes, services, real implementation status, and active gaps |
| `_template-hu.md` | — | ✅ Intact | Reference User Story template (do not modify without team consensus) |

---

## Correlations with other sections

| Section | Relationship |
|---------|--------------|
| `01-context/` | `scope.md` defines boundaries; `glossary.md` defines Ubiquitous Language used in `FR01`–`FR22` and `BR-01`–`BR-16` |
| `02-domain/` | Every `BR-*` maps directly to entity/aggregate invariants in `entities-and-rules.md` and events in `domain-events.md` |
| `03-product/` | `vision.md` defines `KR1`–`KR4` and delivery horizons (`H1`–`H3`) traced in `traceability-matrix.md` |
| `05-architecture/` | `NFR-001`–`NFR-009` drive `ADR-003`–`ADR-006` (microservices, MySQL 8 per service, RabbitMQ, JWT + API Gateway) |
| `06-data/` | `models.md` enforces `BR-*` invariants via MySQL 8 `CHECK`, `UNIQUE`, `FK`, `outbox`, and `processed_events` |
| `07-api/` | `FR01`–`FR22` define the OpenAPI 3.0.3 endpoints and payloads per microservice |
| `11-quality/` | `traceability-matrix.md` defines the mandatory test classes, BDD scenarios, ≥ 80% coverage, and ArchUnit gates |

---

## Requirements quality rules

- ❌ *"The system must be fast"* → Unverifiable. ✅ *"P95 latency < 300 ms under 50 RPS (`NFR-001`)."*
- ❌ *"The system must work on phones"* → Unverifiable. ✅ *"WCAG 2.1 AA via `axe-core` and responsive at 360px (`NFR-009`)."*
- ✅ Every requirement must be **specific, measurable, achievable, relevant, and verifiable (SMART)**, written in English (`ADR-001`), and synchronized with `06-data/models.md`.

---

## Questions this section answers

- **What must the system do for each of the 5 roles?** → `functional.md` (`FR01`–`FR22`, `BR-01`–`BR-16`) and `user-stories.md` (22 BDD HUs).
- **Under what speed, security, accessibility, and resilience metrics?** → `non-functional.md` (`NFR-001`–`NFR-009`).
- **Which microservice and teammate owns each FR?** → `fr-to-service-mapping.md` and `traceability-matrix.md`.
- **Which requirements are already in development vs. pending?** → `traceability-matrix.md` (live status synced to Phase 1 GitHub repos and identified gaps).
