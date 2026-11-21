# Prompts para Google Stitch — resi-complex (UX/UI)

> **Proyecto en Google Stitch:** https://stitch.google.com/projects/15727882496787728254
>
> **Propósito:** Especificación y prompts listos para copiar y pegar en **Google Stitch** para generar mockups profesionales y minimalistas de **resi-complex**, alineados con `12-ux-ui/design-system.md` y `12-ux-ui/navigation-map.md`.
>
> **Componentes obligatorios incluidos:** `Toast`, `Modal`, `Tooltip`, `Skeleton loader` y `Badge` (Status, Priority, Unit Type).

---

## 1. Base visual global (Design System)

Incluye este bloque de contexto en Stitch para mantener consistencia visual entre todas las pantallas generadas:

```text
Design a clean, minimalist, institutional SaaS web application UI for "resi-complex" (a residential & commercial complex management platform).

Visual Style:
- Minimalist, modern, high-trust institutional aesthetic with generous whitespace, subtle 1px borders (#E2E6EC), crisp typography, and zero visual clutter.
- Light mode primary interface: Page background #F7F8FA, elevated cards #FFFFFF with subtle shadow (0 1px 2px rgba(0,0,0,0.05)) and 8px border-radius.
- Color Palette:
  - Primary Institutional Blue: #1A5FB4 (hover/dark #0B2E5C, soft tint #E8F0FE)
  - Secondary Muted Green: #2E7D5B
  - Text Primary: #1C1E21 | Text Secondary: #5F6368 | Disabled: #9AA0A6
  - Semantic: Success #2E7D32, Warning #B98900, Error #C62828, Info #1A5FB4
- Typography: Inter for UI text and tables, JetBrains Mono for IDs, amounts, dates, and license plates.
- Spacing: 4px grid system. WCAG AA contrast compliance.
```

---

## 2. Prompt — UI Kit de Componentes Obligatorios

Usa este prompt en Stitch para generar una lámina dedicada a los 5 componentes clave solicitados:

```text
Create a minimalist, professional UI Component Sheet for "resi-complex" on a #F7F8FA canvas with crisp white #FFFFFF showcase cards, using Inter and JetBrains Mono fonts and primary blue #1A5FB4. Display these 5 components clearly labeled:

1. BADGES (pill-shaped, border-radius 9999px, 12px medium text with subtle tinted backgrounds and small status dot):
   - Status Badges: PENDING (amber #B98900), ASSIGNED (blue #1A5FB4), IN_PROGRESS (blue #1A5FB4), UNDER_REVIEW (blue #1A5FB4), RESOLVED (green #2E7D32), PAID (green #2E7D32), APPROVED (green #2E7D32), OVERDUE (red #C62828), REJECTED (red #C62828).
   - Priority Badges: LOW (neutral gray), MEDIUM (blue), HIGH (amber), URGENT (bold red #C62828 with subtle red border).
   - Unit Type Badges: RESIDENTIAL (muted green #2E7D5B tint), COMMERCIAL (blue #1A5FB4 tint).

2. TOAST NOTIFICATIONS (floating bottom-right cards, 8px radius, subtle shadow, 4s auto-dismiss):
   - Success toast: dark #1C1E21 surface with green check icon — "Maintenance request created."
   - Error toast: white surface with red left accent and "Retry" ghost button — "Could not save changes."
   - Network warning toast: "No connection. Retrying..."

3. MODAL (centered dialog, 16px radius, shadow-lg, dimmed overlay rgba(28,30,33,0.5)):
   - Destructive confirmation modal: Title "Reject expense proposal?", clear body copy explaining the action is irreversible, Secondary "Cancel" button, and Danger red (#C62828) "Reject proposal" button.

4. TOOLTIP (compact dark #1C1E21 popover, 4px radius, 12px white text, subtle pointer arrow):
   - Shown hovering over an info icon next to a table header or disabled action: "Only Board members can approve or reject proposals."

5. SKELETON LOADER (minimalist loading state using soft gray #E8ECEF rounded bars):
   - A KPI card skeleton and a 3-row data table skeleton showing placeholder bars for ID, title, badge pill, and action button.
```

---

## 3. Prompts por Pantalla (Flujos principales)

### Pantalla A — Dashboard del Administrador (`/dashboard`)
*Incluye: `Badge`, `Tooltip` y `Toast`.*

```text
Design a minimalist desktop web dashboard for "resi-complex" (/dashboard) for the ADMINISTRATOR role.
- Left sidebar (240px, white #FFFFFF, subtle right border): brand logo "resi-complex" at top, clean navigation links (Dashboard active in #E8F0FE with #1A5FB4 text, Units, People, Requests, Fees, Announcements, Expenses, Reports, Profile).
- Main content on #F7F8FA background:
  - Header: "Dashboard" with subtle role pill "ADMINISTRATOR" and a primary blue button "+ Publish announcement".
  - 4 minimalist KPI cards in a row (Open Requests, Pending Fees, Active Visits, Proposals Under Review) with clean numbers and a small info icon on one card displaying an active dark Tooltip ("Updated in real time from reports-service").
  - Clean Data Table below: "Recent Maintenance Requests" with columns ID (JetBrains Mono), Unit, Subject, Priority (Priority Badges including bold red URGENT), Status (Status Badges: PENDING, IN_PROGRESS, RESOLVED).
  - Bottom-right corner: a sleek dark Toast notification ("Announcement published successfully").
```

