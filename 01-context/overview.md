# System Overview

---

## What is Residential-complex?

Residential-complex (resi-complex) is a web-based information system that lets the administration of a mixed residential complex — one with both housing units and commercial establishments — manage everything from a single place: units, residents and shop owners/tenants, maintenance requests, differentiated administration fees, visitor and vehicle access, correspondence, segmented announcements, and Board-approved extraordinary expenses. It replaces manual, fragmented tools (notebooks, spreadsheets, WhatsApp) with one traceable digital record.

## Problem it solves

**Before the system:** Small and medium residential complexes manage these processes manually or across disconnected tools (notebooks, Excel, WhatsApp, physical logbooks). This causes duplicated or lost visitor records, difficulty tracking administration fees — especially with different rates for residential and commercial units — no traceability on maintenance requests, and no formal record of Board approvals for extraordinary expenses. The result is a lack of transparency toward residents and commercial tenants.

**With the system:** Units, fees, maintenance, access control, correspondence, and expense approvals are centralized and traceable. Residents and commercial owners/tenants get transparency into their own fees, requests, and approved expenses; the Board gets a permanent, auditable history of every approval or rejection; and the Administrator gets real-time information (pending fees, request status, approved expenses) to support decisions.

## Main users

| Role | Description | What they do in the system |
|------|-------------|--------------------------|
| Administrator | Staff member who runs the day-to-day administration of the complex | Manages units and people, generates fees, assigns maintenance requests, publishes announcements, creates extraordinary expense proposals |
| Board of Trustees | Elected body responsible for financial oversight of the complex | Approves or rejects expense proposals, reviews financial reports |
| Person (Resident / Commercial Owner or Tenant) | Occupant of a residential unit, or owner/tenant of a commercial unit | Creates maintenance requests, checks their fee status, views announcements segmented to their unit type, checks pending correspondence |
| Maintenance Staff | Staff member who resolves maintenance requests | Views assigned requests and updates their status (received → in progress → resolved) |
| Security Guard | Staff member responsible for the entrance/lobby (portería) | Logs visitor and vehicle entry/exit (personal visitors or commercial clients), logs incoming correspondence |

## Technology stack

| Layer | Technology | Justification |
|-------|-----------|---------------|
| Frontend | Responsive Web UI (HTML5/CSS3/JS + Design System in `12-ux-ui/` validated via Google Stitch; Thymeleaf/SPA-compatible REST client) | Mobile-usable down to `360px` viewport and WCAG 2.1 AA compliant (`NFR-009`), decoupled from backend microservices via the API Gateway |
| API Gateway | Spring Cloud Gateway (`:8080`, local fallback `:8090`) | Single entry point for centralized JWT verification, `X-User-*` header propagation, CORS, and rate limiting (`ADR-006`) |
| Backend | Java 25 + Spring Boot 4.0.8, Hexagonal Architecture (Ports & Adapters) across 9 microservices | Stack confirmed for SENA ADSO 3239188 (`01-context/scope.md`, `ADR-003`); enforces strict domain isolation and independent testability |
| Database | MySQL 8.0+ (`InnoDB`, `utf8mb4`), dedicated database per service (`Database per Service`) | Matches the relational, ACID nature of the domain (`ADR-004`, `06-data/models.md`), with Flyway migrations and `ddl-auto=validate` |
| Message broker | RabbitMQ 3.12+ (AMQP 0-9-1, Topic Exchange `resicomplex.events`, DLQ) + Transactional Outbox | Lightweight event-driven transport for the 16 domain events and reactive policies (`ADR-005`, `02-domain/domain-events.md`) |
| Infrastructure | Docker & Docker Compose (local MVP); Kubernetes-ready OCI containers (`NFR-007`) | Reproducible 12-Factor local setup (`05-architecture/deployment.md`, `10-devops/local-setup.md`) |

## Current status

- **Phase:** Active Phase 1 implementation (SDD documentation `00`–`15`, OpenAPI 3.0.3 contracts, and Google Stitch UX/UI established; 7 of 10 service repositories already implemented on `https://github.com/resi-complex`)
- **Current version:** v0.2.0-dev
- **Last release:** Phase 1 MVP Core & Operational Services in progress (`dev`)
- **Next milestone:** Complete remaining Cut 1–2 service repositories (`units-service`, `people-service`, `maintenance-service`), wire RabbitMQ + Outbox end-to-end, and reach ≥ 80% JaCoCo + ArchUnit quality gates (`NFR-006`)

## Project contacts

| Role | Name | GitHub / Responsibility |
|------|------|-------------------------|
| Tech Lead & IAM / Gateway Owner | Fabián Trujillo | `@Trujillo7915` — Architecture, `api-gateway`, `iam-service` |
| Core Units, People & Maintenance Owner | María Alexandra Garzón | `@AlexandraVargass` — `units-service`, `people-service`, `maintenance-service` |
| Core Billing, Finance & Reports Owner | Félix Murcia | `@FelixJRbyte` — `billing-service`, `finance-approval-service`, `reports-service` |
| Security, Access & Communications Owner | Julián Losada | `@julianlp-hub` — `access-control-service`, `communications-service`, Security & DevOps |
