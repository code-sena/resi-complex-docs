# Access Control Service (`resi-complex-access-control-service`)

> **Front Desk & Access Authority** of the system. Manages visitor admissions, vehicle entry/exit tracking (differentiating personal visits from commercial establishment clients), and front desk package/correspondence logging (`FR13`, `FR14`, `FR15`).
> Authoritative owner of `Visit` and `Correspondence` entities.

---

## Location in the Architecture

| Field | Value |
|-------|-------|
| Number in catalog | 09 |
| Service Name | `resi-complex-access-control-service` |
| Local port | `8087` |
| Repository | `https://github.com/resi-complex/resi-complex-access-control-service.git` |
| DB engine | MySQL 8.0 (`access_control_db`) |
| Architecture layer | Hexagonal / Layered (Controller → Service → Repository → Entity) |
| Tech stack | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway, Swagger/OpenAPI |
| Communicates with | Domain event bus (RabbitMQ) |
| Consumed by | API Gateway (:8080), Security Guard touch interface, Resident portal |

---

## Responsibilities (What this service DOES)

- **Visitor Admission Logging (FR13):** Security guards log incoming visitors by recording full name, government document number, destination unit ID, and visitor category (`PERSONAL_VISITOR` vs. `COMMERCIAL_CLIENT`).
- **Vehicle Access Control (FR14):** Records vehicle license plate and vehicle classification (`CAR`, `MOTORCYCLE`, `NONE`) tied to an incoming visit.
- **Active Visits Monitoring & Exit Tracking:** Provides real-time queries of visitors currently inside the complex (`/visits/active`) and timestamps their departure (`PATCH /visits/{id}/exit`).
- **Front Desk Correspondence & Packages (FR15):** Logs incoming mail and parcels (envelopes, packages, food deliveries) designated per unit, and tracks pickup delivery transitions (`PENDING_PICKUP` → `DELIVERED`).

## Out of Scope (What it does NOT do)

- **Does not operate physical hardware:** Electronic turnstiles, RFID gates, and barrier arms are digitalized as logs only; physical IoT automation is out of MVP scope.
- **Does not manage tenant rosters directly:** References to `destination_unit_id` are validated against `units-service` via API Gateway or asynchronous caches.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|--------|----------|-------------|--------|
| `POST` | `/api/v1/visits` | Register visitor admission (+ optional vehicle) | `201 Created` |
| `GET` | `/api/v1/visits/active` | Query visitors currently inside the complex | `200 OK` |
| `PATCH` | `/api/v1/visits/{id}/exit` | Record visitor departure timestamp | `200 OK` |
| `POST` | `/api/v1/correspondence` | Register incoming parcel or letter | `201 Created` |
| `GET` | `/api/v1/correspondence` | List pending parcels (filter by `?unitId={id}`) | `200 OK` |
| `PATCH` | `/api/v1/correspondence/{id}/deliver` | Mark package as collected by resident/merchant | `200 OK` |

Swagger UI is available locally at: `http://localhost:8087/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start the MySQL database
mysql -u root -p -e "CREATE DATABASE access_control_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and start the service
./mvnw spring-boot:run

# 3. Verify health & active visits
curl http://localhost:8087/api/v1/visits/active
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL, tables, and Mermaid ER diagram (`access_control_db`)
- [events.md](./events.md) — Published domain events (`VisitRegistered`, `CorrespondenceReceived`)
- [decisions.md](./decisions.md) — Design decisions (visit differentiation, state transitions)
- [runbook.md](./runbook.md) — Operational guidelines, monitoring, and troubleshooting
- [API Contract](../../../07-api/contracts/openapi/access-control-service.yaml)

---

## Environment Variables

| Variable | Description | Default / Example |
|----------|-------------|-------------------|
| `SERVER_PORT` | Local service port | `8087` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/access_control_db?useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | Database user | `root` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
