# Data Model — `iam-service` (`iam_db`)

> **Engine:** MySQL 8.0+ (`InnoDB`, `utf8mb4_unicode_ci`) — Dedicated database per `ADR-004`.  
> **Migrations:** Flyway forward-only scripts (`V1__init_iam.sql`) with `spring.jpa.hibernate.ddl-auto=validate`.  
> **Authoritative Source:** Synchronized 1:1 with `06-data/models.md`.

---

## Entity-Relationship Diagram

```mermaid
erDiagram
    users ||--o{ refresh_tokens : "issues / rotates"
    users {
        CHAR_36 id PK
        VARCHAR_255 email UK
        VARCHAR_255 password_hash
        VARCHAR_20 role
        VARCHAR_20 status
        INT failed_attempts
        TIMESTAMP locked_until
        TIMESTAMP created_at
        TIMESTAMP updated_at
        TIMESTAMP deleted_at
    }
    refresh_tokens {
        CHAR_36 id PK
        CHAR_36 user_id FK
        VARCHAR_255 token_hash
        CHAR_36 family_id
        BOOLEAN revoked
        CHAR_36 replaced_by
        TIMESTAMP expires_at
        TIMESTAMP created_at
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

## Tables Owned by `iam_db`

### 1. `users`
- Stores one authentication account per user (`FR01`, `FR20`).
- **Constraints:**
  - `uq_users_email`: Unique email across active/historical accounts.
  - `chk_users_role`: `role IN ('ADMINISTRATOR','BOARD','PERSON','MAINTENANCE_STAFF','SECURITY_GUARD')`.
  - `chk_users_status`: `status IN ('PENDING_ACTIVATION','ACTIVE','INACTIVE','LOCKED')`.
- **Security:** `password_hash` stores BCrypt hashes (cost ≥ 12); `failed_attempts` and `locked_until` enforce `BR-02` (5 failures → 15 min lock).

### 2. `refresh_tokens`
- Persists 7-day rotatable refresh tokens (`BR-03`, `ADR-006`).
- **Constraints:**
  - `fk_refresh_tokens_user`: `FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE`.
  - `token_hash` stores BCrypt hash of the opaque token (never plaintext).
  - `family_id` + `revoked` + `replaced_by` enable immediate revocation of the entire session family if a rotated token is replayed.

### 3. `outbox`
- Transactional Outbox table (`ADR-005`) storing `UserRegistered` and `UserActivated` events committed atomically with user mutations.
