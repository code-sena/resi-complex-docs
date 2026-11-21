# 12 — UX/UI

> **What is this?** The complete user experience and visual design system of `resi-complex`:
> route topology across all **22 User Stories (`FR01`–`FR22`)**, role-based access controls across
> the **5 canonical authentication roles**, WCAG 2.1 AA design tokens & domain status badges, and
> high-fidelity Google Stitch screens (`HTML` + `PNG`).
> **Language Compliance:** Written in English per `ADR-001` (rendered Stitch mockups display Colombian Spanish target-user copy for pilot usability).

---

## Status

| File | Status | Notes |
|------|--------|-------|
| [`navigation-map.md`](./navigation-map.md) ⭐ | ✅ Complete | Full route tree, 26-screen map covering all **22 HUs (`FR01`–`FR22`)**, 5-role access matrix (synchronized with `00-governance/security-policy.md`), and 4 end-to-end cross-role user flows |
| [`design-system.md`](./design-system.md) | ✅ Complete | CSS design tokens, base components, complete domain **Status / Priority / Unit Type / Announcement** badge mappings synced with `06-data/models.md`, and WCAG 2.1 AA + `360px` mobile minimums (`NFR-009`) |
| [`wireframes.md`](./wireframes.md) | ✅ Complete | High-fidelity Google Stitch mockups & UI Kit exported from [Google Stitch Project `15727882496787728254`](https://stitch.google.com/projects/15727882496787728254), traced to `FR01`–`FR22` |
| [`stitch/`](./stitch/) | ✅ Complete | 5 exported HTML screens (`stitch/screens/`), 5 high-resolution PNG screenshots (`stitch/screenshots/`), and reproducible prompts (`stitch/stitch-mockup-prompts.md`) |

---

## What Is in This Folder

### `navigation-map.md` ⭐ (Start Here)
The complete frontend route tree covering all 9 business modules + IAM/Profile, a 26-screen map (route, component, minimum role, backing microservice, related HU/FR), a 5-role access matrix (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`), and 4 cross-role flows (Login, Maintenance lifecycle with `WORK_COMPLETED` verification, Front-desk visitor/vehicle flow, and Board extraordinary expense approval).

### `design-system.md`
`resi-complex`'s design tokens (colors, `Inter`/`JetBrains Mono` typography, 4px spacing scale, radii, shadows), reusable base components (`Button`, `Form`, `Toast`, `Modal`, `Tooltip`, `Skeleton`, `Data Table`), and complete domain badges synced 1:1 with `06-data/models.md` (`MaintenanceRequest`, `AdministrationFee`, `Correspondence`, `VisitorPreauthorization`, `ExpenseProposal`, `Announcement`, `Unit`).

### `wireframes.md` & `stitch/`
High-fidelity mockups and mandatory UI Kit designed in **Google Stitch** (`15727882496787728254`):
- **01 — Mandatory Component UI Kit (`01-ui-kit-componentes`)**: `Badge`, `Toast`, `Modal`, `Tooltip`, `Skeleton loader`.
- **02 — Administrator Dashboard (`/dashboard`)**: KPI cards from `reports-service`, recent requests table, `Tooltip`, `Toast`.
- **03 — Maintenance Requests List (`/requests`)**: Filters, badges, pagination, and `Skeleton loader` progressive state.
- **04 — Extraordinary Expense Approval (`/expenses/EXP-12`)**: Board voting view (`BOARD`) with destructive confirmation `Modal`.
- **05 — Front Desk: Visitor Registration (`/front-desk/visits`)**: Simplified `SECURITY_GUARD` sidebar, active visits table, keyboard shortcut `Tooltip` (`Alt+N`), and exit confirmation `Toast`.

---

## Correlations with Other Sections

| This section is fed by... | And feeds into... |
|---------------------------|-------------------|
| `04-requirements/user-stories.md` (22 HUs, `FR01`–`FR22`) | Every screen, form, and cross-role user flow in `navigation-map.md` |
| `04-requirements/non-functional.md` (`NFR-009` WCAG 2.1 AA, `360px` mobile) | Contrast, keyboard navigation, and responsive layout rules in `design-system.md` |
| `06-data/models.md` & `02-domain/entities-and-rules.md` | Exact status/priority enum badges and form validation invariants |
| `00-governance/security-policy.md` & `07-api/authentication.md` | 5-role access matrix and `:own` / `:assigned` UI visibility rules |
| `09-microservices/service-catalog.md` & `07-api/contracts/openapi/` | Exact backing microservice and API endpoint per screen |

---

## Questions This Section Answers

- **How many screens cover the MVP?** → **26 named screens** across all 9 Bounded Contexts + IAM/Profile (`navigation-map.md`), with 5 anchor high-fidelity Stitch screens & UI Kit (`wireframes.md`).
- **How does each role navigate?** → See the 5-role Access Matrix in `navigation-map.md` (`ADMINISTRATOR`, `BOARD`, `PERSON`, `MAINTENANCE_STAFF`, `SECURITY_GUARD`), mirroring `00-governance/security-policy.md` 1:1.
- **What visual language and accessibility standards apply?** → Institutional blue/green semantic palette (`design-system.md`), WCAG 2.1 AA contrast (`4.5:1`), keyboard accessibility, and mobile usability down to `360px` (`NFR-009`).
