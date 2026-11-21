# Domain Events — Communications Service

> Specifies asynchronous domain events published and consumed by `resi-complex-communications-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

### `AnnouncementPublished`
Emitted immediately when an administrator creates and publishes a new announcement for the complex.

* **Routing Key:** `communications.announcement.published`
* **Producer:** `resi-complex-communications-service`
* **Consumers:** `reports-service`, Mobile/Web Notification clients

#### Payload Schema:
```json
{
  "eventId": "c71a391a-7b3b-47e0-9bc2-3b2d184285cb",
  "eventType": "AnnouncementPublished",
  "occurredAt": "2026-11-20T10:15:30Z",
  "data": {
    "announcementId": "8fa1f40b-7128-4efc-8b7a-9a99ef3c091d",
    "title": "Suspensión programada de suministro de agua potable",
    "targetScope": "ALL",
    "publishedBy": "550e8400-e29b-41d4-a716-446655440000",
    "status": "PUBLISHED"
  }
}
```

---

## 2. Events Consumed

The Communications Service can subscribe to key operational events across the system to trigger automated occupant notifications:

| Event | Source Service | Routing Key | Action Triggered |
| :--- | :--- | :--- | :--- |
| `CorrespondenceReceived` | `access-control-service` | `access.correspondence.received` | Dispatches an in-app alert to the recipient resident regarding a waiting package at the front desk. |
| `MaintenanceRequestStatusUpdated` | `maintenance-service` | `maintenance.request.status_updated` | Dispatches notification when a reported maintenance request moves to `IN_PROGRESS` or `RESOLVED`. |
| `FeeGenerated` | `billing-service` | `billing.fee.generated` | Notifies unit occupants that the monthly administration fee invoice is available. |
| `ExpenseProposalApproved` | `finance-approval-service` | `finance.expense.approved` | Creates a bulletin entry for transparency when a major expense is approved by the Board. |

---

## 3. Resilience & Delivery Guarantees

1. **Transactional Outbox:** State changes in `communications_db` and event publication are bound in the same database transaction.
2. **Idempotent Consumer:** Incoming event messages are deduplicated by `eventId` in an `idempotent_events` table before applying notification side-effects.
