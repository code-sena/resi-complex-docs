# Domain Events — Units Service

> Specifies asynchronous domain events published and consumed by `resi-complex-units-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

### `UnitRegistered`
Emitted immediately when a new residential or commercial physical unit is registered by an Administrator (`FR02`).

* **Routing Key:** `units.unit.registered`
* **Producer:** `resi-complex-units-service`
* **Consumers:** `billing-service`, `access-control-service`, `communications-service`, `reports-service`

#### Payload Schema (JSON):
```json
{
  "eventId": "550e8400-e29b-41d4-a716-446655440000",
  "eventType": "UnitRegistered",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "Unit",
  "occurredAt": "2026-11-20T22:30:00Z",
  "version": 1,
  "payload": {
    "unitId": "550e8400-e29b-41d4-a716-446655440001",
    "unitNumber": "Apto 301",
    "unitType": "RESIDENTIAL"
  },
  "metadata": {
    "correlationId": "550e8400-e29b-41d4-a716-446655440002",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

---

### `UnitUpdated`
Emitted when metadata or classification of an existing unit is modified or when a unit is soft-deleted (`FR03`).

* **Routing Key:** `units.unit.updated`
* **Producer:** `resi-complex-units-service`
* **Consumers:** `billing-service`, `access-control-service`, `communications-service`, `reports-service`

#### Payload Schema (JSON):
```json
{
  "eventId": "660e8400-e29b-41d4-a716-446655440010",
  "eventType": "UnitUpdated",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "Unit",
  "occurredAt": "2026-11-20T22:35:00Z",
  "version": 1,
  "payload": {
    "unitId": "550e8400-e29b-41d4-a716-446655440001",
    "unitNumber": "Apto 301-A",
    "unitType": "RESIDENTIAL",
    "isDeleted": false
  },
  "metadata": {
    "correlationId": "660e8400-e29b-41d4-a716-446655440012",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

---

### `CommercialEstablishmentRegistered`
Emitted when the commercial business profile is registered or updated for a `COMMERCIAL` unit (`FR04`).

* **Routing Key:** `units.establishment.registered`
* **Producer:** `resi-complex-units-service`
* **Consumers:** `communications-service`, `access-control-service`, `reports-service`

#### Payload Schema (JSON):
```json
{
  "eventId": "770e8400-e29b-41d4-a716-446655440020",
  "eventType": "CommercialEstablishmentRegistered",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440001",
  "aggregateType": "CommercialEstablishment",
  "occurredAt": "2026-11-20T22:40:00Z",
  "version": 1,
  "payload": {
    "establishmentId": "880e8400-e29b-41d4-a716-446655440030",
    "unitId": "550e8400-e29b-41d4-a716-446655440001",
    "businessName": "Minimarket Los Andes",
    "businessType": "Comercio de víveres",
    "openingTime": "07:00",
    "closingTime": "21:00"
  },
  "metadata": {
    "correlationId": "770e8400-e29b-41d4-a716-446655440022",
    "userId": "550e8400-e29b-41d4-a716-446655440004"
  }
}
```

---

## 2. Event Consumption Matrix (Downstream Effects)

| Event | Consuming Service | Action Performed | Idempotency Strategy |
|---|---|---|---|
| `UnitRegistered` | `billing-service` | Generates default tariff association based on `unitType` | Dedup via `eventId` |
| `UnitRegistered` | `access-control-service` | Registers unit in local cache for visitor destination validation | Upsert by `unitId` |
| `UnitRegistered` | `communications-service` | Associates unit into the residential or commercial bulletin group | Upsert by `unitId` |
| `UnitUpdated` | `billing-service` | Updates nomenclature and alerts if unit was deactivated (`isDeleted = true`) | Dedup via `eventId` |
| `CommercialEstablishmentRegistered` | `access-control-service` | Enables customer entry validation during establishment opening hours | Upsert by `unitId` |

---

## 3. Reliability & Transactional Guarantees

1. **Transactional Outbox:** Any insert or update to `units` or `commercial_establishments` writes an event record to `outbox_events` within the same ACID database transaction.
2. **Outbox Poller / Debezium CDC:** A background scheduled process polls unpublished outbox records (`status = 'PENDING'`) and delivers them to RabbitMQ, marking them as `PUBLISHED` only after acknowledgment (`ACK`).
