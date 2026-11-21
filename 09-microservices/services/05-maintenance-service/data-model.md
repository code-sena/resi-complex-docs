# Data Model — `maintenance-service` (`maintenance_db`)

> **Engine:** MySQL 8.0+ (`InnoDB`, `utf8mb4_unicode_ci`) — Dedicated database per `ADR-004`.  
> **Migrations:** Flyway forward-only scripts + `spring.jpa.hibernate.ddl-auto=validate`.  
> **Authoritative Source:** Synchronized 1:1 with `06-data/models.md`.

---

## Entity-Relationship Diagram

```mermaid
erDiagram
    maintenance_requests {
        CHAR_36 id PK
        CHAR_36 person_id "Logical ref -> people_db.person.id"
        CHAR_36 unit_id "Logical ref -> units_db.units.id (NULL if COMMON_AREA)"
        VARCHAR_20 location_type
        VARCHAR_100 common_area_name
        VARCHAR_50 type
        VARCHAR_500 description
        VARCHAR_10 priority
        CHAR_36 assigned_to "Logical ref -> iam_db.users.id (MAINTENANCE_STAFF)"
        VARCHAR_20 status
        INT resident_rating
        VARCHAR_255 rejection_notes
        TIMESTAMP created_at
        TIMESTAMP updated_at
        TIMESTAMP deleted_at
    }
    outbox {
        CHAR_36 id PK
        CHAR_36 event_id UK
        VARCHAR_100 event_type
        VARCHAR_100 aggregate_type
        CHAR_36 aggregate_id
        JSON payload
        VARCHAR_150 routing_key
        VARCHAR_20 status
        INT retry_count
        DATETIME_3 created_at
    }
```

---

## Key Constraints & Domain Invariants

1. `chk_maintreq_location_type`: `location_type IN ('PRIVATE_UNIT','COMMON_AREA')`.
2. `chk_maintreq_location_ref` (`BR-08` / `INV-001`): `(location_type = 'PRIVATE_UNIT' AND unit_id IS NOT NULL) OR (location_type = 'COMMON_AREA' AND common_area_name IS NOT NULL)`.
3. `chk_maintreq_priority`: `priority IN ('LOW','MEDIUM','HIGH','URGENT')`.
4. `chk_maintreq_status` (`BR-09`): `status IN ('PENDING','ASSIGNED','IN_PROGRESS','WORK_COMPLETED','RESOLVED')`.
5. `chk_maintreq_rating`: `resident_rating IS NULL OR resident_rating BETWEEN 1 AND 5`.
6. Indexes on `(person_id)`, `(unit_id)`, `(status)`, `(assigned_to)`, and `(deleted_at)` support fast role-scoped `:own` and `:assigned` listings.
