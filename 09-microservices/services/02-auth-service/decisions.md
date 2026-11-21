# Local Technical Decisions — `iam-service`

## Decision 1 — Exactly 5 Canonical Authentication Roles (Single `PERSON` Role)
- **Context:** Mixed residential/commercial complexes have residents, commercial owners, and commercial tenants. Creating separate IAM roles per business subtype duplicates permission rules and couples IAM to unit occupancy changes.
- **Decision:** Enforce strictly 5 roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`) in `iam_db.users` (`00-governance/security-policy.md`, `01-context/glossary.md`). Business subtypes (`RESIDENT`, `COMMERCIAL_OWNER`, `COMMERCIAL_TENANT`) belong in `people-service`.
- **Consequences:** Clean, stable RBAC matrix across all 9 services.

## Decision 2 — MySQL 8 (`iam_db`) + Flyway + `ddl-auto=validate`
- **Context:** Refresh token rotation and account lockout require ACID transactions and strict schema governance (`ADR-004`).
- **Decision:** Store `users`, `refresh_tokens`, and `outbox` in dedicated MySQL 8 schema `iam_db` managed exclusively via Flyway (`V1__init_iam.sql`) with Hibernate `ddl-auto=validate`.
- **Consequences:** Eliminates Redis as a hard MVP dependency (`NFR-003`) while preserving atomic refresh-token reuse detection (`BR-03`).

## Decision 3 — HS256 (≥ 256-bit Secret) in MVP with RS256/JWKS Upgrade Path
- **Context:** Local formative deployment runs a single Gateway + IAM pair (`ADR-006`).
- **Decision:** Sign 1-hour JWT access tokens with `HS256` via shared `JWT_SECRET` in Phase 1 MVP, while keeping `JwtTokenProviderPort` hexagonal so switching to asymmetric `RS256` + `GET /api/v1/auth/jwks` requires changing only one adapter.
- **Consequences:** Zero key-management friction in local dev and effortless upgrade to asymmetric signing in production.
