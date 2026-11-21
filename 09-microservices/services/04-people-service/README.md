# People Service (`resi-complex-people-service`)

> **Census & Unit Occupancy Authority** of the system. Authoritative owner of personal profiles (residents, commercial owners, and commercial tenants) and the tenancy/occupancy lifecycle of condominium units (`HU-PPL-001`, `HU-PPL-002`, `FR21`).

---

## Location in the Architecture

| Field | Value |
|---|---|
| Number in catalog | 04 |
| Service Name | `resi-complex-people-service` |
| Local port | `8084` |
| Repository | `https://github.com/code-sena/resi-complex-people-service.git` |
| DB engine | MySQL 8.0 (`people_db`) |
| Architecture layer | Hexagonal / Clean Architecture (Controller → Application Service → Domain Model → Outbox / Repository) |
| Tech stack | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway, Swagger/OpenAPI |
| Communicates with | Domain Event Broker (RabbitMQ), `units-service` (logical unit reference validation) |
| Consumed by | API Gateway (:8080), Web/Mobile Frontends (Resident, Merchant, Admin portals) |

---

## Responsibilities (What this service DOES)

- **Person Registry (HU-PPL-001 / FR21):** Manages the personal profiles of occupants classified into `RESIDENT`, `COMMERCIAL_OWNER`, and `COMMERCIAL_TENANT`, enforcing unique identification document (`document_id`).
- **Unit Occupancy Assignment (HU-PPL-001):** Links persons to units with explicit roles (`OWNER` or `TENANT`), active status (`ACTIVE`), and validity dates (`start_date`, `end_date`).
- **Tenancy Deactivation on Lease End (HU-PPL-002):** Deactivates occupancy (`status = 'INACTIVE'`, sets `end_date`), immediately revoking ownership-level privileges (`:own`) over unit resources (maintenance requests, visitor pre-authorization, correspondence) while preserving all historical records for audit.
- **Event-Driven Outbox:** Publishes reliable domain events (`PersonRegistered`, `OccupancyAssigned`, `OccupancyDeactivated`) to synchronize downstream bounded contexts (`billing-service`, `access-control-service`, `communications-service`, `maintenance-service`).

## Out of Scope (What it does NOT do)

- **Does not manage passwords or JWT credentials:** Credentials and authentication tokens are handled by `auth-service` / `iam-service`.
- **Does not manage physical unit specs:** Nomenclature and classifications are managed by `units-service`.
- **Does not collect administration fees:** Handled by `billing-service`.
- **Does not record gate entries/exits:** Handled by `access-control-service`.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|---|---|---|---|
| `GET` | `/health` | Service health status and database connectivity | `200 OK` |
| `GET` | `/api/v1/people` | List people with pagination and filters (`personType`, `unitId`, `search`) | `200 OK` |
| `POST` | `/api/v1/people` | Register person with initial active unit occupancy (`HU-PPL-001`) | `201 Created` |
| `GET` | `/api/v1/people/{id}` | Get person profile with complete occupancy history | `200 OK` |
| `PUT` | `/api/v1/people/{id}` | Update contact details (full name, phone, email) | `200 OK` |
| `DELETE` | `/api/v1/people/{id}` | Soft delete person profile | `204 No Content` |
| `GET` | `/api/v1/occupancies` | List occupancies with filters (`unitId`, `personId`, `status`) | `200 OK` |
| `POST` | `/api/v1/occupancies` | Create a new unit occupancy / lease record | `201 Created` |
| `GET` | `/api/v1/occupancies/{id}` | Get specific occupancy details | `200 OK` |
| `PATCH` | `/api/v1/occupancies/{id}/deactivate` | Deactivate occupancy upon lease termination (`HU-PPL-002`) | `200 OK` |
| `GET` | `/api/v1/units/{unitId}/occupants` | Get current active occupants for a unit | `200 OK` |

Swagger UI is available locally at: `http://localhost:8084/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start MySQL database and create the schema
mysql -u root -p -e "CREATE DATABASE people_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and launch the service
./mvnw spring-boot:run

# 3. Verify health
curl http://localhost:8084/health
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL schemas, tables, and Mermaid ER diagram (`people_db`)
- [events.md](./events.md) — Domain events (`PersonRegistered`, `OccupancyAssigned`, `OccupancyDeactivated`)
- [decisions.md](./decisions.md) — Architecture decision records (occupancy lifecycle, logical decoupling)
- [runbook.md](./runbook.md) — Operational procedures, metrics, troubleshooting, and alerts
- [API Contract](../../../07-api/contracts/openapi/people-service.yaml) — OpenAPI 3.0.3 specification

---

## Environment Variables

| Variable | Description | Default / Example |
|---|---|---|
| `SERVER_PORT` | Local service port | `8084` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/people_db?useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | Database username | `root` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
| `SERVICES_UNITS_URL` | Base URL of units-service | `http://localhost:8083` |
| `SPRING_RABBITMQ_HOST` | Message broker host | `localhost` |
