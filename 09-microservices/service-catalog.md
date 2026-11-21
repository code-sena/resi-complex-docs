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
| 03 | `units-service` | Physical units (residential/commercial) & 1:1 business profiles | 8083 | MySQL (`units_db`) | Relational | 🟢 Active |
| 08 | `communications-service` | Segmented announcements (all, residential, commercial) | 8082 | MySQL (`communications_db`) | Relational | 🟢 Active |
| 09 | `access-control-service` | Visitor, vehicle entry/exit, front desk correspondence | 8087 | MySQL (`access_control_db`) | Relational | 🟢 Active |

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

### 03 — `units-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/03-units-service/` |
| **Responsibility** | Manage physical condominium spaces (residential/commercial) and 1:1 commercial profiles |
| **Type** | Core Domain |
| **Port** | 8083 |
| **Technology** | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway |
| **DB** | MySQL 8.0 (`units_db`) |
| **Bounded Context** | `02-domain/domain-map.md` → Units Management Context |
| **Externally exposed** | Yes (via API Gateway :8080) |

**Key endpoints:**
- `GET /health` — Service health and database connectivity
- `GET /api/v1/units` — List units with pagination and filtering (`unitType`, `search`)
- `POST /api/v1/units` — Register physical unit (residential or commercial) (`FR02`)
- `GET /api/v1/units/{id}` — Get single unit detail with establishment if commercial
- `PUT /api/v1/units/{id}` — Update unit metadata (`FR03`)
- `DELETE /api/v1/units/{id}` — Soft delete unit, blocked if debt exists (`FR03`, `BR-05`)
- `GET /api/v1/units/{id}/commercial` — Get commercial establishment profile (`FR04`)
- `POST /api/v1/units/{id}/commercial` — Register 1:1 business profile for commercial unit (`FR04`, `BR-04`)
- `PUT /api/v1/units/{id}/commercial` — Update commercial establishment profile (`FR04`)

**Events published:**
- `UnitRegistered` → Topic: `units.unit.registered`
- `UnitUpdated` → Topic: `units.unit.updated`
- `CommercialEstablishmentRegistered` → Topic: `units.establishment.registered`

**Events consumed:**
- *(None directly — synchronous HTTP check against `billing-service` on unit deletion)*

---

### 08 — `communications-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/08-communications-service/` |
| **Responsibility** | Manage official announcements and community communications with audience segmentation |
| **Type** | Supporting service |
| **Port** | 8082 |
| **Technology** | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway |
| **DB** | MySQL 8.0 (`communications_db`) |
| **Bounded Context** | `02-domain/domain-map.md` → Communications Context |
| **Externally exposed** | Yes (via API Gateway :8080) |

**Key endpoints:**
- `POST /api/v1/announcements` — Publish a new segmented announcement
- `GET /api/v1/announcements` — Query announcements filtered by unit type
- `GET /api/v1/announcements/{id}` — Get single announcement detail
- `PUT /api/v1/announcements/{id}` — Update announcement content/scope
- `DELETE /api/v1/announcements/{id}` — Soft delete announcement

**Events published:**
- `AnnouncementPublished` → Topic: `communications.announcement.published`

---

### 09 — `access-control-service`

| Field | Value |
|-------|-------|
| **Folder** | `09-microservices/services/09-access-control-service/` |
| **Responsibility** | Manage visitor and vehicle ingress/egress and front desk correspondence / parcel receipt |
| **Type** | Supporting service |
| **Port** | 8087 |
| **Technology** | Java 17+, Spring Boot 3.x, Spring Data JPA, Hibernate, Flyway |
| **DB** | MySQL 8.0 (`access_control_db`) |
| **Bounded Context** | `02-domain/domain-map.md` → Access Control Context |
| **Externally exposed** | Yes (via API Gateway :8080) |

**Key endpoints:**
- `POST /api/v1/visits` — Register incoming visitor admission
- `GET /api/v1/visits/active` — List visitors currently inside the complex
- `PATCH /api/v1/visits/{id}/exit` — Record visitor departure timestamp
- `POST /api/v1/correspondence` — Register incoming package/mail
- `GET /api/v1/correspondence` — List pending packages by unit
- `PATCH /api/v1/correspondence/{id}/deliver` — Mark parcel as collected

**Events published:**
- `VisitRegistered` → Topic: `access.visit.registered`
- `VisitEnded` → Topic: `access.visit.ended`
- `CorrespondenceReceived` → Topic: `access.correspondence.received`
- `CorrespondenceDelivered` → Topic: `access.correspondence.delivered`

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
