# README de entrega · Guía 1: Modelo de datos del software

> Este archivo pertenece a la **base común del equipo** en `06-data/G1-README-entrega.md`. La tabla de la sección 2 tiene **una fila por aprendiz**, y la sección 4 tiene **una subsección por aprendiz**. Cada aprendiz edita **solo su fila y su subsección**, desde su propia rama `g1/<NN-servicio>-<usuario>`. Borra las instrucciones entre `>` en tu sección cuando la diligencies.
>
> No escribas nombres completos, números de documento, teléfonos ni correos. Identifícate **solo con tu usuario de GitHub**.

## 1. Datos del equipo

| Campo | Valor |
|---|---|
| Ficha | 3239188 |
| Repositorio | `code-sena/resi-complex-docs` |
| Proyecto | `resi-complex` (Plataforma de gestión para conjuntos residenciales mixtos) |
| Rama de integración (destino de las solicitudes de integración) | `dev` |
| Catálogo de servicios acordado | `09-microservices/service-catalog.md` (fecha del commit del acuerdo: `2026-11-21`, commit `7d8fabb`) |
| Estándar de nombrado y modelado del equipo | `06-data/modeling-conventions.md` |
| Tipo de identificador del equipo | `UUID` (`CHAR(36)` en MySQL 8; justificado en `06-data/modeling-conventions.md`) |

## 2. Tabla de propiedad (una fila por aprendiz, registrada el 26 de noviembre)

> Cada servicio o conjunto de tablas tiene **un solo dueño**. `01-api-gateway` no sirve como único servicio. Si tu servicio tiene menos de 4 entidades propias, agrega un segundo servicio o un conjunto acotado de tablas. Agrega **solo tu fila** desde tu rama `g1/<NN-servicio>-<usuario>`.

| Usuario de GitHub | Servicio(s) a cargo | Tablas a cargo | Motor | Rama | Solicitud de integración |
|---|---|---|---|---|---|
| `Trujillo7915` | `02-auth-service` | `users`, `roles`, `permissions`, `user_roles`, `role_permissions`, `refresh_tokens`, `outbox` | MySQL 8 | `g1/02-auth-service-Trujillo7915` | #26 |

## 3. Archivos compartidos

| Archivo | Qué escribe cada aprendiz |
|---|---|
| `09-microservices/service-catalog.md` | El equipo acuerda el catálogo; cada aprendiz completa la ficha de su servicio |
| `06-data/models.md` | `## Servicio: NN-<servicio>` con el motor, el usuario responsable y el enlace a su `data-model.md` |
| `06-data/data-dictionary.md` | `## Servicio: NN-<servicio> (@usuario)` con el diccionario de sus tablas |
| `06-data/modeling-conventions.md` | El equipo declara ≥ 6 reglas; cada aprendiz las verifica en su `data-model.md` |
| `09-microservices/data-ownership-matrix.md` | Las filas de las entidades de su servicio, con su usuario en la última columna |

## 4. Sección individual de cada aprendiz

> Copia el bloque de `Plantilla_README_Entrega_G1.md` una vez por aprendiz al final de esta sección desde tu propia rama `g1/<NN-servicio>-<usuario>` y diligéncialo completamente.

### Aprendiz `@Trujillo7915`

**Servicio(s):** `02-auth-service` · **Rama:** `g1/02-auth-service-Trujillo7915` · **Solicitud de integración:** #26

#### 4.1 Archivos de mi entrega

