# Microservices Catalog

> **What to fill in here:** The complete inventory of system services.
> For the detail of each service, go to `09-microservices/services/NN-service-name/`.
> This catalog is the executive view — one row per service.

---

## Services map

```
                          ┌─────────────────┐
  External clients        │   API Gateway   │
  (Web, Mobile, API)  ──▶ │   :8080         │
                          └────────┬────────┘
                                   │ routes by path
          ┌───────────────┬────────┴───────────┬──────────────────┐
          ▼               ▼                    ▼                  ▼
  ┌──────────────┐ ┌──────────────┐  ┌──────────────────┐ ┌──────────────┐
  │ auth-service │ │[service-A]   │  │ [service-B]      │ │[service-N]   │
  │    :3001     │ │    :3002     │  │    :3003         │ │    :300N     │
  └──────────────┘ └──────┬───────┘  └────────┬─────────┘ └──────────────┘
                           │                   │
                           └──────────┬────────┘
                                      ▼
                            ┌─────────────────┐
                            │  Message Broker  │
                            │  (Kafka/RabbitMQ)│
                            └─────────────────┘
```

---

## Service registry

| # | Service | Main responsibility | Port | DB | DB type | Status |
|---|---------|-------------------|------|-----|---------|--------|
| 01 | `api-gateway` | Routing, authentication, rate limiting | 8080 | Redis | Cache | 🟢 Active |
| 02 | `auth-service` | Registration, login, JWT tokens, RBAC | 8081 | MySQL | Relational | 🟢 Active |
| 06 | `billing-service` | Monthly administration fees with differentiated residential and commercial tariffs | 8084 | MySQL (`billing_db`) | Relational | 🟢 Active |
| 07 | `finance-approval-service` | Extraordinary expense proposals and immutable Board member voting | 8088 | MySQL (`finance_db`) | Relational | 🟢 Active |
| 10 | `reports-service` | Executive CQRS read projections for fee arrears and maintenance statistics | 8089 | MySQL (`reports_db`) | Relational | 🟢 Active |

**Statuses:** 🟢 Active in prod | 🟡 In development | 🔴 Planned | ⏸ Deprecated

---

## Detail per service

### 01 — `api-gateway`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/01-api-gateway/` |
| **Responsibility** | Single entry point. Routes, authenticates, and applies rate limiting |
| **Type** | Infrastructure service (not domain) |
| **Port** | 8080 (HTTPS 8443) |
| **Technology** | [Kong / NGINX + custom / Spring Cloud Gateway] |
| **DB** | Redis (token cache, rate limiting) |
| **Communicates with** | All internal services |
| **Externally exposed** | Yes — it is the only entry point |

**Key endpoints:**
- `/*` — Proxy to internal services based on path

---

### 02 — `auth-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/02-auth-service/` |
| **Responsibility** | Identity and access: registration, login, refresh token, RBAC |
| **Type** | Supporting service |
| **Port** | 3001 |
| **Technology** | [Node.js + Express / Spring Boot / etc.] |
| **DB** | PostgreSQL |
| **Bounded Context** | `02-domain/domain-map.md` → IAM Context |
| **Externally exposed** | Only through the API Gateway |

**Key endpoints:**
- `POST /auth/register` — User registration
- `POST /auth/login` — Authentication, returns JWT
- `POST /auth/refresh` — Renew token
- `DELETE /auth/logout` — Invalidate refresh token

**Events published:**
- `UserRegistered` → Topic: `iam.user.registered`
- `SessionStarted` → Topic: `iam.session.started`

---

### 03 — `[service-name]`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/03-[name]/` |
| **Responsibility** | [Describe in one sentence] |
| **Type** | [Core / Supporting / Generic] |
| **Port** | 300X |
| **Technology** | [Stack] |
| **DB** | [Engine and name] |
| **Bounded Context** | [Reference in domain-map] |
| **Externally exposed** | [Yes / Internal only] |

**Key endpoints:**
- `[METHOD] /[path]` — [description]

**Events published:**
- `[EventName]` → Topic: `[topic.name]`

**Events consumed:**
- `[EventName]` from `[source service]`

---

### 06 — `billing-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/06-billing-service/` |
| **Responsibility** | Generate monthly administration fees with differentiated residential and commercial tariffs, calculate arrears, and record payments |
| **Type** | Core Domain service |
| **Port** | 8084 |
| **Technology** | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate, Flyway |
| **DB** | MySQL 8.0 (`billing_db`) |
| **Bounded Context** | `02-domain/domain-map.md` → Billing Context |
| **Externally exposed** | Yes (via API Gateway :8080) |

