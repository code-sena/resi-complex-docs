# 07 — API Contracts

> **What is this?** The formal **Contract-First** OpenAPI 3.0.3 specification for `api-gateway` and all
> **9 microservices** of `resi-complex`. Every contract is registered in [`redocly.yaml`](../redocly.yaml)
> and validated with `npx @redocly/cli lint`.

---

## Why Contracts Are Critical in Microservices

In a microservices architecture, an undocumented API change can silently break the API Gateway, the frontend, or peer services. **Contract-First development** (`05-architecture/overview.md` Principle `P1`) ensures frontend, gateway, and backend teams build in parallel against a single linted specification.

---

## Complete API Contract Registry (10 / 10 Contracts Registered in `redocly.yaml`)

| # | Canonical Service | OpenAPI File | Port | FR Coverage | Allowed Canonical Roles | Validation |
|---|---|---|---|---|---|---|
| 01 | `api-gateway` | [`api-gateway.yaml`](./contracts/openapi/api-gateway.yaml) | `8080` *(local `:8090`)* | Cross-cutting (`ADR-006`) | All 5 roles (JWT edge verifier) | ✅ Linted (Redocly) |
| 02 | `iam-service` *(alias `auth-service`)* | [`auth-service.yaml`](./contracts/openapi/auth-service.yaml) | `8081` | `FR01`, `FR20` | Public auth + `ADMINISTRATOR` + all roles (own profile) | ✅ Linted (Redocly) |
| 03 | `communications-service` | [`communications-service.yaml`](./contracts/openapi/communications-service.yaml) | `8082` | `FR07`, `FR11` | `ADMINISTRATOR` (publish), All 5 roles (read scoped) | ✅ Linted (Redocly) |
| 04 | `units-service` | [`units-service.yaml`](./contracts/openapi/units-service.yaml) | `8083` | `FR02`, `FR03`, `FR04` | `ADMINISTRATOR`, `BOARD`, `PERSON` (`:own`), `MAINTENANCE_STAFF`, `SECURITY_GUARD` | ✅ Linted (Redocly) |
| 05 | `billing-service` | [`billing-service.yaml`](./contracts/openapi/billing-service.yaml) | `8084` | `FR08`, `FR09`, `FR10` | `ADMINISTRATOR`, `BOARD` (read), `PERSON` (`:own`) | ✅ Linted (Redocly) |
| 06 | `people-service` | [`people-service.yaml`](./contracts/openapi/people-service.yaml) | `8085` | `FR21` | `ADMINISTRATOR`, `BOARD` (read), `PERSON` (`:own`), `SECURITY_GUARD` (lookup) | ✅ Linted (Redocly) |
| 07 | `maintenance-service` | [`maintenance-service.yaml`](./contracts/openapi/maintenance-service.yaml) | `8086` | `FR05`, `FR06` | `ADMINISTRATOR`, `BOARD` (read), `PERSON` (`:own`), `MAINTENANCE_STAFF` (`:assigned`) | ✅ Linted (Redocly) |
| 08 | `access-control-service` | [`access-control-service.yaml`](./contracts/openapi/access-control-service.yaml) | `8087` | `FR12`–`FR16`, `FR22` | `ADMINISTRATOR`, `BOARD` (read), `SECURITY_GUARD`, `PERSON` (`:own`) | ✅ Linted (Redocly) |
| 09 | `finance-approval-service` *(alias `finance-service`)* | [`finance-service.yaml`](./contracts/openapi/finance-service.yaml) | `8088` | `FR17`, `FR18` | `ADMINISTRATOR` (create/read), `BOARD` (vote/read) | ✅ Linted (Redocly) |
| 10 | `reports-service` | [`reports-service.yaml`](./contracts/openapi/reports-service.yaml) | `8089` | `FR19` | `ADMINISTRATOR`, `BOARD` | ✅ Linted (Redocly) |

### Shared Components & Standards
- [`guidelines.md`](./guidelines.md) ⭐ — Mandatory REST naming, `/api/v1/` versioning, pagination (`page`/`limit`), HTTP status codes, RFC 7807 `ErrorResponse`, and PR OpenAPI checklist.
- [`authentication.md`](./authentication.md) ⭐ — JWT (1h access / 7d rotating refresh), Gateway `X-User-*` propagation headers, **5 canonical RBAC roles** (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`), and `:own` / `:assigned` ownership rules.
- [`contracts/openapi/_shared.yaml`](./contracts/openapi/_shared.yaml) — Shared OpenAPI schemas (`PaginatedMeta`, `ErrorResponse`, `UUID`, `Timestamp`, `AuditFields`), reusable parameters, standard error responses, and `bearerAuth` security scheme.
- [`contracts/openapi/_template-service.yaml`](./contracts/openapi/_template-service.yaml) — Reference template for any future v2.0 microservice contract.

---

## Validation Commands

Run from repository root (`c:\IA\resi-complex-docs`):

```bash
# Lint ALL 10 registered contracts at once via redocly.yaml
npx @redocly/cli lint

# Lint a single contract
npx @redocly/cli lint 07-api/contracts/openapi/people-service.yaml
```

---

## Correlations with Other Sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/functional.md` (`FR01`–`FR22`, `BR-01`–`BR-16`) | Endpoints, validation rules, and status transitions |
| `06-data/models.md` (18 domain tables across 9 MySQL 8 DBs) | Request/response schemas, enums, and field constraints |
| `00-governance/security-policy.md` & `ADR-006` | `authentication.md` & RBAC role annotations per endpoint |
| `07-api/contracts/openapi/*.yaml` | `09-microservices/services/01-*`..`10-*` & `11-quality/testing-strategy.md` |
| `07-api/contracts/openapi/*.yaml` | `12-ux-ui/navigation-map.md` (frontend screen data calls) |
