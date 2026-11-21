# Domain Events — `api-gateway`

> **`api-gateway` does not publish or consume domain events on RabbitMQ.**

As a synchronous HTTP edge infrastructure component (`ADR-006`), `api-gateway` operates strictly on the request/response plane:
- **Published Events:** None.
- **Consumed Events:** None.
- **Observability Emissions:** Emits structured JSON HTTP access/security logs (`X-Correlation-Id`, status code, latency, route ID, rate-limit rejections) and Micrometer/Prometheus metrics (`NFR-005`).
