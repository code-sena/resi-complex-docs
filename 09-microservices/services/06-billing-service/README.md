# Billing Service (`resi-complex-billing-service`)

> **Core Domain — Administration Fee & Differentiated Tariff Authority** of the system. Manages monthly administration fee generation, differentiated tariff rules (`RESIDENTIAL` vs. `COMMERCIAL`), overdue calculation, unit account statements, and payment confirmation (`FR08`, `FR09`, `FR10`).
> Authoritative owner of the `Fee` entity.

---

## Location in the Architecture

| Field | Value |
|-------|-------|
| Number in catalog | 06 |
| Service Name | `resi-complex-billing-service` |
| Local port | `8084` |
| Repository | `https://github.com/resi-complex/resi-complex-billing-service.git` |
| DB engine | MySQL 8.0 (`billing_db`) |
| Architecture layer | Hexagonal / Layered (Controller → Service → Repository → Entity) |
| Tech stack | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate, Flyway, SpringDoc OpenAPI 3.0.3 |
| Communicates with | Domain event bus (RabbitMQ) |
| Consumed by | API Gateway (:8080), Web Frontend (Admin, Board, Resident, and Merchant portals) |

---

## Responsibilities (What this service DOES)

- **Monthly Fee Generation (`FR08`):** Generates monthly administration fees per unit (`unitId`, `billingMonth`, `billingYear`, `dueDate`) preventing duplicate emission for the same period.
- **Differentiated Tariff Application (`FR09`):** Enforces differentiated base tariffs according to unit classification (`RESIDENTIAL`: $250,000 COP default vs. `COMMERCIAL`: $450,000 COP default) and freezes `unitType` at generation time.
- **Account Statement & Arrears Calculation (`FR10`):** Computes real-time overdue status (`OVERDUE`), days overdue (`daysOverdue`), daily late fee surcharge (`lateFeeAmount`), and total balance due (`totalAmount`).
- **Payment Recording (`FR10`):** Processes fee payment confirmation (`PATCH /api/v1/fees/{id}/pay`), transitioning fee status to `PAID` and preventing duplicate payments.

## Out of Scope (What it does NOT do)

- **Does not integrate with external payment gateways:** Online card/PSE checkout gateways are excluded from MVP scope (`01-context/scope.md`); the service records payment references and confirmations.
- **Does not manage physical unit catalog:** `unitId` and `unitType` originate from `units-service`.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|--------|----------|-------------|--------|
| `POST` | `/api/v1/fees` | Generate monthly administration fee with differentiated tariff | `201 Created` |
| `GET` | `/api/v1/fees` | List all fees (optional query: `?status=PENDING\|PAID\|OVERDUE`) | `200 OK` |
| `GET` | `/api/v1/fees/{id}` | Get detail of a specific fee including late fee breakdown | `200 OK` |
| `GET` | `/api/v1/fees/unit/{unitId}` | List full account statement for a specific unit | `200 OK` |
| `PATCH` | `/api/v1/fees/{id}/pay` | Record payment confirmation for a fee | `200 OK` |
| `GET` | `/health` | Service health check | `200 OK` |

Swagger UI is available locally at: `http://localhost:8084/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start the MySQL database
mysql -u root -p -e "CREATE DATABASE billing_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and start the service
./mvnw spring-boot:run

# 3. Verify health & fees
curl http://localhost:8084/api/v1/fees
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL, tables, and Mermaid ER diagram (`billing_db`)
- [events.md](./events.md) — Published domain events (`FeeGenerated`, `FeePaid`)
- [decisions.md](./decisions.md) — Design decisions (differentiated tariffs, period uniqueness, Flyway validation)
- [runbook.md](./runbook.md) — Operational guidelines, monitoring, and troubleshooting
- [API Contract](../../../07-api/contracts/openapi/billing-service.yaml)

---

## Environment Variables

| Variable | Description | Default / Example |
|----------|-------------|-------------------|
| `SERVER_PORT` | Local service port | `8084` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/billing_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true` |
| `SPRING_DATASOURCE_USERNAME` | Database user | `resi_dev` |
| `DB_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
