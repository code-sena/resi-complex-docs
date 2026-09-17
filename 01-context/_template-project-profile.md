# Project Profile — resi-complex

> **Purpose:** Internal technical sheet. Summarizes essential metadata about the project in one place.
> **Audience:** Team members, stakeholders, anyone joining the team.

---

## Project identity

| Field | Value |
|-------|-------|
| Official Name | resi-complex |
| Internal code | RC |
| Version | 1.0-alpha (formative) |
| Status | Under development (ADSO SENA — Phases 1–3, 6 weeks) |

---

## Team

| Role | Name | Contact | Responsibilities |
|------|------|---------|------------------|
| Product Owner | [Name] | [Email] | Backlog prioritization, stakeholder alignment |
| Scrum Master | [Name] | [Email] | Process, ceremonies, blockers |
| Tech Lead | [Name] | [Email] | Architecture decisions, technical direction |
| Development Team | [N members] | [Slack/Email] | Implementation of HUs across 9 microservices |
| QA Lead | [Name] | [Email] | Test strategy, acceptance criteria verification |

---

## Key dates

| Milestone | Target date | Status |
|-----------|------------|--------|
| Formative delivery (Phase 1–3 complete) | [Date] | 🟡 In progress |
| MVP launch (Local environment) | [Date] | 🟡 Planned |
| Staging environment | [Date] | 🔴 Not yet scheduled |
| Production environment | [Date] | 🔴 Not yet scheduled |

---

## Technology stack (locked for this project)

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Java | 17+ |
| Framework | Spring Boot | 3.x |
| Database | MySQL | 8.0+ |
| Container | Docker | Latest |
| Event stream | [TBD] | [TBD] |
| API docs | OpenAPI / Swagger | 3.0 |
| Testing | JUnit 5 | Latest |
| Build tool | Maven | 3.8+ |

---

## Budget & resources

| Resource | Allocation |
|----------|-----------|
| Dev team capacity | [N] full-time equivalents |
| QA capacity | [N] full-time equivalents |
| Infrastructure (formative) | Local lab laptops + shared MySQL server |
| Tools budget | Covered by SENA |

---

## External dependencies

| Dependency | Type | Status |
|-----------|------|--------|
| Authentication provider | [None — custom JWT] | ✅ Internal |
| Payment gateway | [Out of scope for MVP] | 🔴 Not applicable |
| Email/SMS provider | [Out of scope for MVP] | 🔴 Not applicable |
| Cloud infrastructure | [Not yet decided] | 🔴 Pending |

---

## Key assumptions

- Single residential complex (not multi-tenant SaaS yet)
- All 5 roles will have accounts in Local environment
- Formative scope: features up to HU-REP-001 only (no scaling, optimization, or production hardening)
- Data will be seeded manually for testing; no bulk import tools

---

## Communication channels

| Channel | Purpose | Frequency |
|---------|---------|-----------|
| Daily standup | Blockers, progress | 15 min daily |
| Sprint planning | HU refinement | Start of sprint |
| Sprint review | Demo to stakeholders | End of sprint |
| Retro | Process improvement | End of sprint |
| Slack / Teams | Ad-hoc questions | As needed |

---

## Success criteria

| Criterion | Definition | Target |
|-----------|-----------|--------|
| All 19 HUs completed | Acceptance criteria pass | 100% |
| Test coverage | Unit + integration tests per FR | ≥ 80% |
| Documentation | ADRs, API contracts, domain events | 100% |
| Performance | P95 < 300ms on critical endpoints | ✅ Measured in load tests |
| Security | OWASP Top 10 + Ley 1581 compliance | Audit ready |

---

**Last updated:** [Date]  
**Maintained by:** [Name]
