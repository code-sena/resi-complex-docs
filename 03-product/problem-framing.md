# Problem Framing — Problem Definition

> **Why this document exists:** Before designing solutions, the team must be
> aligned on the problem it solves. This document captures that alignment.
> A well-defined problem is already halfway to a solution.

---

## 1. The problem in one sentence

> Complete this template:

**Administrators and Boards of Trustees of small-to-medium mixed residential complexes (residential and commercial units)** who **manage these processes manually and in a fragmented way (notebooks, Excel, WhatsApp)** struggle with **visitor disorganization, difficulty tracking differentiated fees, lack of traceability in maintenance requests, and no formal record of extraordinary expense approvals**
because **they lack a centralized system that integrates these processes and distinguishes residential from commercial units**, resulting in **a lack of transparency toward residents and commercial owners, and no traceability for the Board of Trustees when reporting decisions**.


---

## 2. Affected users

| Segment | Description | Estimated size | Priority |
|---------|-------------|---------------|---------|
| Administrator | Manages units, people, fees; approves/assigns requests; publishes announcements; proposes extraordinary expenses | 1 per complex (typical) | High |
| Board of Trustees | Approves/rejects expense-budget proposals; reviews financial reports | 3–7 members per complex (typical in Colombian PH) | High |
| Resident (Person subtype) | Creates maintenance requests, checks fees, sees residential announcements and correspondence | Tens to hundreds, depending on complex size | Medium |
| Commercial owner/tenant (Person subtype) | Same as resident, but for commercial units, with differentiated fees and announcements | Variable, typically smaller than residential | Medium |
| Maintenance staff | Sees assigned requests and updates their status | 1–5 per complex | Medium |
| Security guard | Registers visitor/vehicle entry-exit and incoming correspondence | 1–4 per complex (per shift) | Medium |

*Estimated sizes to be validated with real data once the role-based surveys are applied.(`03-product/*-survey.md`).*

### Jobs-to-be-done (JTBD)

> What job is the user trying to do when they "hire" our product?

- **When** I need to generate and collect differentiated fees by unit type, **I want** a system that automatically calculates and records them per unit, **so that** I don't have to do it manually in Excel and I can show transparency to residents and merchants.
- **When** I receive a maintenance request, **I want** to assign it, track it, and make its status visible to whoever created it, **so that** it doesn't get lost or forgotten, and there is traceability of what was done.
- **When** I need to approve an extraordinary expense as a Board member, **I want** to see the proposal with its justification and record my decision, **so that** there is a verifiable history of approvals/rejections.
- **When** a visitor or a package arrives, **I want** to register it and link it to the correct unit, **so that** the resident or merchant knows they have pending correspondence or who is visiting them.

---

## 3. Evidence of the problem

> The problem must be real. Document the evidence gathered during Phase 0 Discovery (synthesized in `03-product/discovery-brief.md`).

| Evidence type | Source | Date | Key finding |
|--------------|--------|------|------------|
| User interviews | Role-based interview scripts (`03-product/interview-resi-complex.md`) | Phase 1 Discovery | Administrators and Board members confirmed that manual Excel fee tracking and physical meeting minutes cause frequent disputes and lack of auditable history |
| Surveys | 5 role-based surveys (Admin, Board, Person, Maintenance, Security — `03-product/*-survey.md`) | Phase 1 Discovery | Confirmed pain points across all 5 roles: unsegmented WhatsApp notices, unverified maintenance closures, slow paper front-desk logs, and opaque fee calculations |
| Benchmarking | ConjuntoApp, TUCO 360, Edifia, PH360 (`03-product/resi-complex-tech-watch.md`) | Phase 1 Discovery | None of the 4 validated Colombian PH competitors natively covers residential + commercial units with differentiated rates and segmented rules — that is resi-complex's primary differentiation gap |
| Direct observation | Mixed-use Colombian PH operations (Ley 675 de 2001 baseline) | Phase 1 Discovery | Small-to-medium complexes rely on notebooks, Excel, and WhatsApp, causing visitor disorganization, billing errors, zero maintenance traceability, and unrecorded Board expense approvals |

---

## 4. Current user solution (and its problems)

> How does the user solve the problem today?

| Current solution | Limitations | Cost/Friction |
|-----------------|------------|--------------|
| Physical notebooks (visitors, correspondence) | No traceability, not remotely consultable, illegible handwriting, can be lost or damaged | High operational friction at shift changes; unverified parcel handovers trigger resident disputes |
| Excel (fees, units) | Does not natively enforce residential vs. commercial rate rules, prone to formula/copy-paste errors, no audit history | Hours of monthly manual reconciliation per billing cycle; frequent billing disputes with commercial tenants |
| WhatsApp (announcements, maintenance requests) | Does not segment by unit type, messages get buried, no formal status, assignee, or priority | Urgent repairs missed; residents and commercial tenants receive irrelevant notices; zero SLA visibility |
| Meetings/physical minutes or email (expense approval) | No centralized, queryable history; hard to verify Board quorum and individual votes | Slow approval cycles and low transparency during annual assemblies |

---

## 5. Solution hypothesis

> This is the product solution hypothesis validated through Phase 0 Discovery and materialized in `05-architecture/`.

