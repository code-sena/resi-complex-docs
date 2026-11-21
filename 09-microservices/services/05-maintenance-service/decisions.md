# Local Technical Decisions — `maintenance-service`

## Decision 1 — Explicit `WORK_COMPLETED` Verification Gate (`BR-09`)
- **Context:** Discovery (`03-product/discovery-brief.md`) revealed residents complained when staff marked repairs closed without confirmation.
- **Decision:** Model `MaintenanceRequest` as a strict state machine where `MAINTENANCE_STAFF` can transition only up to `WORK_COMPLETED`. Transitioning to terminal `RESOLVED` requires requester verification (`/verify`), or `/reject` with mandatory `rejection_notes` returning the ticket to `IN_PROGRESS`.
- **Consequences:** Full accountability and resident satisfaction rating (`1–5`) before ticket closure.

## Decision 2 — Enforce Location Invariant at Both Domain and MySQL Layers (`BR-08`)
- **Context:** Maintenance covers both private apartments/shops (`PRIVATE_UNIT`) and shared complex infrastructure (`COMMON_AREA`, e.g., elevators, lobby).
- **Decision:** Enforce `chk_maintreq_location_ref` inside the `MaintenanceRequest` domain constructor AND as a MySQL 8 `CHECK` constraint.
- **Consequences:** Impossible to persist orphaned unit tickets or unnamed common-area tickets.
