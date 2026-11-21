# IAM / Auth Service (`resi-complex-iam-service`)

> **Identity & Access Management Authority** of `resi-complex` (`FR01`, `FR20`, `ADR-006`). Manages user accounts across the **5 canonical authentication roles** (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`), BCrypt credential verification, account lockout (`BR-02`), staff provisioning/activation (`FR20`), 1-hour JWT access tokens, and 7-day single-use rotating refresh tokens with family reuse detection (`BR-03`).

---

## Location in the Architecture

| Field | Value |
|---|---|
| Number in catalog | 02 *(folder alias: `02-auth-service`; canonical service ID: `iam-service`)* |
| Service Name | `resi-complex-iam-service` |
| Local port | `8081` |
| Repository | `https://github.com/resi-complex/resi-complex-iam-service.git` |
| DB engine | MySQL 8.0 (`iam_db`: `users`, `refresh_tokens`, `outbox`) |
| Architecture layer | Hexagonal Architecture (Ports & Adapters: Domain → Application Use Cases → Infrastructure Adapters) |
| Tech stack | Java 25, Spring Boot 4.0.8, Spring Security, BCrypt, Spring Data JPA, Hibernate (`ddl-auto=validate`), Flyway (`V1__init_iam.sql`), OpenAPI 3.0.3 |
| Communicates with | RabbitMQ (`resicomplex.events` via Transactional Outbox) |
| Consumed by | `api-gateway` (`:8080` / local `:8090`), Web Frontend (Login, Activation, Admin User Management) |

---

## Responsibilities (What This Service DOES)

- **User Authentication (`FR01` / `HU-IAM-001`):** Validates credentials via BCrypt (cost ≥ 12), enforces temporary lockout after 5 consecutive failed attempts (`15 min`, `BR-02`), and issues a 1-hour signed JWT + 7-day opaque refresh token.
- **Rotating Refresh Tokens (`BR-03`):** Stores BCrypt-hashed refresh tokens in `iam_db.refresh_tokens`, rotates on every `/api/v1/auth/refresh`, and revokes the entire token family immediately if reuse of a rotated token is detected.
- **Staff Provisioning & User Activation (`FR20` / `HU-IAM-002`):** Allows `ADMINISTRATOR` to provision privileged accounts (`BOARD`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`, `ADMINISTRATOR`) and issue secure activation flows (`BR-01`).
- **RBAC Role Authority (`00-governance/security-policy.md`):** Enforces the exact 5 authentication roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`) and embeds canonical permissions in JWT claims.

## Out of Scope (What It Does NOT Do)

- **Does not store resident PII or unit occupancies:** Full profile data (`document_id`, `phone`, `person_type`, `unit_occupancies`) is strictly isolated in `people-service` (`people_db`, Ley 1581 de 2012).
- **Does not evaluate resource ownership (`:own` / `:assigned`):** Downstream microservices verify resource ownership inside their Application layer (`P5`).

---

## Local Startup

```bash
# 1. Ensure MySQL 8 has the dedicated database
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS iam_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run the service (Flyway runs V1__init_iam.sql automatically; Hibernate validates schema)
mvn spring-boot:run

# 3. Execute automated hexagonal test suite (3/3 tests passing)
mvn test

# 4. Verify health
curl http://127.0.0.1:8081/health
```

---

## Environment Variables

| Variable | Description | Example / Default |
|---|---|---|
| `SERVER_PORT` | HTTP listen port | `8081` |
| `DB_URL` | MySQL 8 JDBC URL | `jdbc:mysql://localhost:3306/iam_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true` |
| `DB_USERNAME` | MySQL user for `iam_db` | `resi_dev` |
| `DB_PASSWORD` | MySQL password (injected via env/Vault — never hardcoded) | *(required)* |
| `JWT_SECRET` | HS256 signing key (≥ 256 bits; matches `api-gateway`) | *(required)* |
| `JWT_ACCESS_EXPIRATION_SECONDS` | Access token TTL | `3600` (1 hour) |
| `JWT_REFRESH_EXPIRATION_DAYS` | Refresh token TTL | `7` (7 days) |
| `IAM_MAX_FAILED_ATTEMPTS` | Failed logins before lockout (`BR-02`) | `5` |
| `IAM_LOCKOUT_MINUTES` | Temporary lockout duration | `15` |

---

## Documentation Links

- OpenAPI Contract → [`07-api/contracts/openapi/auth-service.yaml`](../../../07-api/contracts/openapi/auth-service.yaml)
- Canonical Data Model → [`06-data/models.md`](../../../06-data/models.md) (`iam_db`)
- Architecture Decision → [`ADR-006`](../../../05-architecture/decisions/records/ADR-006-jwt-authentication-strategy.md)
- [data-model.md](./data-model.md) · [events.md](./events.md) · [decisions.md](./decisions.md) · [runbook.md](./runbook.md)
