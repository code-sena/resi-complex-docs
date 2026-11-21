# Reports Service (`resi-complex-reports-service`)

> **Executive Analytics & CQRS Read-Projection Authority** of the system. Maintains pre-aggregated read views (`pending_fees_view`, `requests_by_status_view`) to serve sub-second financial arrears and maintenance KPIs to Administrators and Board members (`FR19`).
> Authoritative owner of `PendingFeesView` and `RequestsByStatusView` read projections.

---

## Location in the Architecture

| Field | Value |
|-------|-------|
| Number in catalog | 10 |
| Service Name | `resi-complex-reports-service` |
| Local port | `8089` |
| Repository | `https://github.com/resi-complex/resi-complex-reports-service.git` |
| DB engine | MySQL 8.0 (`reports_db`) |
| Architecture layer | CQRS Read Store / Layered (Controller → Service → Repository → Read Entity) |
| Tech stack | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate, Flyway, SpringDoc OpenAPI 3.0.3 |
| Communicates with | Domain event bus (RabbitMQ) |
| Consumed by | API Gateway (:8080), Web Frontend (Administrator and Board executive dashboards `/reports`) |

---

## Responsibilities (What this service DOES)

- **Portfolio & Critical Arrears Reporting (`FR19`):** Exposes aggregated portfolio balance (`totalPendingPortfolio`), total units with pending debt, and units in critical arrears (`overdueMonths > 2`).
- **Maintenance Status Breakdown (`FR19`):** Exposes real-time counts of maintenance requests grouped by status (`PENDING`, `IN_PROGRESS`, `RESOLVED`).
- **Read-Model Projection Updates:** Updates `pending_fees_view` and `requests_by_status_view` without coupling to operational databases.

## Out of Scope (What it does NOT do)

- **Does not mutate operational transactions:** Never writes to `billing_db` or `maintenance_db`.
- **Does not perform cross-database SQL JOINs:** Operates exclusively on its own denormalized projection tables in `reports_db`.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|--------|----------|-------------|--------|
| `GET` | `/api/v1/reports/fee-stats` | Get consolidated portfolio metrics and critical overdue units (>2 months) | `200 OK` |
| `POST` | `/api/v1/reports/fee-stats` | Upsert unit debt read projection | `200 OK` |
| `GET` | `/api/v1/reports/maintenance-stats` | Get maintenance request counts grouped by status | `200 OK` |
| `POST` | `/api/v1/reports/maintenance-stats` | Update maintenance status count projection | `200 OK` |
| `GET` | `/health` | Service health check | `200 OK` |

Swagger UI is available locally at: `http://localhost:8089/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start the MySQL database
mysql -u root -p -e "CREATE DATABASE reports_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and start the service
./mvnw spring-boot:run

# 3. Verify health & reports
curl http://localhost:8089/api/v1/reports/fee-stats
curl http://localhost:8089/api/v1/reports/maintenance-stats
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL, projection tables, and Mermaid ER diagram (`reports_db`)
- [events.md](./events.md) — Consumed domain events (`FeeGenerated`, `FeePaid`, `MaintenanceRequestCreated`, `MaintenanceRequestStatusUpdated`)
- [decisions.md](./decisions.md) — Design decisions (CQRS read projections, critical arrears threshold)
- [runbook.md](./runbook.md) — Operational guidelines, monitoring, and troubleshooting
- [API Contract](../../../07-api/contracts/openapi/reports-service.yaml)

---

## Environment Variables

| Variable | Description | Default / Example |
|----------|-------------|-------------------|
| `SERVER_PORT` | Local service port | `8089` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/reports_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true` |
| `SPRING_DATASOURCE_USERNAME` | Database user | `resi_dev` |
| `DB_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
