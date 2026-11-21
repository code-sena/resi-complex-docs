# Data Model — `people-service` (`people_db`)

> **Engine:** MySQL 8.0+ (`InnoDB`, `utf8mb4_unicode_ci`) — Dedicated database per `ADR-004`.  
> **Migrations:** Flyway forward-only scripts + `spring.jpa.hibernate.ddl-auto=validate`.  
> **Authoritative Source:** Synchronized 1:1 with `06-data/models.md`.

---

## Entity-Relationship Diagram

```mermaid
erDiagram
    person ||--o{ unit_occupancies : "occupies / owns"
    person {
        CHAR_36 id PK
        VARCHAR_150 full_name
        VARCHAR_30 document_id UK
        VARCHAR_20 person_type
        CHAR_36 unit_id "Logical ref -> units_db.units.id"
        VARCHAR_20 phone
        VARCHAR_255 email
        TIMESTAMP created_at
        TIMESTAMP updated_at
        TIMESTAMP deleted_at
    }
    unit_occupancies {
        CHAR_36 id PK
        CHAR_36 unit_id "Logical ref -> units_db.units.id"
        CHAR_36 person_id FK
        VARCHAR_20 occupancy_type
        VARCHAR_20 status
        DATE start_date
        DATE end_date
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

## Key Constraints & Invariants

1. `uq_person_document`: Unique government `document_id` across `person`.
2. `chk_person_type`: `person_type IN ('RESIDENT','COMMERCIAL_OWNER','COMMERCIAL_TENANT')` (`BR-06`).
3. `fk_occupancies_person`: `FOREIGN KEY (person_id) REFERENCES person(id) ON DELETE RESTRICT ON UPDATE RESTRICT`.
4. `chk_occupancies_type`: `occupancy_type IN ('OWNER','TENANT')`.
5. `chk_occupancies_status`: `status IN ('ACTIVE','INACTIVE')`.
6. `chk_occupancies_dates`: `end_date IS NULL OR end_date >= start_date`.
7. Composite indexes `idx_occupancies_unit_status (unit_id, status)` and `idx_occupancies_person_status (person_id, status)` ensure sub-millisecond `:own` ownership checks (`BR-07`).
