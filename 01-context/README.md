# 01 — Project Context

> **What is this?** The "why" of the system. Anyone new must be able to read this
> folder and understand what problem the project solves, what it includes, and what it does NOT include.

## Why this section exists

Before designing anything, the team needs to agree on:
- What problem are we solving?
- For whom?
- What is in scope and what is out of scope?
- What does each term we use mean?

Without this, each team member works with different assumptions and the project fragments.

---

## What is here and how to fill it in

### `overview.md` ⭐
Executive description of the system in maximum 1 page.

**Status:** ✅ COMPLETED

**Contains:**
- **System name:** resi-complex
- **Problem it solves:** Administrators of mixed-use residential complexes in Colombia manage units, tenants, maintenance, fees, and guests using disconnected tools (notebooks, Excel, WhatsApp). Result: lost information, no audit trail, operational inefficiency.
- **Solution:** Single integrated platform where 5 roles (Administrator, Board of Trustees, Residents, Maintenance Staff, Security) manage everything from one place.
- **Main users:**
  - Administrator: unit management, billing, expense approval
  - Resident/Commercial Owner: report maintenance, check fees and correspondence
  - Security Guard: log visitors and incoming packages
  - Maintenance Staff: accept and complete requests
  - Board of Trustees: approve extraordinary expenses
- **Technology stack:**
  - Backend: Java 17+ + Spring Boot 3.x
  - Database: MySQL 8.0+
  - Infrastructure: Docker, event-driven microservices
  - Current status: Under development (formative ADSO SENA, 6 weeks)

---

### `scope.md` ⭐
System boundaries: what it does and what it does NOT do.

**Status:** ✅ COMPLETED

**In scope (MVP):**
- Unit registration & classification (residential/commercial)
- Person management (residents & commercial tenants/owners)
- IAM: 5 roles with JWT authentication
- Maintenance: create, assign, track requests
- Billing: auto-generate fees with differentiated rates by unit type
- Access Control: log visitors (entry/exit, vehicles), incoming correspondence
- Communications: segmented announcements by unit type
- Financial: create & approve extraordinary expense proposals
- Reports: maintenance status & fee arrears (read-only from events)

**Out of scope (MVP):**
- Payment gateway integration
- Email/SMS notifications (in-app only)
- Multi-property management (one complex)
- Mobile app
- Staging/Production infrastructure (Local environment only)
- Advanced analytics, visitor pre-registration, SaaS multi-tenancy

**Future versions:**
- Multi-property SaaS
- Payment integration (Stripe, PayU)
- Mobile app, email/SMS, AI-driven maintenance prediction

---

### `glossary.md` ⭐
Dictionary of the project domain.

**Status:** ✅ COMPLETED

**Contains 20+ key terms with precise definitions:**
- Unit: A property (residential or commercial) in the complex
- Resident: Person occupying a residential unit
- Commercial Owner/Tenant: Person occupying a commercial unit
- Fee: Monthly administration charge per unit (rate varies by type)
- Maintenance Request: Problem reported by a resident (type, priority, status lifecycle)
- Visit: Record of a visitor's entry/exit
- Correspondence: Incoming package or letter for a unit
- Expense Proposal: Request for extraordinary spending (requires Board approval)
- Board of Trustees: 5+ residents who approve extraordinary expenses
- Domain Event: Fact that occurred (FeeGenerated, MaintenanceRequestCreated, etc.)

(See full definitions in glossary.md)

---

### `_template-project-profile.md`
Project technical sheet for internal records.

**Status:** ✅ CREATED (with resi-complex values pre-filled)

**Contains:**
- Project identity (name: resi-complex, status: under development)
- Team roles & contacts (PO, Scrum Master, Tech Lead, Dev team, QA)
- Key dates & milestones (formative delivery target)
- Locked technology stack (Java 17+, Spring Boot 3.x, MySQL 8.0+, Docker)
- Budget & resources, external dependencies, assumptions
- Communication channels (daily standup, sprint ceremonies)
- Success criteria (19 HUs, ≥80% test coverage, OWASP compliance)

---

### `_template-scope-declaration.md`
Formal scope declaration template for presentations or deliverables.

