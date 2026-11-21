# Local Technical Decisions — `people-service`

## Decision 1 — Single `person` Entity with `person_type` Enum (`BR-06`)
- **Context:** Residents, commercial owners, and commercial tenants share the same personal attributes and authenticate under the single IAM role `PERSON`.
- **Decision:** Model all three subtypes inside `people_db.person` discriminated by `person_type` (`RESIDENT`, `COMMERCIAL_OWNER`, `COMMERCIAL_TENANT`).
- **Consequences:** Avoids table fragmentation and preserves 1:1 alignment with `01-context/glossary.md`.

## Decision 2 — Decouple Tenancy History via `unit_occupancies` (`BR-07`)
- **Context:** When a tenant's lease expires, deleting the person record would break historical maintenance, fee, and correspondence audit trails; leaving them linked would leak future occupants' private data (violating Ley 1581 de 2012).
- **Decision:** Track unit relationships in `unit_occupancies` (`OWNER`/`TENANT`, `ACTIVE`/`INACTIVE`, `start_date`/`end_date`). Deactivation sets `status = 'INACTIVE'` and `end_date`, revoking `:own` access instantly while keeping full history intact.
- **Consequences:** Clean GDPR/Ley 1581 compliance and accurate multi-year occupancy history per unit.
