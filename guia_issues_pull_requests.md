# Guía de Trabajo: Issues, Ramas y Pull Requests — resi-complex

> Guía rápida para el equipo de desarrollo de `resi-complex` sobre cómo gestionar ramas, commits y Pull Requests hacia la rama `dev`.

---

## 1. Flujo General de Trabajo

```
[dev] ───────(crear rama)───────▶ [docs/api-auth-service]
                                            │
                                      (hacer commits)
                                            │
                                            ▼
                                     [Pull Request] ──▶ (Revisión & Merge) ──▶ [dev]
                                     (Closes #6)
```

1. **Partir siempre de `dev` actualizada:**
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b docs/api-auth-service
   ```
2. **Realizar los cambios y validar localmente:**
   - Validar contratos OpenAPI con Redocly:
     ```bash
     npx @redocly/cli lint 07-api/contracts/openapi/auth-service.yaml
     ```
   - Debe dar **0 errores**.
3. **Confirmar autor y correo en Git:**
   ```bash
   git config user.name "Tu Nombre"
   git config user.email "tu-correo-github@example.com"
   ```
4. **Hacer commit siguiendo Conventional Commits:**
   ```bash
   git add .
   git commit -m "docs(07-api): update role enums and add administration endpoints (FR20)"
   ```
5. **Publicar la rama en GitHub:**
   ```bash
   git push -u origin docs/api-auth-service
   ```
6. **Abrir el Pull Request (PR) hacia `dev`:**
   - Base branch: `dev`
   - Compare branch: `docs/api-auth-service`
   - Título: `docs(07-api): update role enums and add user admin endpoints`
   - Descripción: Incluir `Closes #6` para que GitHub cierre automáticamente el issue correspondiente al mergear.

---

## 2. Convenciones de Ramas

| Tipo de Tarea | Prefijo de Rama | Ejemplo |
| :--- | :--- | :--- |
| Documentación / OpenAPI | `docs/` | `docs/api-auth-service`, `docs/api-units-service` |
| Nueva funcionalidad | `feat/` | `feat/auth-login`, `feat/unit-registration` |
| Corrección de error | `fix/` | `fix/token-expiration-claim` |
| Configuración / DevOps | `chore/` | `chore/initial-docs`, `chore/docker-compose` |

---

## 3. Formato del Mensaje de Pull Request

Utilizar la plantilla oficial (`.github/pull_request_template.md`):

```markdown
## What does this PR do?
Updates auth-service OpenAPI contract with the 6 domain roles and adds administrative endpoints (FR20) for user management and role assignment.

## Type of change
- [x] Documentation (docs)

## Related HU(s)
Closes #6

## Changes included
- `07-api/contracts/openapi/auth-service.yaml` — updated Role enum to 6 domain roles, added endpoints for users CRUD, role assignment, and role catalog.
- `07-api/guidelines.md` — REST API guidelines for team endpoints.
- `07-api/authentication.md` — authentication flows and role × service permissions matrix.

## Tests performed
- [x] npx @redocly/cli lint executed with 0 errors and 0 warnings.
```

---

## 4. Criterio de Terminado (DoD) para el Rol de Seguridad (R2)

- [x] Contrato `auth-service.yaml` con roles corregidos y endpoints de administración (FR20).
- [x] Respuestas 401 y 403 especificadas en cada operación protegida.
- [x] `npx @redocly/cli lint` pasa con 0 errores.
- [x] PR creado apuntando a la rama `dev` con `Closes #6`.
- [x] 0 alertas de seguridad abiertas en GitHub Security.