**We believe that** a web-based microservices system (Java 25 + Spring Boot 4.0.8 + MySQL 8 + RabbitMQ) that centralizes the management of residential and commercial units, differentiated fees, maintenance, access control, correspondence, segmented communications, and extraordinary expense approvals
**for** administrators, Boards of Trustees, residents, commercial owners/tenants, maintenance staff, and security guards of small-to-medium residential complexes,
**will achieve** greater transparency toward residents and merchants, and full traceability for the Board of Trustees.
**We will know we succeeded when** the pilot complex migrates its core manual workflows (Excel/WhatsApp/notebooks) into `resi-complex` and 100% of fees, maintenance requests, front-desk logs, and Board approvals are recorded and queryable with an auditable trail.

---

## 6. Success metrics (North Star)

| Metric | Current baseline | Target (MVP Pilot / 6-month) | How to measure it |
|--------|----------------|------------------------------|-------------------|
| API response time (`NFR-001`) | N/A (manual tools) | P95 < 300 ms under 50 RPS (all normal operations < 2 s) | k6 load tests & Prometheus metrics (`NFR-001`, `NFR-005`) |
| % of maintenance requests tracked end-to-end with verified status | 0% (manual/WhatsApp) | ≥ 95% of requests logged & closed via `WORK_COMPLETED → RESOLVED` | `reports-service` maintenance breakdown (`FR05–FR07`, `FR19`) |
| % of monthly fees generated automatically with differentiated rates | 0% (manual Excel) | 100% of active residential & commercial units billed automatically | `billing-service` & `reports-service` portfolio reports (`FR08–FR10`, `FR19`) |
| % of extraordinary expense proposals with immutable Board vote history | 0% (physical minutes) | 100% of proposals resolved with append-only quorum record | `finance-approval-service` audit ledger (`FR17–FR18`) |
| Accessibility & mobile readiness (`NFR-009`) | N/A | 100% of core flows passing WCAG 2.1 AA (`axe-core`) down to `360px` | Automated `axe-core` & Lighthouse audits (`12-ux-ui/`) |

**North Star Metric:** **Percentage of core complex administration workflows** (differentiated monthly billing, maintenance lifecycle, front-desk visitor/correspondence logging, and Board expense approvals) **executed end-to-end inside `resi-complex` with a verifiable audit trail** — target **≥ 90%** within 6 months of pilot adoption.

---

## 7. Hypothesis risks

| Risk | Probability | Impact | Mitigation / Validation Experiment |
|------|------------|--------|------------------------------------|
| Administrators/guards resist adopting the system over notebooks/Excel/WhatsApp | Medium | High | Validate responsive high-fidelity Stitch UI prototypes (`12-ux-ui/wireframes.md`) with minimal-click front-desk flows (`Alt+N` shortcuts, 4-digit pickup PIN) |
| Architectural drift between documentation (`resi-complex-docs`) and microservice repos | Medium | High | Enforce Contract-First OpenAPI linting (`npx @redocly/cli lint`), ArchUnit fitness tests, and `04-requirements/traceability-matrix.md` reviews every sprint |
| MVP scope creep from future candidates (online payments, QR scanners, digital assemblies) | Medium | High | Gate every Sprint Planning against the explicit Out-of-Scope list in `01-context/scope.md` and `03-product/roadmap.md` |
| Uneven delivery pace across Cut 1 foundational services (`units`, `people`, `maintenance`) | Medium | High | Track dependencies explicitly in `03-product/product-backlog.md` and run integration smoke tests behind `api-gateway` (`:8080`) |

---

## 8. Out of scope (we do not solve in MVP v1.0)

> Explicitly synchronized with `01-context/scope.md` to prevent scope creep.

- **Full IFRS/NIIF accounting** — the system records fees and Board expense approvals, but does not replace a certified accounting ledger.
- **Physical access-control hardware** (turnstiles, biometrics, RFID gates) — the system digitally logs entries/exits reported at the front desk (`FR12–FR14`).
- **Online payments** (PSE / card checkout gateways) — fees and partial payment vouchers are recorded and consulted (`FR08–FR10`), but live payment processing is deferred to v2.0.
- **Digital assemblies / general-assembly electronic voting** — only Board of Trustees extraordinary expense voting (`FR17–FR18`) is in MVP scope.
- **Common-area reservations** — not covered by `FR01–FR22`; candidate for v2.0.
- **AI over documents** — evaluated in `resi-complex-tech-watch.md` (only 1 of 4 competitors offers it); not adopted in MVP.
- **External WhatsApp / SMS / SMTP channels** — notifications are delivered in-app via `communications-service` (`FR07`, `FR11`) to keep MVP self-contained with zero paid third-party dependencies.
- **QR-code / camera pass scanning for pre-authorizations** — digital visitor pre-authorization (`FR22`) IS included in Sprint 4 (`access-control-service`), while camera/hardware QR scanning remains deferred to v2.0.

---

## Correlations

- Discovery synthesis & personas → `03-product/discovery-brief.md`
- Product vision & OKRs → `03-product/vision.md`
- Roadmap & prioritized backlog → `03-product/roadmap.md`, `03-product/product-backlog.md`
- HUs that implement this solution → `04-requirements/user-stories.md`
- Detailed operational KPIs → `13-operations/README.md`
