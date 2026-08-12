# Detailed Work Log — Rishav Anand

Internal reference. Not surfaced on the portfolio site or in the resume PDF —
those carry only a brief, capability-level summary.

**Period:** May 2025 – Aug 2026 (~15 months)
**Scale:** ~766 commits · 146 merged PRs on the core platform · work across 7 repositories

---

## At a glance

| Area | Commits | Merged PRs | Role |
|---|---|---|---|
| Core legacy platform (maps, reports, investigations) | 766 | 146 | Primary owner across feature areas |
| Modern React app (Driver View / Observer View) | 97 | 36 | Built POC → production |
| Deployment / release versioning | 123 | — | Release train owner |
| Python workers & Flask APIs | 42 | 13 | Feature + pipeline work |
| Analyzer simulation mode | 10 | 3 | Local-dev enablement |
| ZMQ data-ingestion simulator | 4 | — | QA test data |
| Handheld localization pipeline | 1 | — | Contributor |

---

## 1. Core legacy platform (primary focus)

### A. Handheld data on maps & reports (~79 commits)
Largest single feature area — end-to-end ownership from mock layer through production.

- Built the handheld data map layer: toggle, gauge component, concentration filters
  (methane / ethane), tooltips, legend integration.
- Connected the platform to the Flask service layer for handheld data — proxy URL, CSRF,
  polling with HTTP 200/202 semantics, retry and timeout handling.
- Performance work for large reports (10k+ points): viewport caching, cache limit raised
  100k → 200k, eliminated rebuilds on zoom/pan, reduced legend repaint overhead.
- Handheld indications layer — per-disposition filters, confirmed-sources UI, triangle
  markers, popout popups.
- Handheld ZIP export from the report modal — correct naming, no-data messaging,
  dual-licence gating.
- Report-area handheld data licence, second-pass artifacts, on-demand zip generation.
- UAT fixes: toggle stuck on zoom, spinner visibility, canvas flicker, bounding-box
  expansion, upstream data gaps.

### B. Investigation workflow (~58 commits)
- **Do Not Investigate (DNI)** status — full feature: DB migration, licence gating, legend
  filter, assignment logic, mobile-view hiding, button visibility rules.
- Investigation page overhaul — server-side filtering (duration, asset length, material
  type), pagination fixes, back-button state retention.
- Investigation templates — "No Gas" template rolled out to all customers, plus migration
  scripts.
- Investigation assignment — autocomplete search box, design-aligned modal styles, timezone
  display, assignee clearing on DNI.
- Confirmed sources — new API endpoint, shape anchoring, GPS snap, hollow-triangle edge
  cases, keyboard navigation in the investigator dropdown.
- Box-number visibility, z-index layering, scroll behaviour fixes.

### C. Map layers (~45 commits)
- Enhanced popups across three layer types with asset metadata, emission rate, duration.
- Filterable legend items tied to assignment state.
- Breadcrumb colours on dark mode, leak-alert coordinates.
- Map fluctuation fixes: collinear tracks, degenerate-geometry centroid crash, NaN label
  flicker.
- Layer z-ordering rules: confirmed sources above indication layers, breadcrumbs above
  polygons.
- Selectable gaps on hidden features, DNI gap visibility rules.

### D. High-flow peak reporting (~20 commits)
- High-flow peaks table in compliance & emissions PDFs — optional and licence-gated.
- High-flow indication banner, post-survey audio, bubble-scale fixes.
- Peak name in historical popup, FWXM peak-validity metric, disposition filter for
  high-flow alerts.
- Customer-report licence gating for high-flow fields (live vs historical peaks).
- Deduplication radius — made configurable in customer parameters, with licence gating for
  the radius label.

### E. Emissions & compliance reporting (~23 commits)
- Emissions meta CSV zip-download UI plus API visibility.
- Emission report zip-generator fixes, risk-job failure fixes.
- Report-viewer icon for both report types, on-demand zip generation, empty-zip S3 handling.
- Dynamic scroll on compliance/emission report tables.
- Copy-compliance-report crash fix, page-number and search fixes.

### F. Reports, PDFs & capture pipeline (~49 commits)
- Capture pipeline asset-metadata table plus migration.
- Median ringdown rate added to the survey QA check.
- High-flow peak table in the capture PDF (both platform and worker side).
- Report parameter fixes (FWXM slider minimum at 0).
- 3-minute timeout for large report downloads.

### G. Licensing & feature gating (~27 commits)
Handheld data, handheld export, DNI, high-flow, two generations of artefact licences,
customer-report licence vs logged-in-user licence, deduplication-radius licence label.

### H. Security & platform upgrades
- DataTables upgrade 1.10.16 → 1.11.3 (JS vulnerability remediation).
- SSO hardening — IDP alias sanitization and domain validation.
- Disabled minification for the DataTables bundle for minifier compatibility.
- CSRF token handling for handheld export.