**Key endpoints:**
- `POST /api/v1/fees` — Generate monthly administration fee
- `GET /api/v1/fees` — List all administration fees (filter by `?status=PENDING|PAID|OVERDUE`)
- `GET /api/v1/fees/{id}` — Get single fee detail and late fee calculation
- `GET /api/v1/fees/unit/{unitId}` — List full account statement for a unit
- `PATCH /api/v1/fees/{id}/pay` — Record payment confirmation for a fee

**Events published:**
- `FeeGenerated` → Topic: `billing.fee.generated`
- `FeePaid` → Topic: `billing.fee.paid`

**Events consumed:**
- `UnitRegistered` from `units-service`

---

### 07 — `finance-approval-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/07-finance-service/` |
| **Responsibility** | Manage extraordinary expense proposals and append-only Board member voting with automated quorum resolution |
| **Type** | Supporting service |
| **Port** | 8088 |
| **Technology** | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate, Flyway |
| **DB** | MySQL 8.0 (`finance_db`) |
| **Bounded Context** | `02-domain/domain-map.md` → Financial Approval Context |
| **Externally exposed** | Yes (via API Gateway :8080) |

**Key endpoints:**
- `POST /api/v1/expense-proposals` — Create extraordinary expense proposal
- `GET /api/v1/expense-proposals` — List expense proposals (filter by `?status=PENDING_REVIEW|APPROVED|REJECTED`)
- `GET /api/v1/expense-proposals/{id}` — Get proposal detail and vote breakdown
- `POST /api/v1/expense-proposals/{id}/votes` — Cast immutable Board member vote (`APPROVED` / `REJECTED`)
- `GET /api/v1/expense-proposals/{id}/votes` — List votes for a proposal

**Events published:**
- `ExpenseProposalCreated` → Topic: `finance.expense.created`
- `ExpenseProposalApproved` → Topic: `finance.expense.approved`
- `ExpenseProposalRejected` → Topic: `finance.expense.rejected`

---

### 10 — `reports-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/10-reports-service/` |
| **Responsibility** | Maintain CQRS read projections (`pending_fees_view`, `requests_by_status_view`) and serve consolidated portfolio and maintenance statistics |
| **Type** | Supporting / Analytics service |
| **Port** | 8089 |
| **Technology** | Java 25, Spring Boot 4.0.8, Spring Data JPA, Hibernate, Flyway |
| **DB** | MySQL 8.0 (`reports_db`) |
| **Bounded Context** | `02-domain/domain-map.md` → Reports Context |
| **Externally exposed** | Yes (via API Gateway :8080) |

**Key endpoints:**
- `GET /api/v1/reports/fee-stats` — Get portfolio arrears and critical overdue units (>2 months)
- `POST /api/v1/reports/fee-stats` — Upsert unit debt read projection
- `GET /api/v1/reports/maintenance-stats` — Get maintenance request counts grouped by status
- `POST /api/v1/reports/maintenance-stats` — Update maintenance status count projection

**Events consumed:**
- `FeeGenerated`, `FeePaid` from `billing-service`
- `MaintenanceRequestCreated`, `MaintenanceRequestStatusUpdated` from `maintenance-service`

---

## Service communication matrix

> Who calls whom and through which channel.

| Source service | Destination service | Channel | Type | Description |
|----------------|--------------------|---------|----- |-------------|
| api-gateway | auth-service | HTTP/REST | Synchronous | Validate JWT on each request |
| [svc-a] | [svc-b] | Event | Asynchronous | Notify of [event] |
| [svc-b] | [svc-c] | HTTP/REST | Synchronous | Query [data] |

---

## Data ownership matrix

Each business entity has a single owner service.
Others access via API or events, never directly to the DB.

| Entity / Data | Owner service (Source of Truth) | How other services access |
|---------------|--------------------------------|--------------------------|
| User / Identity | auth-service | REST API (`GET /users/:id`) |
| [Entity A] | [service-A] | `[EntityACreated]` event |
| [Entity B] | [service-B] | REST API (`GET /b/:id`) |

---

## How to add a new service

1. Assign the next available number in this catalog
2. Create the folder: `cp -r 09-microservices/_template/service 09-microservices/services/NN-new-service`
3. Fill in the README, data-model, events, and decisions of the new service
4. Create the OpenAPI contract in `07-api/contracts/openapi/NN-new-service.yaml`
5. Add the service to this catalog
6. Add the ADR for the decision to create the service in `05-architecture/decisions/`
7. Update the diagram in `05-architecture/overview.md`

---

## Correlations

- Architecture and patterns → `05-architecture/`
- API contracts per service → `07-api/contracts/openapi/`
- System events → `02-domain/domain-events.md`
- Template for creating services → `09-microservices/_template/service/`
- Full detail per service → `09-microservices/services/NN-[name]/`
