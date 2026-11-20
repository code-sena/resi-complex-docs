# REST API Design Guidelines — resi-complex

> **Purpose:** Defines the mandatory REST API design standards for all microservices in the `resi-complex` project.
> Every team member must adhere to these guidelines when authoring OpenAPI contracts (`07-api/contracts/openapi/`) and implementing endpoints.

---

## 1. URI & Resource Naming Conventions

### 1.1 Base URL & Versioning
- **Versioning Strategy:** Version in the URL path (`/api/v1/`).
- **Gateway Entry Point:** All public traffic enters through the API Gateway on port `8080`.
- **Format:** `https://api.resicomplex.com/api/v1/{resource}`

### 1.2 Resource Naming
- **Nouns, not verbs:** Use plural nouns for collections (`/users`, `/units`, `/requests`), never verbs (`/getUsers`, `/createUnit`).
- **Kebab-case for multi-word paths:** `/maintenance-requests`, `/access-logs`.
- **Sub-resources for hierarchical relationships:**
  - `GET /users/{userId}/roles` (roles belonging to a specific user)
  - `GET /units/{unitId}/residents` (residents associated with a specific unit)
- **Actions that are not pure CRUD:** Use sub-resource states or explicit controller nouns:
  - `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`
  - `POST /expenses/{expenseId}/approvals`

---

## 2. HTTP Methods & Idempotency

| Method | Idempotent | Safe | Typical Use Case | Expected Success Status |
| :--- | :--- | :--- | :--- | :--- |
| **GET** | Yes | Yes | Retrieve resource or list | `200 OK` |
| **POST** | No | No | Create resource, execute action | `201 Created` or `200 OK` |
| **PUT** | Yes | No | Replace resource or full sub-resource collection | `200 OK` or `204 No Content` |
| **PATCH** | No / Yes | No | Partial update of resource fields | `200 OK` |
| **DELETE** | Yes | No | Delete or soft-delete resource | `204 No Content` |

---

## 3. Standard HTTP Status Codes

### Success Codes (2xx)
- `200 OK`: Request succeeded with response body.
- `201 Created`: Resource successfully created (must return the created resource in body or `Location` header).
- `204 No Content`: Request succeeded with empty body (e.g., `DELETE`, `POST /logout`).

### Client Error Codes (4xx)
- `400 Bad Request`: Input validation failed, malformed JSON, or missing required fields.
- `401 Unauthorized`: Missing, expired, or invalid JWT authentication token.
- `403 Forbidden`: Authenticated user lacks required RBAC role or ownership permissions.
- `404 Not Found`: Target resource or endpoint does not exist.
- `409 Conflict`: Business state conflict (e.g., email already registered, concurrent modification).
- `422 Unprocessable Entity`: Syntactically valid JSON that violates domain invariants.
- `423 Locked`: Account temporarily locked due to repeated failed login attempts.

### Server Error Codes (5xx)
- `500 Internal Server Error`: Unexpected server exception (generic message returned, internal stack trace logged with correlation ID).
- `503 Service Unavailable`: Downstream dependency (DB, Broker) unreachable or circuit breaker open.

---

## 4. Request & Response Formats

### 4.1 Pagination
All collection endpoints must support query parameter pagination with strict limits:
- `page`: 1-based index (default `1`, minimum `1`).
- `limit`: Number of items per page (default `20`, maximum `100`).

**Response Shape:**
```json
{
  "data": [ ... ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 142,
    "totalPages": 8
  }
}
```

### 4.2 Standard Error Response (RFC 7807)
All error responses adhere to the standard schema defined in `_shared.yaml`:
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Invalid input data provided",
  "details": [
    {
      "field": "email",
      "message": "Must be a well-formed email address"
    }
  ],
  "traceId": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## 5. Security & Headers

1. **Authorization Header:**
   - Format: `Authorization: Bearer <jwt_access_token>`
   - Required on all endpoints except explicitly declared public endpoints (`/login`, `/register`, `/jwks`, `/health`).
2. **Correlation ID Header:**
   - `X-Correlation-ID`: Trace ID passed across services for end-to-end distributed tracing.
3. **Content-Type:**
   - Always `application/json; charset=utf-8`.

---

## 6. Contract-First & OpenAPI Quality Checklist

Before submitting an OpenAPI contract pull request:
- [ ] Valid OpenAPI 3.0.3 YAML structure.
- [ ] Every path operation includes `operationId`, `summary`, and `description`.
- [ ] Every protected operation declares `security: [ { bearerAuth: [] } ]`.
- [ ] Allowed roles are clearly stated in the operation `description`.
- [ ] Explicit response schemas for `401 Unauthorized` and `403 Forbidden`.
- [ ] No hardcoded passwords, secret keys, or real PII in examples.
- [ ] Linter check: `npx @redocly/cli lint <contract.yaml>` executes with **0 errors**.
