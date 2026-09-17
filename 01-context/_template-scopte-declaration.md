# Scope Declaration — resi-complex

**Formal Statement for Stakeholders & Presentations**

---

## Executive summary

**resi-complex** is a microservices-based system for the integrated management of mixed-use residential complexes (residential units + commercial establishments). The system manages units, tenants/owners, maintenance, billing, access control, communications, and extraordinary expense approval through a centralized, auditable platform.

**Target:** One residential complex with residential and commercial units.  
**Timeline:** 6 weeks (formative ADSO SENA delivery).  
**Team:** [N] developers, 1 QA lead, 1 Tech Lead, 1 Product Owner.

---

## In scope (MVP) — Formative delivery

### Core capabilities (19 functional requirements, 8 non-functional requirements)

#### 1. Identity & Access (IAM Service)
- ✅ Role-based authentication (5 roles: Administrator, Board, Person, Maintenance Staff, Security Guard)
- ✅ JWT token generation and refresh
- ✅ Role-specific permission enforcement

#### 2. Unit management (Units Service)
- ✅ Register residential and commercial units
- ✅ Edit and delete units (with validation)
- ✅ Register commercial establishment data (business type, hours)

#### 3. People management (People Service)
- ✅ Register residents and commercial tenants/owners
- ✅ Link people to units
- ✅ Manage person-unit relationships

#### 4. Maintenance (Maintenance Service)
- ✅ Create maintenance requests with type, priority, description
- ✅ Assign requests to maintenance staff
- ✅ Track request lifecycle (PENDING → ASSIGNED → IN_PROGRESS → RESOLVED)
- ✅ Notify requesters of status changes

#### 5. Billing (Billing Service)
- ✅ Auto-generate monthly fees per unit
- ✅ Differentiated rates by unit type (residential vs. commercial)
- ✅ Track fee payment status
- ✅ View pending fee arrears

#### 6. Communications (Communications Service)
- ✅ Publish announcements segmented by unit type (residential / commercial / all)
- ✅ Notify users of relevant events (maintenance status, fee reminders)

#### 7. Access Control & Front Desk (Access Control Service)
- ✅ Log visitor entry/exit (personal visitor vs. commercial client)
- ✅ Register vehicles associated with visits
- ✅ Log incoming correspondence (packages, letters)
- ✅ Allow residents to check pending correspondence

#### 8. Financial Approval (Finance Approval Service)
- ✅ Create proposals for extraordinary expenses
- ✅ Approve/reject proposals (Board of Trustees only)
- ✅ Maintain permanent audit trail of decisions

#### 9. Reporting (Reports Service)
- ✅ Maintenance requests by status report
- ✅ Pending fee arrears report
- ✅ Read-only views built from domain events

### Non-functional requirements

- ✅ Performance: P95 < 300ms on critical endpoints (50 RPS load test)
- ✅ Availability: 99.9% SLO target (Local environment only)
- ✅ Security: JWT auth, bcrypt passwords, OWASP Top 10 alignment, Ley 1581 de 2012 (Colombian data protection)
- ✅ Scalability: Horizontal auto-scaling up to 3 instances per service
- ✅ Observability: Structured JSON logs + correlation IDs + Prometheus metrics
- ✅ Maintainability: ≥ 80% test coverage (≥ 90% in domain layer)
- ✅ Portability: Docker images, environment-based configuration
- ✅ Disaster Recovery: RTO < 2 min (single service), < 5 min (DB failover)

---

## Out of scope (MVP)

### Infrastructure & deployment
- ❌ Staging environment (pending definition)
- ❌ Production environment (not scheduled)
- ❌ Container orchestration platform (Kubernetes or alternative — TBD)
- ❌ Cloud provider (AWS, Azure, GCP — not yet decided)
- ❌ CI/CD pipeline automation (planned, not implemented)

### Business features
- ❌ Payment gateway integration (residents pay in person or by manual transfer)
- ❌ Email/SMS notifications (in-app notifications only for MVP)
- ❌ Multi-property management (one complex only)
- ❌ SaaS multi-tenancy (single-tenant application)
- ❌ Mobile app (web app only)
- ❌ Analytics & business intelligence beyond basic reports
- ❌ Integration with third-party admin systems

### Advanced features
- ❌ Visitor pre-registration
- ❌ Automatic fine calculation for late fees
- ❌ Budget forecasting
- ❌ Document management system
- ❌ Video surveillance integration

### Compliance & audit (at MVP)
- ⚠️ Full Ley 1581 implementation (basic compliance only; full audit trail will be prepared)
- ⚠️ SOC 2 audit (scheduled for post-formative phase)

---

## Candidates for future versions

### Version 2 (Post-formative)
- Multi-property management (same system, many complexes)
- Payment gateway integration (Stripe, PayU)
- Email & SMS notifications
- Mobile app (iOS, Android)
- Visitor pre-registration portal
- Advanced reporting & analytics
- Video surveillance system integration

### Version 3+
- SaaS licensing model
- API for third-party integrations
- Budget forecasting and variance analysis
- Compliance certifications (SOC 2, ISO 27001)
- Full AI-driven maintenance prediction

---

## Approval

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Product Owner | [Name] | [ ] | [ ] |
| Tech Lead | [Name] | [ ] | [ ] |
| Scrum Master | [Name] | [ ] | [ ] |
| Instructor/Stakeholder | [Name] | [ ] | [ ] |

---

**Version:** 1.0  
**Last updated:** [Date]  
**Valid until:** [Date of next review]

---

## How to use this document

- **For the team:** Reference when deciding whether a feature is in scope
- **For stakeholders:** Understand what will be delivered and what won't
- **For presentations:** Present this to confirm alignment before starting a sprint
- **For change control:** If scope changes, update this document and re-obtain signatures
