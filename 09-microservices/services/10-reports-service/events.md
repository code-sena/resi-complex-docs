# Domain Events — Reports Service

> Specifies asynchronous domain events consumed by `resi-complex-reports-service`.  
> Broker: **RabbitMQ** | Exchange: `resicomplex.events` (Topic Exchange)

---

## 1. Events Published

`reports-service` is a terminal CQRS read-projection service and **does not publish domain events**.

---

## 2. Events Consumed

| Event | Source Service | Routing Key | Projection Updated |
| :--- | :--- | :--- | :--- |
| `FeeGenerated` | `billing-service` | `billing.fee.generated` | Increments `total_pending` and updates `overdue_months` in `pending_fees_view`. |
| `FeePaid` | `billing-service` | `billing.fee.paid` | Decrements `total_pending` and resets/decrements `overdue_months` in `pending_fees_view`. |
| `MaintenanceRequestCreated` | `maintenance-service` | `maintenance.request.created` | Increments `request_count` for `status = 'PENDING'` in `requests_by_status_view`. |
| `MaintenanceRequestStatusUpdated` | `maintenance-service` | `maintenance.request.status_updated` | Decrements `previousStatus` count and increments `newStatus` count in `requests_by_status_view`. |
