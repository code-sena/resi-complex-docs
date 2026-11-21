# Communications Service (`resi-complex-communications-service`)

> **Announcements & Segmented Broadcast Authority** of the system. Manages official administrative announcements, emergency alerts, segmented communications (`FR11`), and reactive event-driven in-app notifications (`FR07`) targeting residential units, commercial establishments, or the entire complex.
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
| Tech stack | Java 25 (17+ baseline), Spring Boot 4.0.8 (3.x+ compatible), Spring Data JPA, Hibernate (`ddl-auto=validate`), Flyway, OpenAPI 3.0.3 |
| Communicates with | Domain event bus (RabbitMQ `resicomplex.events`) |
| Consumed by | API Gateway (`:8080`), Web Frontend (All 5 canonical roles) |

---

## Responsibilities (What this service DOES)

- **Publish Segmented Announcements (`FR11` / `HU-COMM-001`):** Allows `ADMINISTRATOR` to publish announcements with title, body, urgency (`INFORMATIVE`, `IMPORTANT`, `EMERGENCY`), target scope (`ALL`, `RESIDENTIAL_ONLY`, `COMMERCIAL_ONLY`), and publisher tracking.
- **Reactive In-App Notifications (`FR07` / `HU-MAINT-003`):** Consumes operational events idempotently (`processed_events`) from `maintenance-service`, `billing-service`, `access-control-service`, and `finance-approval-service` to deliver in-app user alerts.
- **Segmented Audience Filtering:** Returns announcements tailored to the viewer's unit classification (commercial premises see general and commercial announcements; residences see general and residential announcements).
- **Lifecycle Management & Audit:** Supports soft deletion (`deleted_at`) while preserving regulatory traceability.

## Out of Scope (What it does NOT do)

- **Does not manage authentication directly:** JWT token validation is offloaded to `api-gateway` / `iam-service`.
- **Does not manage physical unit data:** Reference `unitType` is verified through token claims or inter-service contracts (`units-service`).
- **Does not send external SMS, SMTP, or WhatsApp:** Communication is natively in-app through the announcements board and notification feed (`01-context/scope.md`).

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
