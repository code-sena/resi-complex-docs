# Technical Decisions — Communications Service

> Architecture Decision Records (ADRs) and design choices specific to `resi-complex-communications-service`.

---

### DEC-COMM-001: Query-Level Audience Segmentation
* **Status:** Accepted
* **Context:** The system must differentiate announcements for residential vs. commercial occupants. Retrieving all records and filtering in application memory increases bandwidth and latency.
* **Decision:** Enforce segmentation at the SQL database layer using custom JPA queries:
  ```java
  @Query("SELECT a FROM Announcement a WHERE a.deletedAt IS NULL AND (a.targetScope = 'ALL' OR a.targetScope = :scope)")
  List<Announcement> findByTargetScope(@Param("scope") String scope);
  ```
* **Consequences:** Low memory footprint, optimized query cache, and indexed lookups on `target_scope`.

---

### DEC-COMM-002: Soft Deletes with `deleted_at`
* **Status:** Accepted
* **Context:** Administrative notices often have legal and compliance implications in Colombian condominium management (Ley 675 de 2001). Physical deletion destroys historical records.
* **Decision:** Implement logical deletion where `DELETE /api/v1/announcements/{id}` sets `deleted_at = LocalDateTime.now()` instead of issuing a SQL `DELETE`.
* **Consequences:** All active queries must filter with `WHERE deleted_at IS NULL`. Historical traceability is completely preserved.

---

### DEC-COMM-003: Hibernate DDL Validation Requirement
* **Status:** Accepted
* **Context:** SENA ADSO guidelines require strict control over production database evolution and separation between Java entities and SQL DDL.
* **Decision:** Set `spring.jpa.hibernate.ddl-auto=validate`. All structural changes are committed as versioned Flyway scripts (`V1__init_communications.sql`, `V2__...`).
* **Consequences:** Hibernate verifies schema compatibility at startup and fails fast if Java entities deviate from the database schema.
