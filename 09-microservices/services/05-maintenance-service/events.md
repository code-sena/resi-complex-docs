# Domain Events — `maintenance-service`

> Published reliably via `maintenance_db.outbox` to RabbitMQ Topic Exchange **`resicomplex.events`** (`ADR-005`).

---

## Published Events

### 1. `MaintenanceRequestCreated`
- **Routing Key:** `maintenance.request.created`
- **Trigger:** `POST /api/v1/maintenance-requests` (`FR05`).
- **Consumers:**
  - `communications-service`: Alerts `ADMINISTRATOR`; if `priority == URGENT`, triggers immediate alert to `MAINTENANCE_STAFF` (`02-domain/domain-events.md` reactive policy).
  - `reports-service`: Increments `PENDING` count in `requests_by_status_view` (`FR19`).
- **Payload Example:**
```json
{
  "eventId": "9a8b7c6d-1234-4567-89ab-cdef01234567",
  "eventType": "MaintenanceRequestCreated",
  "version": "1.0",
  "occurredAt": "2026-11-21T00:00:00Z",
  "aggregateId": "33333333-4444-5555-6666-777777777777",
  "payload": {
    "requestId": "33333333-4444-5555-6666-777777777777",
    "personId": "22222222-3333-4444-5555-666666666666",
    "locationType": "PRIVATE_UNIT",
    "unitId": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
    "priority": "URGENT",
    "status": "PENDING"
  }
}
```

### 2. `MaintenanceRequestStatusUpdated`
- **Routing Key:** `maintenance.request.status-updated`
- **Trigger:** `/assign`, `/status`, `/verify`, or `/reject` transitions (`FR06`).
- **Consumers:**
  - `communications-service`: Delivers in-app status notification to requester or assigned technician (`FR07`, `HU-MAINT-003`).
  - `reports-service`: Updates `requests_by_status_view` counters (`FR19`).

---

## Consumed Events

- **None required in MVP.**
