# People Service (`resi-complex-people-service`)

> **People & Unit Occupancy Management Authority** of `resi-complex` (`FR21`, `BR-06`, `BR-07`, `BR-16`). Authoritative owner of `person` profiles (`RESIDENT`, `COMMERCIAL_OWNER`, `COMMERCIAL_TENANT`) and their formal `unit_occupancies` tenancy lifecycle (`OWNER` / `TENANT`, `ACTIVE` / `INACTIVE`), isolating personal data in `people_db` per Colombian **Ley 1581 de 2012 (Habeas Data)**.

---

## Location in the Architecture

| Field | Value |
|---|---|
| Number in catalog | 04 |
| Service Name | `resi-complex-people-service` |
| Local port | `8085` |
| Repository | `https://github.com/resi-complex/resi-complex-people-service.git` |
| DB engine | MySQL 8.0 (`people_db`: `person`, `unit_occupancies`, `outbox`) |
| Architecture layer | Hexagonal Architecture (Domain → Application Use Cases → Infrastructure Adapters) |
| Tech stack | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate (`ddl-auto=validate`), Flyway, OpenAPI 3.0.3 |
| Communicates with | RabbitMQ (`resicomplex.events` via Outbox), `units-service` (`:8083` logical `unitId` validation) |
| Consumed by | `api-gateway` (`:8080`), Web Frontend (Admin People Portal, Profile), downstream `:own` ownership checks (`billing`, `maintenance`, `access-control`) |

---

## Responsibilities (What This Service DOES)

- **Register Person Profiles (`FR21` / `HU-PPL-001`):** Registers residents, commercial owners, and commercial tenants inside a single `person` entity (`BR-06`) enforcing document uniqueness (`uq_person_document`).
- **Manage Unit Occupancy Lifecycle (`FR21` / `HU-PPL-002`):** Links persons to units via `unit_occupancies` (`OWNER` or `TENANT`, `start_date`, `end_date`) and transitions finished leases to `INACTIVE` (`BR-07`), immediately revoking `:own` access without erasing historical records.
- **Publish Domain Events:** Emits `PersonRegistered` and `OccupancyDeactivated` via the transactional `outbox` table (`ADR-005`).
- **Ley 1581 PII Isolation (`ADR-004`):** Restricts direct storage and querying of personal identity/contact data to `people_db`.

## Out of Scope (What It Does NOT Do)

- **Does not manage credentials, JWTs, or RBAC roles:** Owned exclusively by `iam-service` (`:8081`).
- **Does not manage physical units or commercial establishment profiles:** Owned exclusively by `units-service` (`:8083`).

---

## Local Startup

```bash
# 1. Create dedicated MySQL 8 database
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS people_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run service (Flyway runs schema migrations; Hibernate validates schema)
mvn spring-boot:run

# 3. Verify health
curl http://127.0.0.1:8085/health
```

---

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `SERVER_PORT` | HTTP listen port | `8085` |
| `DB_URL` | MySQL 8 JDBC URL (`people_db`) | `jdbc:mysql://localhost:3306/people_db?useSSL=false&serverTimezone=UTC` |
| `DB_USERNAME` | MySQL username | `resi_dev` |
| `DB_PASSWORD` | MySQL password (env/Vault) | *(required)* |
| `RABBITMQ_HOST` | RabbitMQ host | `localhost` |

---

## Documentation Links

- OpenAPI Contract → [`07-api/contracts/openapi/people-service.yaml`](../../../07-api/contracts/openapi/people-service.yaml)
- Canonical Data Model → [`06-data/models.md`](../../../06-data/models.md) (`people_db`)
- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