| Artefacto | Archivo | Origen («nuevo» o commit mío anterior, con su enlace) | Qué ajusté o agregué |
|---|---|---|---|
| M1 Modelo conceptual | `09-microservices/services/02-auth-service/data-model.md` | En progreso (base propia previa en `7d8fabb`) | Pendiente ampliar a 7 tablas RBAC + traza a requisitos |
| M2 Modelo lógico y cuadro de reglas | `09-microservices/services/02-auth-service/data-model.md` | En progreso (base propia previa en `7d8fabb`) | Pendiente cuadro de reglas de transformación, auditoría y borrado lógico |
| M3 Normalización | `09-microservices/services/02-auth-service/normalization.md` | Nuevo (pendiente) | Pendiente 0FN → 3FN (llave compuesta y dependencia transitiva) |
| Sección del índice | `06-data/models.md` | En progreso (base propia previa en `7d8fabb`) | Pendiente alinear encabezado `## Servicio: 02-auth-service` con las 7 tablas |
| D1 Diccionario | `06-data/data-dictionary.md` | Nuevo (pendiente) | Pendiente diccionario completo del 100 % de columnas de `02-auth-service` |
| D2 Verificación del estándar | `09-microservices/services/02-auth-service/data-model.md` | Nuevo (pendiente) | Pendiente tabla de verificación de las reglas de `06-data/modeling-conventions.md` |
| D3 Motor | `09-microservices/services/02-auth-service/data-model.md` | En progreso (base propia previa en `7d8fabb`) | Pendiente justificación con ≥ 3 condiciones y 1 alternativa descartada |
| D4 Políticas | `09-microservices/services/02-auth-service/data-security.md` | Nuevo (pendiente) | Pendiente ≥ 6 políticas (integridad, confidencialidad, disponibilidad) |
| D5 Roles y datos personales | `09-microservices/services/02-auth-service/data-security.md` | Nuevo (pendiente) | Pendiente matriz de ≥ 3 roles y tabla de tratamiento de datos personales |
| D6 Propiedad | `09-microservices/data-ownership-matrix.md` | Nuevo (pendiente) | Pendiente filas de las entidades de `02-auth-service` |

**Qué reutilicé (2 a 4 líneas):**
Se parte del modelo base propio de `02-auth-service` (`iam_db`) previamente publicado por `@Trujillo7915` en `dev` (commit `7d8fabb`: tablas `users`, `refresh_tokens` y `outbox`).

**Qué ajusté para llegar al estándar de la rúbrica (2 a 4 líneas):**
Se ampliará el modelo relacional normalizando el esquema RBAC (`roles`, `permissions`, `user_roles`, `role_permissions`) para cumplir los mínimos de entidades, tablas, relaciones N:M, auditoría y borrado lógico en el 100 % de las tablas.

**Qué hice nuevo (2 a 4 líneas):**
Registro inicial de propiedad de `02-auth-service` el 26 de noviembre de 2026; los artefactos `normalization.md`, `data-security.md`, diccionario D1 y matriz D6 se completarán en entregas sucesivas antes del 6 de diciembre.

#### 4.2 Autoverificación de mínimos

Producto 1:
- [ ] M1 ≥ 4 entidades propias, ≥ 3 relaciones con cardinalidad mínima y máxima, ≥ 1 1:N, ≥ 1 N:M (o multivaluado declarado), externas marcadas, traza a requisitos
- [ ] M2 ≥ 5 tablas con PK; FK solo internas; referencias externas sin FK; auditoría y borrado lógico; ≥ 2 índices; cuadro de ≥ 4 reglas; `erDiagram` con tipos
- [ ] M3 ≥ 2 tablas de 0FN a 3FN con dependencias funcionales (una con llave compuesta y una con transitiva)
- [ ] Sección de mi servicio en `06-data/models.md`

Producto 2:
- [ ] D1 diccionario del 100 % de mis columnas, idéntico a mi modelo lógico
- [ ] D2 ≥ 6 reglas del estándar verificadas sobre todas mis tablas
- [ ] D3 motor con ≥ 3 condiciones y 1 alternativa descartada
- [ ] D4 ≥ 6 políticas (2 por principio), con hash y sal, TLS, respaldo y prueba de restauración
- [ ] D5 matriz de ≥ 3 roles contra todas mis tablas y tratamiento de datos personales
- [ ] D6 filas de mis entidades en la matriz de propiedad

#### 4.3 Días de trabajo

| Día | Qué avancé (mensaje del commit) |
|---|---|
| 2026-11-26 | `docs(g1): registra propiedad del servicio 02-auth-service` |
| *(pendiente Día 2+)* | |

#### 4.4 Uso de inteligencia artificial (obligatorio)

| Herramienta | Para qué la usé | Qué verifiqué o cambié después |
|---|---|---|
| Antigravity (asistente IA) | Revisar lista de verificación contra `Rubricas_G1.md`, estructurar la base común del equipo y redactar el registro inicial de propiedad | Verifiqué la coherencia con `ADR-004`, el motor MySQL 8, las 7 tablas propias de `02-auth-service` y las reglas de autoría |

#### 4.5 Declaración de autoría

Declaro que el contenido de mis archivos y de mis secciones es de mi autoría, que lo que reutilicé está indicado en 4.1, que no entregué como propio material del instructor, plantillas sin diligenciar ni contenido de otros aprendices, y que puedo explicar cada artefacto en la verificación oral.

Usuario de GitHub: `Trujillo7915` · Fecha: `2026-11-26`
