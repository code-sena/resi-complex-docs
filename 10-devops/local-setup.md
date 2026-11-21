# Local Project Setup — resi-complex

> This guide enables any developer on the SENA ADSO 3239188 team to run the full `resi-complex`
> microservices stack locally in under 30 minutes.
> **Aligned with:** `05-architecture/deployment.md`, `09-microservices/service-catalog.md`, `ADR-004`, `ADR-005`, `ADR-006`.
> **Last Synchronized:** 2026-11-21

---

## 1. Prerequisites

| Tool | Minimum Version | Verify Command |
|------|-----------------|----------------|
| **Docker & Docker Compose** | Docker 24.0+ / Compose 2.20+ | `docker --version && docker compose version` |
| **Java JDK** | **Java 25** (17+ minimum baseline) | `java -version` |
| **Apache Maven** | 3.9+ (or repo `./mvnw`) | `mvn -version` |
| **Git** | 2.40+ | `git --version` |
| **Node.js (optional — docs linting)** | 20+ (for `npx @redocly/cli lint`) | `node --version` |

**Recommended:** IntelliJ IDEA or VS Code (Extension Pack for Java), Postman/Insomnia, MySQL Workbench or Adminer (`:8091`).

---

## 2. Canonical Local Port Allocation

Verify these ports are free before starting (`netstat -ano | findstr :8081` on Windows or `lsof -i :8081` on macOS/Linux):

| Port | Component / Service | Database / Purpose |
|------|---------------------|--------------------|
| **`8080`** *(local fallback **`8090`**)* | `api-gateway` | Single HTTP entry point (`SERVER_PORT=8090` bare-metal Windows fallback if `:8080` is occupied) |
| **`8081`** | `iam-service` | MySQL `iam_db` — Auth, 5 RBAC roles, JWT & refresh tokens (`FR01`, `FR20`) |
| **`8082`** | `communications-service` | MySQL `communications_db` — Segmented announcements & reactive alerts (`FR07`, `FR11`) |
| **`8083`** | `units-service` | MySQL `units_db` — Residential/commercial units & establishments (`FR02`–`FR04`) |
| **`8084`** | `billing-service` | MySQL `billing_db` — Differentiated fees & partial payments (`FR08`–`FR10`) |
| **`8085`** | `people-service` | MySQL `people_db` — Person profiles & unit occupancies (`FR21`) |
| **`8086`** | `maintenance-service` | MySQL `maintenance_db` — Maintenance request lifecycle (`FR05`–`FR06`) |
| **`8087`** | `access-control-service` | MySQL `access_control_db` — Visits, vehicles, pre-auths & correspondence (`FR12`–`FR16`, `FR22`) |
| **`8088`** | `finance-approval-service` | MySQL `finance_approval_db` *(local alias `finance_db`)* — Board voting (`FR17`–`FR18`) |
| **`8089`** | `reports-service` | MySQL `reports_db` — CQRS executive projections (`FR19`) |
| **`3306`** | **MySQL 8.0** | Dedicated logical schemas (`*_db`) managed via Flyway (`ADR-004`) |
| **`5672`** / **`15672`** | **RabbitMQ 3.12+** | AMQP broker (`resicomplex.events`) + Management UI (`ADR-005`) |
| **`8091`** | Adminer *(dev-only)* | Browser-based MySQL 8 database inspector (moved off `:8090` to avoid Gateway fallback conflict) |

---

## 3. Step-by-Step Setup

### Step 1: Clone Required Repositories
```bash
# Documentation Single Source of Truth
git clone https://github.com/code-sena/resi-complex-docs.git

# Example microservice repositories (org: https://github.com/resi-complex)
git clone https://github.com/resi-complex/resi-complex-api-gateway.git
git clone https://github.com/resi-complex/resi-complex-iam-service.git
```

### Step 2: Configure Environment Variables
```bash
cp .env.example .env
```

**Core variables (never commit `.env`):**

| Variable | Description | Example Local Value |
|----------|-------------|---------------------|
| `SERVER_PORT` | Microservice HTTP port | `8081` (`8080` or `8090` for Gateway) |
| `DB_URL` | MySQL 8 JDBC URL for the service's own schema | `jdbc:mysql://localhost:3306/iam_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true` |
| `DB_USERNAME` | MySQL username | `resi_dev` |
| `DB_PASSWORD` | MySQL password (set locally, Vault in prod) | `<local-dev-password>` |
| `JWT_SECRET` | Shared HS256 secret (≥ 256 bits) between `iam-service` & `api-gateway` | Generate via `openssl rand -base64 48` |
| `RABBITMQ_HOST` / `RABBITMQ_PORT` | RabbitMQ AMQP connection | `localhost` / `5672` |

### Step 3: Start Shared Infrastructure (MySQL 8 + RabbitMQ)
```bash
docker compose up -d mysql rabbitmq
docker compose ps
```
Ensure all 9 logical schemas (`iam_db`, `communications_db`, `units_db`, `billing_db`, `people_db`, `maintenance_db`, `access_control_db`, `finance_approval_db`, `reports_db`) exist with `CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci`.

### Step 4: Build, Test & Run a Microservice
```bash
# Run unit & Testcontainers tests (Flyway runs automatically; Hibernate enforces ddl-auto=validate)
mvn test

# Start the microservice
mvn spring-boot:run
```

### Step 5: Verify Health Endpoints
```bash
curl http://127.0.0.1:8080/health   # or :8090 on Windows bare-metal fallback
curl http://127.0.0.1:8081/health   # iam-service
```

---

## 4. Local Operational URLs

| Resource | URL |
|----------|-----|
| API Gateway | `http://127.0.0.1:8080` *(Windows bare-metal fallback: `http://127.0.0.1:8090`)* |
| RabbitMQ Management UI | `http://127.0.0.1:15672` |
| Adminer (MySQL 8 UI) | `http://127.0.0.1:8091` |
| OpenAPI Contract Linting | `npx @redocly/cli lint` (from `resi-complex-docs` root) |

---

## 5. Common Troubleshooting

- **Port `8080` already in use on Windows:** Run `resi-complex-api-gateway` with `SERVER_PORT=8090` (default in its `application.properties`).
- **Hibernate `Schema-validation` failure on boot:** Confirm the dedicated MySQL schema exists and Flyway (`spring.flyway.enabled=true`) executed cleanly without manual table edits.
- **401 Unauthorized across services:** Confirm `JWT_SECRET` is identical in `iam-service` (`:8081`) and `api-gateway`.
