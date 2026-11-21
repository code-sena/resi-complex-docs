# Domain Events — People Service

> Specifies asynchronous domain events published and consumed by `resi-complex-people-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

### `PersonRegistered`
Emitted immediately when a new person profile (Resident or Commercial Owner/Tenant) is registered in the system (`HU-PPL-001`, `FR21`).

* **Routing Key:** `people.person.registered`
* **Producer:** `resi-complex-people-service`
* **Consumers:** `billing-service`, `maintenance-service`, `access-control-service`, `communications-service`, `reports-service`

#### Payload Schema (JSON):
```json
{
  "eventId": "110e8400-e29b-41d4-a716-446655440000",
  "eventType": "PersonRegistered",
  "aggregateId": "220e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "Person",
  "occurredAt": "2026-11-20T23:00:00Z",
  "version": 1,
  "payload": {
    "personId": "220e8400-e29b-41d4-a716-446655440001",
    "fullName": "Carlos Andrés Gómez",
    "documentId": "1020304050",
    "personType": "RESIDENT",
    "unitId": "550e8400-e29b-41d4-a716-446655440001",
    "email": "carlos.gomez@example.com",
    "phone": "+57 300 123 4567"
  },
  "metadata": {
    "correlationId": "110e8400-e29b-41d4-a716-446655440002",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

---

### `OccupancyAssigned`
Emitted when a person is formally linked to a physical unit with an active tenancy or ownership lease (`HU-PPL-001`).

* **Routing Key:** `people.occupancy.assigned`
* **Producer:** `resi-complex-people-service`
* **Consumers:** `access-control-service`, `billing-service`, `reports-service`

#### Payload Schema (JSON):
```json
{
  "eventId": "330e8400-e29b-41d4-a716-446655440010",
  "eventType": "OccupancyAssigned",
  "aggregateId": "440e8400-e29b-41d4-a716-446655440011",
  "aggregateType": "UnitOccupancy",
  "occurredAt": "2026-11-20T23:05:00Z",
  "version": 1,
  "payload": {
    "occupancyId": "440e8400-e29b-41d4-a716-446655440011",
    "unitId": "550e8400-e29b-41d4-a716-446655440001",
    "personId": "220e8400-e29b-41d4-a716-446655440001",
    "occupancyType": "TENANT",
    "status": "ACTIVE",
    "startDate": "2026-01-01",
    "endDate": "2026-12-31"
  },
  "metadata": {
    "correlationId": "330e8400-e29b-41d4-a716-446655440012",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

---

### `OccupancyDeactivated`
Emitted immediately when an occupant's tenancy terminates or is deactivated (`HU-PPL-002`), triggering immediate revocation of unit-level authorizations.

* **Routing Key:** `people.occupancy.deactivated`
* **Producer:** `resi-complex-people-service`
* **Consumers:** `access-control-service`, `iam-service`, `billing-service`, `reports-service`

#### Payload Schema (JSON):
```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440020",
  "eventType": "OccupancyDeactivated",
  "aggregateId": "440e8400-e29b-41d4-a716-446655440011",
  "aggregateType": "UnitOccupancy",
  "occurredAt": "2026-11-20T23:10:00Z",
  "version": 1,
  "payload": {
    "occupancyId": "440e8400-e29b-41d4-a716-446655440011",
    "unitId": "550e8400-e29b-41d4-a716-446655440001",
    "personId": "220e8400-e29b-41d4-a716-446655440001",
    "status": "INACTIVE",
    "endDate": "2026-11-20"
  },
  "metadata": {
    "correlationId": "550e8400-e29b-41d4-a716-446655440022",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

---

## 2. Event Consumption Matrix (Downstream Effects)

| Event | Consuming Service | Action Performed | Idempotency Strategy |
|---|---|---|---|
| `PersonRegistered` | `iam-service` | Generates secure in-app user activation invitation token | Upsert by `personId` |
| `PersonRegistered` | `communications-service` | Enrolls contact channels into complex-wide notification distribution | Upsert by `personId` |
| `OccupancyAssigned` | `access-control-service` | Authorizes visitor pre-registration and parcel pickup for the unit | Upsert by `occupancyId` |
| `OccupancyAssigned` | `billing-service` | Links recipient for administration fee statements | Upsert by `occupancyId` |
| `OccupancyDeactivated` | `access-control-service` | Revokes visitor authorization and parcel pickup permissions immediately | Dedup via `eventId` |
| `OccupancyDeactivated` | `iam-service` | Revokes `:own` ownership scope on unit endpoints | Dedup via `eventId` |

---

## 3. Reliability & Transactional Guarantees

1. **Transactional Outbox:** Every operation that alters `person` or `unit_occupancies` writes an event record to `outbox_events` inside the same database transaction.
2. **Outbox Poller:** A Spring `@Scheduled` background worker reads pending outbox entries, publishes them to RabbitMQ, and records the processed timestamp upon receiving broker confirmation (`ACK`).
