# Maintenance Service (`resi-complex-maintenance-service`)

> **Maintenance Request Lifecycle Authority** of `resi-complex` (`FR05`, `FR06`, `BR-08`, `BR-09`). Manages private-unit and common-area maintenance requests from reporting through assignment, repair execution, and verified closure (`PENDING → ASSIGNED → IN_PROGRESS → WORK_COMPLETED ⇄ IN_PROGRESS → RESOLVED`), publishing domain events via Transactional Outbox to drive in-app alerts (`FR07`) and executive reports (`FR19`).

---

## Location in the Architecture

| Field | Value |
|---|---|
| Number in catalog | 05 |
| Service Name | `resi-complex-maintenance-service` |
| Local port | `8086` |
| Repository | `https://github.com/resi-complex/resi-complex-maintenance-service.git` |
| DB engine | MySQL 8.0 (`maintenance_db`: `maintenance_requests`, `outbox`) |
| Architecture layer | Hexagonal Architecture (Worked Reference Example in `05-architecture/hexagonal-architecture.md`) |
| Tech stack | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate (`ddl-auto=validate`), Flyway, OpenAPI 3.0.3 |
| Communicates with | RabbitMQ (`resicomplex.events` via Outbox), `people-service` (`:8085` `:own` active occupancy check) |
| Consumed by | `api-gateway` (`:8080`), Web Frontend (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF` portals) |

---

## Responsibilities (What This Service DOES)

- **Report Maintenance Requests (`FR05` / `HU-MAINT-001`):** Allows `PERSON` (with active occupancy for `PRIVATE_UNIT`) or `ADMINISTRATOR` to create repair tickets for private units or shared common areas (`BR-08`), emitting `MaintenanceRequestCreated` (urgent requests immediately alert staff via `communications-service`).
- **Assign & Track Lifecycle (`FR06` / `HU-MAINT-002`):** Allows `ADMINISTRATOR` to assign `MAINTENANCE_STAFF` (`PENDING → ASSIGNED`), and assigned technicians (`:assigned`) to progress `ASSIGNED → IN_PROGRESS → WORK_COMPLETED`.
- **Verified Closure Loop (`BR-09`):** Prevents technicians from closing tickets unilaterally — requires requester verification (`WORK_COMPLETED → RESOLVED`, optional 1–5 `resident_rating`) or rejection with `rejection_notes` (`WORK_COMPLETED → IN_PROGRESS`).
- **Event Publishing (`FR07`, `FR19`):** Emits `MaintenanceRequestCreated` and `MaintenanceRequestStatusUpdated` atomically via `maintenance_db.outbox`.

## Out of Scope (What It Does NOT Do)

- **Does not send notifications directly:** In-app notification delivery is handled reactively by `communications-service` (`:8082`, `FR07`).
- **Does not manage building budgets or extraordinary expenses:** Major capital approvals are owned by `finance-approval-service` (`:8088`).

---

## Local Startup

```bash
# 1. Create dedicated MySQL 8 database
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS maintenance_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run service (Flyway applies migrations; Hibernate validates schema)
mvn spring-boot:run

# 3. Verify health
curl http://127.0.0.1:8086/health
```

---

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `SERVER_PORT` | HTTP listen port | `8086` |
| `DB_URL` | MySQL 8 JDBC URL (`maintenance_db`) | `jdbc:mysql://localhost:3306/maintenance_db?useSSL=false&serverTimezone=UTC` |
| `DB_USERNAME` | MySQL username | `resi_dev` |
| `DB_PASSWORD` | MySQL password (env/Vault) | *(required)* |
| `RABBITMQ_HOST` | RabbitMQ host | `localhost` |

---

## Documentation Links

- OpenAPI Contract → [`07-api/contracts/openapi/maintenance-service.yaml`](../../../07-api/contracts/openapi/maintenance-service.yaml)
- Hexagonal Reference Example → [`05-architecture/hexagonal-architecture.md`](../../../05-architecture/hexagonal-architecture.md)
- Canonical Data Model → [`06-data/models.md`](../../../06-data/models.md) (`maintenance_db`)
- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
