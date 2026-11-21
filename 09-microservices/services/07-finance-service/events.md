# Domain Events — Finance Approval Service

> Specifies asynchronous domain events published and consumed by `resi-complex-finance-approval-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

### `ExpenseProposalCreated`
Emitted when the Administrator submits a new extraordinary expense proposal for Board review.

* **Routing Key:** `finance.expense.created`
* **Producer:** `resi-complex-finance-approval-service`
* **Consumers:** `communications-service` (notifies Board members of pending vote)

#### Payload Schema:
```json
{
  "eventId": "d11e22f3-a4b5-4678-9012-3456789abc01",
  "eventType": "ExpenseProposalCreated",
  "occurredAt": "2026-11-20T10:00:00Z",
  "data": {
    "proposalId": "880e8400-e29b-41d4-a716-446655440801",
    "title": "Impermeabilización de cubiertas Torre A y B",
    "estimatedAmount": 3500000.00,
    "proposedBy": "550e8400-e29b-41d4-a716-446655440000",
    "status": "PENDING_REVIEW"
  }
}
```

---

### `ExpenseProposalApproved`
Emitted when a proposal reaches the required approval quorum from the Board of Directors.

* **Routing Key:** `finance.expense.approved`
* **Producer:** `resi-complex-finance-approval-service`
* **Consumers:** `communications-service`, `reports-service`

---

### `ExpenseProposalRejected`
Emitted when a proposal is rejected by majority vote of the Board of Directors.

* **Routing Key:** `finance.expense.rejected`
* **Producer:** `resi-complex-finance-approval-service`
* **Consumers:** `communications-service`, `reports-service`
