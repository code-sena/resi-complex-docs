# Domain Events — Access Control Service

> Specifies asynchronous domain events published and consumed by `resi-complex-access-control-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

### `VisitRegistered`
Emitted immediately when the guard logs an admission at the entrance booth.

* **Routing Key:** `access.visit.registered`
* **Payload:**
```json
{
  "eventId": "e1f1816f-ec59-4b13-90d1-0536d934bb6a",
  "eventType": "VisitRegistered",
  "occurredAt": "2026-11-20T14:30:00Z",
  "data": {
    "visitId": "71333d07-2ee5-4b0c-99d7-84bc7bf807b5",
    "visitorName": "Juan Carlos Pérez",
    "documentNumber": "1098765432",
    "destinationUnitId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "visitType": "PERSONAL_VISITOR",
    "vehiclePlate": "XYZ-789",
    "entryTime": "2026-11-20T14:30:00Z"
  }
}
```

---

### `VisitEnded`
Emitted when the visitor departs and the guard marks the exit.

* **Routing Key:** `access.visit.ended`
* **Payload:**
```json
{
  "eventId": "f2e29270-fd6a-4c24-a1e2-1647ea45cc7b",
  "eventType": "VisitEnded",
  "occurredAt": "2026-11-20T16:45:10Z",
  "data": {
    "visitId": "71333d07-2ee5-4b0c-99d7-84bc7bf807b5",
    "exitTime": "2026-11-20T16:45:10Z",
    "durationMinutes": 135
  }
}
```

---

### `CorrespondenceReceived`
Emitted when a package or envelope arrives at the front desk.

* **Routing Key:** `access.correspondence.received`
* **Consumer:** `communications-service` (triggers instant in-app alert to resident)
* **Payload:**
```json
{
  "eventId": "a9b8c7d6-e5f4-4321-abcd-ef0123456789",
  "eventType": "CorrespondenceReceived",
  "occurredAt": "2026-11-20T11:05:00Z",
  "data": {
    "correspondenceId": "4c3b2a10-9876-5432-10fe-dcba98765432",
    "unitId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "recipientName": "María Fernanda Gómez",
    "packageType": "BOX",
    "courier": "Servientrega"
  }
}
```

---

### `CorrespondenceDelivered`
Emitted when the recipient claims their mail at the front desk.

* **Routing Key:** `access.correspondence.delivered`
* **Consumer:** `reports-service` (audits delivery velocity and resolution rate)

---

## 2. Events Consumed

| Event | Source Service | Routing Key | Purpose |
| :--- | :--- | :--- | :--- |
| `UnitRegistered` | `units-service` | `units.unit.registered` | Updates internal cache of valid unit numbers for rapid auto-completion at the guard console. |
