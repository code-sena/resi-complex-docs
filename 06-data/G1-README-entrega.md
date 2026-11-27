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