**Status:** ✅ CREATED (with resi-complex scope pre-filled)

**Contains:**
- Executive summary of what resi-complex is and for whom
- 19 functional requirements grouped by service (IAM, Units, People, Maintenance, Billing, Communications, Access Control, Finance, Reports)
- 8 non-functional requirements (performance, availability, security, scalability, observability, maintainability, portability, disaster recovery)
- Explicit out-of-scope list (no Staging/Production yet, no payment gateway, no multi-tenant, no mobile)
- Future version roadmap (v2, v3+)
- Signature/approval section for stakeholder alignment

---

## Correlations with other sections

| If you change this... | Also review... | Why |
|-----------------------|----------------|-----|
| The problem described in `overview.md` | `03-product/vision.md`, `03-product/problem-framing.md` | Vision and roadmap must align with the problem statement |
| The scope in `scope.md` | `04-requirements/README.md` (19 FRs), `04-requirements/user-stories.md` (19 HUs), `03-product/vision.md` (3 sprints) | Every feature in scope must be a requirement and a user story |
| A term in `glossary.md` | `02-domain/entities-and-rules.md` (9 aggregates), `02-domain/domain-events.md` (27 events) | Domain model must use the same terms consistently |
| Any scope change | `04-requirements/traceability-matrix.md`, `00-governance/agile-conventions.md` (change control) | Scope creep breaks requirements traceability; requires team alignment |
| Technology stack decisions | `05-architecture/overview.md`, `09-microservices/` (9 services definition) | Tech stack impacts architecture and service design |

---

## Recommended fill order

**For resi-complex (FORMATIVE DELIVERY):**

1. ✅ `overview.md` — **DONE** (30 minutes, full team aligned)
2. ✅ `scope.md` — **DONE** (1 hour discussion, scope locked, no creep)
3. ✅ `glossary.md` — **DONE** (20+ key terms, used consistently across domain model and requirements)
4. ✅ `_template-project-profile.md` — **DONE** (project metadata, team, stack, success criteria)
5. ✅ `_template-scope-declaration.md` — **DONE** (formal scope statement ready for stakeholder presentations)

**All 5 files in 01-context are now complete and serve as the foundation for:**
- 19 functional requirements (`04-requirements/functional.md`)
- 19 user stories (`04-requirements/user-stories.md`)
- 9 bounded contexts & microservices (`02-domain/domain-map.md`)
- 27 domain events (`02-domain/domain-events.md`)
- 3-sprint roadmap (`03-product/vision.md`)

---

## Questions this section must answer

| Question | Answer (resi-complex) | Location |
|----------|----------------------|----------|
| **What does this system exist for?** | To replace disconnected tools (notebooks, Excel, WhatsApp) with a single integrated platform for managing mixed-use residential complexes | `overview.md` |
| **Who are the users?** | 5 roles: Administrator, Board of Trustees, Residents/Commercial Owners, Maintenance Staff, Security Guard | `overview.md` |
| **What does the system NOT do?** | No payment gateway, no email/SMS (in-app only), no multi-property yet, no Staging/Production, no mobile app, no SaaS multi-tenancy | `scope.md` |
| **What does [Unit] mean in this project?** | A property (residential or commercial) in the complex that occupies a physical address and generates a monthly fee | `glossary.md` |
| **What does [Maintenance Request] mean?** | A problem reported by a resident with type, description, priority, and a lifecycle: PENDING → ASSIGNED → IN_PROGRESS → RESOLVED | `glossary.md` |
| **What does [Expense Proposal] mean?** | A request for extraordinary spending, created by Administrator, reviewed and approved/rejected by Board of Trustees, with permanent audit trail | `glossary.md` |
| **Who decides what gets built?** | Product Owner, informed by scope.md and vision.md; team aligns in sprint planning | `_template-project-profile.md` (team roles) |
| **When will it be done?** | Formative delivery: 6 weeks (3 sprints of 2 weeks each). Local environment only. Staging/Production: post-formative | `_template-project-profile.md` (key dates) |
| **How do we know we're done?** | All 19 HUs with passing acceptance criteria, ≥80% test coverage, documentation complete, performance NFRs validated | `_template-project-profile.md` (success criteria) |
