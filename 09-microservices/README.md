# 09 — Microservices

> **What is this?** The complete operational and technical specification for `api-gateway` and all
> **9 domain microservices** of `resi-complex`. Each service has its own folder (`services/01-*` through
> `services/10-*`) with 5 standardized artifacts: `README.md`, `data-model.md`, `events.md`,
> `decisions.md`, and `runbook.md`.

---

## Service Directory Structure (10 / 10 Services Complete)

| Folder | Canonical Service | Port | Dedicated DB | OpenAPI Contract |
|--------|-------------------|------|--------------|------------------|
| [`services/01-api-gateway/`](./services/01-api-gateway/README.md) | `api-gateway` | `8080` *(local `:8090`)* | None (stateless) | `api-gateway.yaml` |
| [`services/02-auth-service/`](./services/02-auth-service/README.md) | `iam-service` *(alias `auth-service`)* | `8081` | `iam_db` | `auth-service.yaml` |
| [`services/03-units-service/`](./services/03-units-service/README.md) | `units-service` | `8083` | `units_db` | `units-service.yaml` |
| [`services/04-people-service/`](./services/04-people-service/README.md) | `people-service` | `8085` | `people_db` | `people-service.yaml` |
| [`services/05-maintenance-service/`](./services/05-maintenance-service/README.md) | `maintenance-service` | `8086` | `maintenance_db` | `maintenance-service.yaml` |
| [`services/06-billing-service/`](./services/06-billing-service/README.md) | `billing-service` | `8084` | `billing_db` | `billing-service.yaml` |
| [`services/07-finance-service/`](./services/07-finance-service/README.md) | `finance-approval-service` *(alias `finance-service`)* | `8088` | `finance_approval_db` *(alias `finance_db`)* | `finance-service.yaml` |
| [`services/08-communications-service/`](./services/08-communications-service/README.md) | `communications-service` | `8082` | `communications_db` | `communications-service.yaml` |
| [`services/09-access-control-service/`](./services/09-access-control-service/README.md) | `access-control-service` | `8087` | `access_control_db` | `access-control-service.yaml` |
| [`services/10-reports-service/`](./services/10-reports-service/README.md) | `reports-service` | `8089` | `reports_db` | `reports-service.yaml` |

---

## Core Governance Files in This Folder

- [`service-catalog.md`](./service-catalog.md) ⭐ — Executive inventory of all 10 services, topology diagram, port/DB/repo map, and Cross-Service Entity Ownership Matrix.
- [`_template/`](./_template/README.md) — Reference scaffolding for any future v2.0 microservice (e.g., `amenities-service`).

---

## Mandatory Service Boundary Rules

Every microservice in `resi-complex` MUST obey these 5 boundary invariants (verified via ArchUnit and PR reviews):
1. **No Shared Database (`ADR-004`):** Never connect to or query another service's MySQL schema.
2. **Contract-First Communication (`P1`):** Synchronous calls must conform to `07-api/contracts/openapi/*.yaml`.
3. **Reliable Async Events (`ADR-005`):** Publish domain events exclusively via the local transactional `outbox` table and deduplicate consumed events via `processed_events`.
4. **Hexagonal Purity (`NFR-006`):** Domain packages must remain pure Java without Spring/JPA annotations.
5. **Ownership-Level Authorization (`P5`):** Every `:own` / `:assigned` endpoint enforces resource ownership inside its Application Use Case layer.

---

## Correlations with Other Sections

| Upstream / Peer Section | Link to `09-microservices/` |
|-------------------------|-----------------------------|
| `02-domain/domain-map.md` & `domain-events.md` | 1:1 Bounded Context mapping and event catalog in `services/*/events.md` |
| `05-architecture/overview.md` & `ADR-003`–`ADR-006` | System topology, ports, and decision foundations |
| `06-data/models.md` | Authoritative MySQL 8 DDL mirrored in `services/*/data-model.md` |
| `07-api/contracts/openapi/` | Formal API contracts implemented by each service |
| `10-devops/local-setup.md` & `13-operations/` | Local execution ports and consolidated runbooks |