### I. User management & admin
- Mobile-app roles — Keycloak integration, role-creation UI, manage-user table column.
- Manage Labels crash fix; German/Spanish datetime formats for label creation.
- 2FA enablement from the Preferences page.
- Manage Customer page timezone display, new customer-parameter fields (UI + backend).
- Support role hidden from mobile-app roles.
- Role and associate-to-location fixes for customer login users.

### J. Localization & timezone
Australian Western timezone support; German/Spanish label-creation datetime fixes; timezone
abbreviation in API responses; shared datetime utility for reuse across investigation tables.

### K. Table UX — filter / sort / pagination (~50 commits)
Server-side filtering across investigation, compliance and emission report tables;
DataTable console-error fixes; ES5 refactoring for minification; column-header stability on
scroll; dynamic scroll; HH:MM:SS duration format; asset-length decimal precision.

### L. Release management (~29 commits)
Version bumps across four minor releases; build-version variables; env-var naming fixes.

---

## 2. Modern React app — Driver View & Observer View (97 commits, 36 PRs)

### Driver View (POC → production)
- Phase M1 base setup through the full live-survey UI.
- **Map:** smooth field-of-view rendering, peak bubbles, multi-polygon FOV, grey trail on
  pause, map-style persistence, position follow.
- **Analyzer integration:** SignalR hub, health gating, survey state machine (analyzer-mode
  mapping), pause/resume/shutdown, serial-number API check on connect.
- **Capture mode** — closed the remaining acceptance-criteria gaps.
- **Field notes** — author/time metadata, breadcrumb enhancements.
- **Alerts panel** — high-flow indication audio, indication colours matched to the legacy
  platform.
- **Session management** — 72-hour session history, disposition filter at session start,
  session timers, deduplicated history.
- Survey-scoped URLs and per-survey map isolation.
- Error handling refactored to toast notifications.
- Sustained QA-fix cycles on pause trail, timers and field notes.

### Observer View
Grey trail on pause and in history view; observer permissions fix; telemetry, timing and
map-layer regression fixes.

### Driver assignment / workforce
- RTK Query endpoints aligned to the documented API contract.
- Assignment Management modal, feature KPI card.
- **i18n across 13 languages** for the driver-assignment locale files.
- Response-envelope wrapping for the workforce check.

### Dev tooling
Pre-commit hooks; typecheck/lint cleanup across the codebase; Docker env fixes;
login/connect flow fixes including an intermittent login failure.

---

## 3. Python backend services (42 commits, 13 PRs)

- Emissions report meta CSV worker — full pipeline from job creation to schema-aligned
  output (ranking group, rate format).
- High-flow peaks table in capture PDF reports, with localized title.
- Name columns for two entity types, sorting fixes, PDF column alignment.
- Report-meta test coverage, PyBuilder unittest discovery, Cython build fixes.
- Survey QA worker fixes (median ringdown rate).
- Asset-metadata endpoint return-type fixes.
- Highlighting-logic fix for a specific highlight type.

---

## 4. Analyzer simulation (10 commits, 3 PRs)

- Sim mode — simulated data instead of recorded-DB replay, for local Driver View
  development.
- Field-note metadata (author, epoch time) in the update path.
- Pause-breadcrumb publish fixes for the relevant analyzer mode.
- Got the analyzer build compiling and running locally.

---

## 5. Deployment & DevOps (123 commits)

Managed the deploy-versions manifest across all microservices; ran the release train across
four minor versions; capture-worker deployment fix.

---

## 6. QA / test infrastructure

- **ZMQ ingestion simulator** — sensor scenario matrix, JSON stitch generators, adaptive
  playback flags, Driver View QA test guide.
- **Localization pipeline** — handheld localization and disposition processing.

---

## Condensed experience block (source for resume / site)

**Software Engineer — Picarro (Apr 2025 – Present)**

- Owned end-to-end delivery of handheld sensor-data visualization on maps and reports —
  service-layer integration, performance work for 10k+ point datasets, export packaging,
  indication layers and licence gating.
- Built a driver-facing and observer-facing real-time survey application from POC to
  production in React/TypeScript — instrument integration over SignalR, capture mode, field
  notes, session history and 13-language i18n.
- Delivered investigation-workflow enhancements — new disposition status, server-side
  filtering and pagination, confirmed-sources API, templates and assignment UI.
- Shipped high-flow peak reporting across PDF report generators and Python workers with
  per-customer licence gating.
- Built an emissions-report metadata pipeline (CSV generation, zip download UI, schema
  alignment).
- Improved platform security — dependency vulnerability remediation, SSO IDP hardening,
  CSRF handling.
- Contributed to user management — mobile-app roles via Keycloak, 2FA preferences,
  localization of label management.
- Managed release deployments across 7 microservices (120+ version updates, four minor
  releases).
- 146 merged PRs on the core platform; cross-stack work spanning C#/.NET MVC,
  JavaScript/OpenLayers, React/TypeScript and Python/Flask.

**Stack:** ASP.NET MVC 5, EF6, JavaScript, OpenLayers, Kendo UI, React, TypeScript,
RTK Query, SignalR, Python, Flask, Celery, C#/.NET, Keycloak, YAML/CI
