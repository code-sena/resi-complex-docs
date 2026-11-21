# Units Service (`resi-complex-units-service`)

> **Physical Units & Commercial Establishment Authority** of the system. Authoritative owner of physical condominium spaces, their classification as residential or commercial units, and the 1:1 business profile attached to commercial units (`FR02`, `FR03`, `FR04`).

---

## Location in the Architecture

| Field | Value |
|---|---|
| Number in catalog | 03 |
| Service Name | `resi-complex-units-service` |
| Local port | `8083` |
| Repository | `https://github.com/code-sena/resi-complex-units-service.git` |
| DB engine | MySQL 8.0 (`units_db`) |
| Architecture layer | Hexagonal / Clean Architecture (Controller → Application Service → Domain Model → Outbox / Repository) |
| Tech stack | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway, Swagger/OpenAPI |
| Communicates with | Domain Event Broker (RabbitMQ), `billing-service` (sync HTTP check for unpaid fees prior to deletion) |
| Consumed by | API Gateway (:8080), Web/Mobile Frontends (Admin, Resident, Merchant portals) |

---

## Responsibilities (What this service DOES)

- **Register Units (FR02 / HU-UNIT-001):** Allows administrators to register physical units with unique nomenclature (`unit_number`) and classification (`RESIDENTIAL` or `COMMERCIAL`).
- **Edit & Soft-Delete Units (FR03 / HU-UNIT-002):** Enables updating unit metadata and executing logical deletion (`deleted_at`), blocking removal if the unit has active or overdue administration fees (`BR-05`).
- **Commercial Establishment Management (FR04 / HU-UNIT-003):** Attaches, stores, and updates the 1:1 business profile (`business_name`, `business_type`, `opening_time`, `closing_time`) exclusively for `COMMERCIAL` units.
- **Event-Driven Outbox:** Publishes reliable domain events (`UnitRegistered`, `UnitUpdated`, `CommercialEstablishmentRegistered`) to feed downstream bounded contexts (`billing-service`, `access-control-service`, `communications-service`, `reports-service`).

## Out of Scope (What it does NOT do)

- **Does not manage people or tenancies:** Resident, tenant, and lease lifecycles are owned strictly by `people-service`.
- **Does not calculate administration fees:** Fee tariffs and collection are owned by `billing-service`.
- **Does not grant physical access:** Gate, visitor, and vehicle validations are owned by `access-control-service`.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|---|---|---|---|
| `GET` | `/health` | Service health status and database connectivity | `200 OK` |
| `GET` | `/api/v1/units` | List units with pagination and filtering (`unitType`, `search`) | `200 OK` |
| `POST` | `/api/v1/units` | Register new residential or commercial unit (`FR02`) | `201 Created` |
| `GET` | `/api/v1/units/{id}` | Get complete detail of a unit by ID | `200 OK` |
| `PUT` | `/api/v1/units/{id}` | Update unit metadata (`FR03`) | `200 OK` |
| `DELETE` | `/api/v1/units/{id}` | Soft delete unit (`FR03`), blocked if debt exists (`BR-05`) | `204 No Content` |
| `GET` | `/api/v1/units/{id}/commercial` | Get commercial establishment details (`FR04`) | `200 OK` |
| `POST` | `/api/v1/units/{id}/commercial` | Register 1:1 establishment for commercial unit (`FR04`) | `201 Created` |
| `PUT` | `/api/v1/units/{id}/commercial` | Update commercial establishment profile (`FR04`) | `200 OK` |

Swagger UI is available locally at: `http://localhost:8083/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start the MySQL database and create the schema
mysql -u root -p -e "CREATE DATABASE units_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and launch the service
./mvnw spring-boot:run

# 3. Verify health
curl http://localhost:8083/health
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL schemas, tables, and Mermaid ER diagram (`units_db`)
- [events.md](./events.md) — Domain events (`UnitRegistered`, `UnitUpdated`, `CommercialEstablishmentRegistered`)
- [decisions.md](./decisions.md) — Architecture decision records (1:1 constraint, soft deletes, outbox pattern)
- [runbook.md](./runbook.md) — Operational guidelines, metrics, troubleshooting, and alerts
- [API Contract](../../../07-api/contracts/openapi/units-service.yaml) — OpenAPI 3.0.3 specification

---

## Environment Variables

| Variable | Description | Default / Example |
|---|---|---|
| `SERVER_PORT` | Local service port | `8083` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/units_db?useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | Database username | `root` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
| `SERVICES_BILLING_URL` | Base URL of billing service for fee balance verification | `http://localhost:8085` |
| `SPRING_RABBITMQ_HOST` | Message broker host | `localhost` |
