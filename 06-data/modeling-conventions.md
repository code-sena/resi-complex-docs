# Estándar de nombrado y modelado de datos — `resi-complex`

> **Documento base del equipo** · Ficha `3239188` · Guía 1 (RAP `593101`)  
> **Motor oficial del equipo:** MySQL 8.0 (`InnoDB`, `utf8mb4_unicode_ci`) según `ADR-004` y `06-data/models.md`.  
> Cada integrante verifica estas reglas sobre **el 100 % de las tablas** de su servicio en `09-microservices/services/NN-<servicio>/data-model.md` (sección D2).

---

## 1. Tipo único de identificador del equipo

- **Identificador elegido:** `UUID` (UUID v4), almacenado en MySQL 8 como `CHAR(36)` (ASCII/UTF-8 de longitud fija).
- **Justificación:**
  1. Evita colisiones entre los 9 microservicios independientes (`ADR-004`: *Database per Service*) al propagar referencias lógicas por API REST y eventos RabbitMQ (`ADR-005`).
  2. Permite generar identificadores en la capa de aplicación antes de persistir sin depender de secuencias centrales ni exponer conteos secuenciales enumerables en las URLs de la API.
  3. Las tablas intermedias puras de relaciones N:M usan **llave primaria compuesta** por sus dos llaves foráneas (`PRIMARY KEY (<entidad_a>_id, <entidad_b>_id)`).

---

## 2. Reglas oficiales de nombrado y modelado

| # | Categoría | Regla del equipo | Ejemplo correcto | Ejemplo incorrecto |
|---|---|---|---|---|
| **R1** | Tablas | Nombres en **plural**, en minúsculas y `snake_case`, sin prefijos de tabla (`tbl_`) ni acentos. | `refresh_tokens`, `user_roles` | `RefreshToken`, `tbl_user`, `rol` |
| **R2** | Columnas | Nombres en minúsculas y `snake_case`, descriptivos y sin abreviaturas opacas; booleanos con prefijo `is_` o participio claro (`revoked`). | `password_hash`, `failed_attempts`, `expires_at` | `pwdHsh`, `numFail`, `fecExp` |
| **R3** | Llaves foráneas (FK) | Columna nombrada `<entidad_singular>_id` y restricción nombrada `fk_<tabla>_<referenciada>` con acción referencial `ON DELETE` explícita (`RESTRICT`, `CASCADE` o `SET NULL`). **Prohibidas las FK hacia tablas de otro microservicio.** | `user_id`, `CONSTRAINT fk_refresh_tokens_users FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE` | `idUser`, FK sin nombre o FK cruzando hacia `people_db` |
| **R4** | Identificadores (PK) | Todas las tablas de entidad propia tienen columna `id CHAR(36) NOT NULL` con `PRIMARY KEY (id)` (o `pk_<tabla>`). Las tablas asociativas N:M usan llave primaria compuesta por sus FK. | `id CHAR(36) PRIMARY KEY`, `PRIMARY KEY (user_id, role_id)` | `id INT AUTO_INCREMENT` mezclado con `UUID` |
| **R5** | Fechas y horas | Todas las marcas de tiempo se almacenan en **UTC** con tipo `TIMESTAMP` (o `DATETIME(3)` cuando se requiera precisión de milisegundos en eventos/outbox) y sufijo `_at` o `_until`. Nunca `VARCHAR`. | `created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP`, `locked_until TIMESTAMP NULL` | `fecha_creacion VARCHAR(30)` |
| **R6** | Auditoría y borrado lógico | Todas las tablas principales incluyen columnas de auditoría (`created_at`, `updated_at`, `created_by`, `updated_by`) y borrado lógico (`deleted_at TIMESTAMP NULL`, donde `NULL` indica registro activo). | `created_at`, `updated_at`, `created_by`, `updated_by`, `deleted_at` | Eliminación física sin `deleted_at` ni trazabilidad de autor |
| **R7** | Índices | Índices nombrados `idx_<tabla>_<columna(s)>`. Toda llave foránea interna y toda columna filtrada frecuentemente en consultas de negocio debe contar con su índice justificado. | `idx_refresh_tokens_user_id`, `idx_users_status` | `index1`, FK sin índice documentado |
| **R8** | Restricciones de integridad | Restricciones únicas con prefijo `uq_<tabla>_<columna(s)>` y validaciones de dominio con prefijo `chk_<tabla>_<regla>`. | `uq_users_email`, `chk_users_failed_attempts` | `UNIQUE` anónimo o reglas de negocio sin `CHECK` |
| **R9** | Referencias entre servicios | Toda referencia a una entidad dueña de otro microservicio se almacena como `<entidad_singular>_id CHAR(36)` **sin `FOREIGN KEY` SQL**, documentando el servicio dueño y el mecanismo de sincronización (API REST o evento RabbitMQ). | `unit_id CHAR(36)` *(dueño: `03-units-service`, validado vía REST/evento)* | `FOREIGN KEY (unit_id) REFERENCES units_db.units(id)` |

---

## 3. Convenciones técnicas complementarias

- **Motor y juego de caracteres:** `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci`.
- **Valores monetarios:** `DECIMAL(12,2)` (nunca `FLOAT` ni `DOUBLE`).
- **Credenciales:** nunca en texto plano ni cifrado reversible; siempre con **hash y sal** (`BCrypt` / `Argon2id`) en columna `password_hash` o `token_hash`.
- **Migraciones:** scripts versionados e inmutables (`V1__` a `V4__` para la entrega de la Guía 2 en `09-microservices/services/NN-<servicio>/db/migrations/`).
