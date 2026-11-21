# Domain Events — `people-service`

> Published reliably via `people_db.outbox` to RabbitMQ Topic Exchange **`resicomplex.events`** (`ADR-005`).

---

## Published Events

### 1. `PersonRegistered`
- **Routing Key:** `people.person.registered`
- **Trigger:** Successful person creation (`POST /api/v1/people`, `FR21`).
- **Consumers:** `communications-service`, `reports-service`.
- **Payload Example:**
```json
{
  "eventId": "1a2b3c4d-5e6f-47a8-9b0c-112233445566",
  "eventType": "PersonRegistered",
  "version": "1.0",
  "occurredAt": "2026-11-21T00:00:00Z",
  "aggregateId": "22222222-3333-4444-5555-666666666666",
  "payload": {
    "personId": "22222222-3333-4444-5555-666666666666",
    "personType": "RESIDENT",
    "unitId": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d"
  }
}
```
*(Per Ley 1581 de 2012 & `00-governance/security-policy.md`, sensitive PII such as government document numbers or phone numbers is NEVER broadcast on the event bus.)*

### 2. `OccupancyDeactivated`
- **Routing Key:** `people.occupancy.deactivated`
- **Trigger:** Lease termination (`PATCH /api/v1/occupancies/{id}/deactivate`, `HU-PPL-002`, `BR-07`).
- **Consumers:** `access-control-service` (cancels active visitor pre-authorizations for vacated unit), `communications-service`.

---

## Consumed Events

- **None required in MVP.** `unitId` references are validated synchronously against `units-service` (`:8083`) on creation.
