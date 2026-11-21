# 03 — Product Definition

> **What is this?** The answer to "what are we going to build?". It is not "how" — that comes in
> architecture. Here the validated problem, product vision, and build plan for **resi-complex**
> (residential complex management system) are defined.

## Why this section exists

Without a clear product definition:
- The team builds features nobody asked for
- Scope grows out of control (scope creep) — for example, building online payments or visitor
  QR pre-authorization before the MVP, when both are explicitly out of scope (see `01-context/scope.md`)
- There is no way to know whether the project was successful

This section is the contract between the team and the instructor about **what resi-complex will build and why**.

---

## What is here and how to fill it in

### `problem-framing.md` ⭐ (Start here) — ✅ Complete
Articulates the problem before proposing solutions.
**Contains:** administrators and Boards of Trustees of small-to-medium mixed residential
complexes struggling with fee tracking, maintenance traceability, and expense-approval records
because they rely on notebooks, Excel, and WhatsApp instead of a centralized system.

**Format used:**
```markdown
## The problem
**Who has it?** Administrators and Boards of Trustees of small-to-medium mixed
  residential complexes (residential + commercial units)
**What problem do they have?** Visitor disorganization, no differentiated-fee tracking,
  no maintenance traceability, no formal expense-approval record
**When does it occur?** Every day, using notebooks, Excel, and WhatsApp instead of one system
**What is the impact?** High monthly reconciliation friction, billing disputes with commercial units, lost maintenance requests, unverified parcel handovers, and zero auditable Board history — synthesized in `discovery-brief.md`
**How do they solve it today?** Manually and in a fragmented way (see `problem-framing.md` §4, Current Solution)

## Why it is worth solving
No validated competitor in the Colombian market (ConjuntoApp, TUCO 360, Edifia, PH360)
natively covers residential + commercial units with differentiated fees — see
`resi-complex-tech-watch.md`.
```

### `discovery-brief.md` — ✅ Complete
Consolidated Phase 0 Discovery synthesis: 5 user personas, validated vs. deferred product hypotheses, Colombian PH regulatory & technical feasibility, and DoR/DoD quality agreements.

### `admin-survey.md`, `board-of-trustees-survey.md`, `maintenance-survey.md`, `resident-merchant-survey.md`, `security-guard-survey.md` — ✅ Complete (Applied & Synthesized)
Role-based discovery instruments, one per resi-complex authentication role: `ADMINISTRATOR`, `BOARD`, `MAINTENANCE_STAFF`, `PERSON` (Resident/Commercial Owner-Tenant), and `SECURITY_GUARD` (see `00-governance/security-policy.md` and `01-context/glossary.md`). Findings are synthesized in `discovery-brief.md`.

### `interview-resi-complex.md` — ✅ Complete (Exempted under `ADR-002`)
Raw Spanish semi-structured interview scripts (opening, current pain, reaction to the proposed system, closing) for Administrator and Board of Trustees, preserved in source language per `ADR-002` and synthesized in English inside `discovery-brief.md`.

### `resi-complex-tech-watch.md` — ✅ Complete (Exempted under `ADR-002`)
Technology watch for resi-complex: validated competitors (ConjuntoApp, TUCO 360, Edifia, PH360), patent landscape (Decisión 486 CAN — protected by copyright, not patents), and evaluated trends (blockchain and full document-AI: not adopted; online payments and QR camera scanning: future v2.0 candidates; differentiated fee billing: core MVP `FR08–FR10`).

### `vision.md` ⭐ — ✅ Complete
The product's north star (Geoffrey Moore template), strategic pillars, 3 delivery horizons (`H1`–`H3`), product principles, and Product Definition of Done (`KR1`–`KR4`).

### `roadmap.md` — ✅ Complete
Horizon-based delivery roadmap (`Cut 1 / H1` → `Cut 2 / H2` → `Cut 3 / H3` → `v2.0+`), aligned 1:1 with `FR01–FR22`, the 9 microservices + `api-gateway`, team owners, exit gates, and live GitHub repository progress.

### `product-backlog.md` ⭐ — ✅ Complete
Prioritized Product Backlog across **9 Epics, 22 Functional Requirements (`FR01–FR22`), 22 User Stories (`115 SP`)**, and 5 Cross-Cutting Technical Enablers (`TECH-01`–`TECH-05`), ordered by MVP business value and inter-service dependency.

---

## User Story format

Every HU in `04-requirements/user-stories.md` follows the canonical Given/When/Then BDD format using resi-complex's 5 real roles (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`) and 9 canonical microservices (`iam-service`, `units-service`, `people-service`, `maintenance-service`, `billing-service`, `communications-service`, `access-control-service`, `finance-approval-service`, `reports-service`).

---

## Correlations with other sections

| This section feeds... | Why |
|-----------------------|-----|
| `04-requirements/functional.md` & `user-stories.md` | The prioritized backlog (`FR01–FR22`, 22 HUs) is formalized into MoSCoW requirements, `BR-01`–`BR-16`, and Gherkin acceptance criteria |
| `02-domain/domain-map.md` | `problem-framing.md`, `discovery-brief.md`, and `vision.md` define the 9 Bounded Contexts (IAM, Units, People, Maintenance, Billing, Communications, Access Control, Financial Approval, Reports) |
| `05-architecture/overview.md` | Strategic pillars, feasibility constraints, and horizons drive `ADR-003`–`ADR-006` |

---

## Questions this section must answer

- **What problem exactly are we solving?** → `problem-framing.md` and `discovery-brief.md`
- **What does product success look like for resi-complex?** → `vision.md` (OKRs `KR1–KR4`) and `problem-framing.md` (North Star Metric)
- **What do we build first and why?** → `roadmap.md` and `product-backlog.md` (Cut 1: IAM, Gateway, Units, People → Cut 2: Operations → Cut 3: Governance)
- **What do we NOT build in this cycle?** → `01-context/scope.md` and `problem-framing.md` §8: full NIIF accounting, physical access hardware, online payment gateways, digital general assemblies, common-area reservations, document AI, external WhatsApp/SMTP channels, and camera QR-pass scanning (`FR22` digital pre-authorization IS in MVP Sprint 4)
