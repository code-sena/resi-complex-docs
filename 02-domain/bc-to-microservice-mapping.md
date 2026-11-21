# Bounded Contexts → Microservices Mapping

This document bridges the gap between domain modeling (02-domain)
and service implementation (09-microservices).

## Mapping table

| # | Bounded Context | Canonical Service (`Port`) | Folder | Status | Owner |
|---|---|---|---|---|---|
| 1 | IAM | `iam-service` (`:8081`) *(alias `auth-service`)* | `09-microservices/services/02-auth-service/` | ✅ Complete | Fabián Trujillo |
| 2 | Units Management | `units-service` (`:8083`) | `09-microservices/services/03-units-service/` | ✅ Complete | María Alexandra Garzón |
| 3 | People Management | `people-service` (`:8085`) | `09-microservices/services/04-people-service/` | ✅ Complete | María Alexandra Garzón |
| 4 | Maintenance | `maintenance-service` (`:8086`) | `09-microservices/services/05-maintenance-service/` | ✅ Complete | María Alexandra Garzón |
| 5 | Billing | `billing-service` (`:8084`) | `09-microservices/services/06-billing-service/` | ✅ Complete | Félix Murcia |
| 6 | Financial Approval | `finance-approval-service` (`:8088`) | `09-microservices/services/07-finance-service/` | ✅ Complete | Félix Murcia |
| 7 | Communications | `communications-service` (`:8082`) | `09-microservices/services/08-communications-service/` | ✅ Complete | Julián Losada |
| 8 | Access Control | `access-control-service` (`:8087`) | `09-microservices/services/09-access-control-service/` | ✅ Complete | Julián Losada |
| 9 | Reports | `reports-service` (`:8089`) | `09-microservices/services/10-reports-service/` | ✅ Complete | Félix Murcia |

*(Edge Infrastructure: `api-gateway` (`:8080` / local `:8090`) lives in `09-microservices/services/01-api-gateway/`, owned by Fabián Trujillo.)*

## Cross-reference

### By Bounded Context (from `02-domain`)
- See `02-domain/domain-map.md` → section "3. Identified Bounded Contexts"
- Every Bounded Context maps 1:1 to a dedicated microservice and MySQL 8 database (`ADR-003`, `ADR-004`).

### By Service (toward `09-microservices`)
- Every service has its `09-microservices/services/NN-name/` folder containing all 5 standardized files: `README.md`, `data-model.md`, `decisions.md`, `events.md`, `runbook.md`.
- Executive catalog & ownership matrix → `09-microservices/service-catalog.md`.

## Relationships between services

See the table in `02-domain/domain-map.md` → section "3.4 Relationships table":

| From | To | Type | Channel |
|---|---|---|---|
| `iam-service` | `api-gateway` & All Services | OHS / PL | REST + JWT (`ADR-006`) |
| `units-service` | `billing-service` | U→D | RabbitMQ (`UnitRegistered`, `UnitUpdated`) + sync debt check (`BR-05`) |
| `units-service` | `access-control-service` | U→D | RabbitMQ (`UnitRegistered`) |
| `people-service` | `billing-service`, `maintenance-service`, `access-control-service` | U→D | RabbitMQ (`PersonRegistered`, `OccupancyDeactivated`) + REST `:own` check |
| `maintenance-service` | `communications-service`, `reports-service` | U→D | RabbitMQ (`MaintenanceRequestCreated/StatusUpdated`) |
| `billing-service`, `finance-approval-service`, `access-control-service` | `communications-service`, `reports-service` | U→D | RabbitMQ (`resicomplex.events`, `ADR-005`) |

---

**Created:** 2026-09-17  
**Last updated:** 2026-11-21