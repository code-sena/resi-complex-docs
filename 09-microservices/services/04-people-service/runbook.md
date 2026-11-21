# Operational Runbook — `people-service`

> **Criticality: P1 (PII Vault & Ownership Source).** Outages block new resident onboarding and `:own` tenancy verification queries.

---

## 1. Health & Diagnostics

```bash
curl -i http://127.0.0.1:8085/health
mysql -u resi_dev -p people_db -e "SELECT status, COUNT(*) FROM unit_occupancies GROUP BY status;"
```

| Symptom | Likely Cause | Immediate Action |
|---------|--------------|------------------|
| `409 Conflict` on person registration | Duplicate `document_id` (`uq_person_document`) | Check existing person record via `GET /api/v1/people?search=<doc>` and attach a new `unit_occupancy` instead of duplicating the person |
| Former tenant still sees old unit fees/parcels | Occupancy row still `ACTIVE` in `people_db.unit_occupancies` | Deactivate via `PATCH /api/v1/occupancies/{id}/deactivate` (`BR-07`); confirm `OccupancyDeactivated` outbox dispatch |
| `422 Unprocessable Entity` on deactivation | `endDate` earlier than `startDate` (`chk_occupancies_dates`) | Verify dates with Administrator and resubmit valid `endDate` |

---

## 2. Privacy & Backup Controls (Ley 1581 de 2012)

- Restrict direct MySQL `people_db` access to least-privilege service account only.
- Verify structured logs redact `documentId`, `phone`, and `email` (`TH-09`).
- **Escalation:** Service Owner (`@AlexandraVargass`) → Tech Lead (`@Trujillo7915`).
