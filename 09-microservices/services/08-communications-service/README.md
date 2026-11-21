# Communications Service (`resi-complex-communications-service`)

> **Announcements & Segmented Broadcast Authority** of the system. Manages official administrative announcements, emergency alerts, and segmented communication targeting residential units, commercial establishments, or the entire complex (`FR12`).
> Authoritative owner of the `Announcement` entity.

---

## Location in the Architecture

| Field | Value |
|-------|-------|
| Number in catalog | 08 |
| Service Name | `resi-complex-communications-service` |
| Local port | `8082` |
| Repository | `https://github.com/resi-complex/resi-complex-communications-service.git` |
| DB engine | MySQL 8.0 (`communications_db`) |
| Architecture layer | Hexagonal / Layered (Controller → Service → Repository → Entity) |
| Tech stack | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway, Swagger/OpenAPI |
| Communicates with | Domain event bus (RabbitMQ) |
| Consumed by | API Gateway (:8080), Web Frontend (Resident / Merchant / Admin portals) |

---

## Responsibilities (What this service DOES)

- **Publish Announcements (FR12):** Allows administrators to publish announcements with title, content, target scope (`ALL`, `ONLY_RESIDENTIAL`, `ONLY_COMMERCIAL`), and publisher tracking.
- **Segmented Audience Filtering:** Returns announcements tailored to the viewer's unit classification (commercial premises see general and commercial announcements; residences see general and residential announcements).
- **Lifecycle Management:** Allows editing (`PUT`) and soft deletion (`DELETE`) of active announcements.
- **Audit & Traceability:** Maintains creation, update, and soft delete timestamps (`deleted_at`) for regulatory transparency.

## Out of Scope (What it does NOT do)

- **Does not manage authentication directly:** JWT token validation is offloaded to the API Gateway / `auth-service`.
- **Does not manage physical unit data:** Reference `unitType` is verified through token claims or inter-service contracts (`units-service`).
- **Does not send external SMS or WhatsApp:** Communication is natively in-app through the announcements board.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|--------|----------|-------------|--------|
| `POST` | `/api/v1/announcements` | Publish a new segmented announcement | `201 Created` |
| `GET` | `/api/v1/announcements` | List announcements (optional query: `?unitType=RESIDENTIAL/COMMERCIAL`) | `200 OK` |
| `GET` | `/api/v1/announcements/{id}` | Get detail of a specific announcement | `200 OK` |
| `PUT` | `/api/v1/announcements/{id}` | Update title, content, or scope of an announcement | `200 OK` |
| `DELETE` | `/api/v1/announcements/{id}` | Soft delete an announcement | `204 No Content` |
| `GET` | `/actuator/health` or `/health` | Service health status | `200 OK` |

Swagger UI is available locally at: `http://localhost:8082/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start the MySQL database
mysql -u root -p -e "CREATE DATABASE communications_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and start the service
./mvnw spring-boot:run

# 3. Verify health
curl http://localhost:8082/api/v1/announcements
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL, tables, and Mermaid ER diagram (`communications_db`)
- [events.md](./events.md) — Published domain events (`AnnouncementPublished`)
- [decisions.md](./decisions.md) — Design decisions (segmentation logic, Flyway versioning)
- [runbook.md](./runbook.md) — Operational guidelines, monitoring, and troubleshooting
- [API Contract](../../../07-api/contracts/openapi/communications-service.yaml)

---

## Environment Variables

| Variable | Description | Default / Example |
|----------|-------------|-------------------|
| `SERVER_PORT` | Local service port | `8082` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/communications_db?useSSL=false&serverTimezone=UTC` |
| `SPRING_DATASOURCE_USERNAME` | Database user | `root` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
