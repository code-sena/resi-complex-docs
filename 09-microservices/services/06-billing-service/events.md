# Domain Events — Billing Service

> Specifies asynchronous domain events published and consumed by `resi-complex-billing-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

### `FeeGenerated`
Emitted when a monthly administration fee is created for a residential or commercial unit.

* **Routing Key:** `billing.fee.generated`
* **Producer:** `resi-complex-billing-service`
* **Consumers:** `communications-service` (resident notification), `reports-service` (portfolio projection update)

#### Payload Schema:
```json
{
  "eventId": "b12c34d5-e6f7-4890-a1b2-c3d4e5f60001",
  "eventType": "FeeGenerated",
  "occurredAt": "2026-11-01T08:00:00Z",
  "data": {
    "feeId": "550e8400-e29b-41d4-a716-446655440101",
    "unitId": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
    "unitType": "COMMERCIAL",
    "amount": 450000.00,
    "billingMonth": 11,
    "billingYear": 2026,
    "dueDate": "2026-11-30",
    "status": "PENDING"
  }
}
```

---

### `FeePaid`
Emitted when a pending or overdue administration fee is marked as paid.

* **Routing Key:** `billing.fee.paid`
* **Producer:** `resi-complex-billing-service`
* **Consumers:** `reports-service` (decrements pending portfolio balance in `pending_fees_view`)

#### Payload Schema:
```json
{
  "eventId": "c23d45e6-f7a8-4901-b2c3-d4e5f6000002",
  "eventType": "FeePaid",
  "occurredAt": "2026-11-20T15:30:00Z",
  "data": {
    "feeId": "550e8400-e29b-41d4-a716-446655440101",
    "unitId": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
    "paidAmount": 450000.00,
    "paymentMethod": "PSE",
    "paymentReference": "PSE-20261120-99481",
    "status": "PAID"
  }
}
```

---

## 2. Events Consumed

| Event | Source Service | Routing Key | Action Triggered |
| :--- | :--- | :--- | :--- |
| `UnitRegistered` | `units-service` | `units.unit.registered` | Registers unit metadata (`unitId`, `unitType`) for automated monthly fee generation cycles. |
