# Bounded Contexts → Microservices Mapping

This document bridges the gap between domain modeling (02-domain)
and service implementation (09-microservices).

## Mapping table

| # | Bounded Context | Microservice | Folder | Status | Owner |
|---|---|---|---|---|---|
| 1 | IAM | auth-service | `09-microservices/services/02-auth-service/` | ✅ Exists | - |
| 2 | Units Management | units-service | `09-microservices/services/03-units-service/` | ✅ Exists | María Alexandra Garzón |
| 3 | People Management | people-service | `09-microservices/services/04-people-service/` | 🔴 To create | TBD |
| 4 | Maintenance | maintenance-service | `09-microservices/services/05-maintenance-service/` | 🔴 To create | TBD |
| 5 | Billing | billing-service | `09-microservices/services/06-billing-service/` | 🔴 To create | TBD |
| 6 | Financial Approval | finance-service | `09-microservices/services/07-finance-service/` | 🔴 To create | TBD |
| 7 | Communications | communications-service | `09-microservices/services/08-communications-service/` | ✅ Exists | Julián Losada |
| 8 | Access Control | access-control-service | `09-microservices/services/09-access-control-service/` | ✅ Exists | Julián Losada |
| 9 | Reports | reports-service | `09-microservices/services/10-reports-service/` | 🔴 To create | TBD |

## Cross-reference

### By Bounded Context (from 02-domain)
- See `02-domain/domain-map.md` → section "3. Identified Bounded Contexts"
- Each BC is already defined; here it is mapped to its implementation

### By Service (toward 09-microservices)
- Each service must have a `09-microservices/services/NN-name/` folder
- It must contain: `README.md`, `data-model.md`, `decisions.md`, `events.md`, `runbook.md`
- Base it on the template: `09-microservices/_template/service/`

## Relationships between services

See the table in `02-domain/domain-map.md` → section "3.4 Relationships table"

| From | To | Type | Channel |
|---|---|---|---|
| auth-service | All | OHS/PL | REST (JWT) |
| units-service | billing-service | U→D | Async event |
| units-service | access-service | U→D | Async event |
| people-service | billing-service | U→D | Async event |
| billing-service | finance-service | U→D | Async event / REST |
| All | reports-service | OHS/PL | Async event |

---

**Created:** [2026-09-17]  
**Last updated:** [2026-09-17]