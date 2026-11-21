# Finance Approval Service (`resi-complex-finance-approval-service`)

> **Financial Governance & Board Voting Authority** of the system. Manages extraordinary expense proposals, append-only Board member voting (`APPROVED` / `REJECTED`), quorum verification, and automated proposal resolution (`FR17`, `FR18`).
> Authoritative owner of `ExpenseProposal` and `ExpenseVote` entities.

---

## Location in the Architecture

| Field | Value |
|-------|-------|
| Number in catalog | 07 |
| Service Name | `resi-complex-finance-approval-service` |
| Local port | `8088` |
| Repository | `https://github.com/resi-complex/resi-complex-finance-approval-service.git` |
| DB engine | MySQL 8.0 (`finance_db`) |
| Architecture layer | Hexagonal / Layered (Controller → Service → Repository → Entity) |
| Tech stack | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate, Flyway, SpringDoc OpenAPI 3.0.3 |
| Communicates with | Domain event bus (RabbitMQ) |
| Consumed by | API Gateway (:8080), Web Frontend (Administrator and Board Member portals) |

---

## Responsibilities (What this service DOES)

- **Extraordinary Expense Proposals (`FR17`):** Allows administrators to create extraordinary expense proposals (`title`, `description`, `estimatedAmount`, `proposedBy`) initialized in `PENDING_REVIEW` status.
- **Immutable Board Voting (`FR18`):** Records append-only votes (`APPROVED` or `REJECTED`) with comments from Board members (`BOARD_MEMBER`), enforcing one vote per member per proposal.
- **Automated Quorum Resolution (`FR18`):** Automatically transitions proposals to `APPROVED` or `REJECTED` and timestamps `approvalDate` once the quorum threshold is reached, blocking subsequent votes.

## Out of Scope (What it does NOT do)

- **Does not execute bank transfers or supplier disbursements:** Tracks governance approval and audit trails only.
- **Does not manage Board member identities:** `boardMemberId` and `proposedBy` are logical references to `auth-service` / `people-service`.

---

## Key Endpoints

| Method | Endpoint | Description | Status |
|--------|----------|-------------|--------|
| `POST` | `/api/v1/expense-proposals` | Create an extraordinary expense proposal | `201 Created` |
| `GET` | `/api/v1/expense-proposals` | List expense proposals (optional query: `?status=PENDING_REVIEW\|APPROVED\|REJECTED`) | `200 OK` |
| `GET` | `/api/v1/expense-proposals/{id}` | Get detail of an expense proposal including vote breakdown | `200 OK` |
| `POST` | `/api/v1/expense-proposals/{id}/votes` | Cast a Board member vote (`APPROVED` / `REJECTED`) | `201 Created` |
| `GET` | `/api/v1/expense-proposals/{id}/votes` | List immutable votes for a proposal | `200 OK` |
| `GET` | `/health` | Service health check | `200 OK` |

Swagger UI is available locally at: `http://localhost:8088/swagger-ui/index.html`

---

## How to Run it Locally

```bash
# 1. Start the MySQL database
mysql -u root -p -e "CREATE DATABASE finance_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 2. Run Flyway migrations and start the service
./mvnw spring-boot:run

# 3. Verify health & proposals
curl http://localhost:8088/api/v1/expense-proposals
```

---

## Related Documents

- [data-model.md](./data-model.md) — DDL, tables, and Mermaid ER diagram (`finance_db`)
- [events.md](./events.md) — Published domain events (`ExpenseProposalCreated`, `ExpenseProposalApproved`, `ExpenseProposalRejected`)
- [decisions.md](./decisions.md) — Design decisions (append-only votes, quorum resolution)
- [runbook.md](./runbook.md) — Operational guidelines, monitoring, and troubleshooting
- [API Contract](../../../07-api/contracts/openapi/finance-service.yaml)

---

## Environment Variables

| Variable | Description | Default / Example |
|----------|-------------|-------------------|
| `SERVER_PORT` | Local service port | `8088` |
| `SPRING_DATASOURCE_URL` | MySQL JDBC URL | `jdbc:mysql://localhost:3306/finance_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true` |
| `SPRING_DATASOURCE_USERNAME` | Database user | `resi_dev` |
| `DB_PASSWORD` | Database password | `****` |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | Hibernate schema validation | `validate` (Mandatory) |
| `SPRING_FLYWAY_ENABLED` | Database migration trigger | `true` |