### Pantalla B — Lista de Solicitudes + Estado de Carga (`/requests`)
*Incluye: `Badge` y `Skeleton loader`.*

```text
Design a split-view or two-state minimalist desktop UI for "resi-complex" Maintenance Requests (/requests):
- Top bar with search input, Status filter dropdown, Priority filter dropdown, and a single Primary blue button "+ New request".
- Main Data Table card (#FFFFFF, 8px radius):
  - Top 3 rows populated with clean data, monospace IDs (REQ-041, REQ-042, REQ-043), Unit Type badges (RESIDENTIAL / COMMERCIAL), Priority badges (URGENT bold red, HIGH amber, MEDIUM blue), and Status badges (IN_PROGRESS, PENDING, ASSIGNED).
  - Bottom 2 rows displaying a clean Skeleton Loader state (soft gray animated-style placeholder bars and pill shapes) representing paginated rows loading.
- Minimalist pagination footer ("Showing 1–20 of 48").
```

### Pantalla C — Aprobación de Gasto Extraordinario (`/expenses/:proposalId`)
*Incluye: `Modal` y `Badge`.*

```text
Design a minimalist desktop screen for "resi-complex" Extraordinary Expense Proposal Detail (/expenses/EXP-12) viewed by a BOARD member.
- Background screen (dimmed by rgba(28,30,33,0.5) overlay): clean detail card showing Subject "Roof Waterproofing — Tower B", Amount "$4,200.00" in JetBrains Mono, Status Badge "UNDER_REVIEW" (blue), justification text, and Approve (Primary blue) / Reject (Danger red) buttons.
- Foreground centered Confirmation Modal (#FFFFFF, 16px radius, crisp shadow):
  - Title: "Reject expense proposal?"
  - Description: "This action is irreversible. The Administrator will be notified immediately."
  - Optional minimalist textarea for rejection note.
  - Footer buttons: Secondary "Cancel" and Danger red (#C62828) "Confirm rejection".
```

### Pantalla D — Portería: Registro de Visitas (`/front-desk/visits`)
*Incluye: `Badge`, `Tooltip` y `Toast`.*

```text
Design a fast, minimalist, keyboard-friendly desktop screen for "resi-complex" Front Desk Visits (/front-desk/visits) for the SECURITY_GUARD role.
- Simplified left sidebar showing only Dashboard, Front Desk (Visits, Correspondence), Announcements, and Profile.
- Header: "Active Visits" with a single Primary blue button "+ Log visit".
- Clean Data Table with columns: Visitor Name, Destination Unit, Visitor Type (Personal / Commercial), Vehicle Plate (JetBrains Mono, e.g. ABC-492), Entry Time, and "Log exit" secondary button.
- Include a dark Tooltip hovering over the "+ Log visit" button showing keyboard shortcut "Shortcut: Alt + N".
- Include a subtle Toast at bottom-right: "Exit logged for Diego Paredes."
```

### Pantalla E — Vista Móvil Responsive para Residente (NFR03)
*Incluye: `Badge` y `Skeleton loader`.*

```text
Design a clean, minimalist mobile web viewport (390x844) for "resi-complex" viewed by a PERSON (Resident) role on /fees and /requests.
- Compact top header with hamburger menu icon, "resi-complex" wordmark, and user avatar.
- Segmented tab bar: "My Requests" (active) and "My Fees".
- Vertical stack of minimalist white cards (#FFFFFF, 8px radius, 12px gap) on #F7F8FA background:
  - Card 1: "Kitchen pipe leak", URGENT red bold badge, IN_PROGRESS blue badge, date in secondary gray.
  - Card 2: "Main lock adjustment", HIGH amber badge, ASSIGNED blue badge.
  - Card 3: Clean Skeleton Loader card at the bottom indicating more items loading.
- Floating bottom-right primary blue "+ New request" button.
```

---

## 4. Checklist de verificación antes de aprobar imágenes de Stitch

- [x] **Paleta:** Predomina el blanco `#FFFFFF` y gris claro `#F7F8FA` con acentos en azul institucional `#1A5FB4`.
- [x] **Tipografía:** `Inter` para interfaz y `JetBrains Mono` para IDs, montos y placas.
- [x] **5 componentes visibles:** `Toast`, `Modal`, `Tooltip`, `Skeleton loader` y `Badge` (Status, Priority, Unit Type).
- [x] **Minimalismo:** Solo 1 botón `Primary` por vista, sin elementos decorativos innecesarios y con contraste WCAG AA.
