# Domain Events — `iam-service`

> Published reliably via the local `iam_db.outbox` table to RabbitMQ Topic Exchange **`resicomplex.events`** (`ADR-005`).

---

## Published Events

### 1. `UserRegistered`
- **Routing Key:** `iam.user.registered`
- **Trigger:** Successful user registration or admin staff provisioning (`FR01`, `FR20`).
- **Consumers:** `communications-service` (welcome in-app notice), audit projections.
- **Payload Example:**
```json
{
  "eventId": "8f2a9c10-4b2e-4f1a-9a3b-1234567890ab",
  "eventType": "UserRegistered",
  "version": "1.0",
  "occurredAt": "2026-11-21T00:00:00Z",
  "aggregateId": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "userId": "550e8400-e29b-41d4-a716-446655440000",
    "role": "PERSON",
    "status": "PENDING_ACTIVATION"
  }
}
```
*(Zero passwords or PII beyond `userId` and canonical `role` are emitted in event payloads per `00-governance/security-policy.md`.)*

### 2. `UserActivated`
- **Routing Key:** `iam.user.activated`
- **Trigger:** Provisioned staff/resident completes account activation (`FR20`).
- **Consumers:** `communications-service`, audit logs.

---

## Consumed Events

- **None.** `iam-service` is an upstream Open Host Service (`02-domain/domain-map.md`) with zero runtime dependencies on business microservices.
