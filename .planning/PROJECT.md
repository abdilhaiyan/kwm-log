# KWM Logistics Equipment Log

## What This Is

A mobile-first web app for KWM Logistics staff to request equipment (cameras, vehicles, other gear) and for managers to approve, reject, and track handover. Staff submit a borrow request with their details and the equipment they need; managers review it, approve or reject, record who approved, and mark when the item was passed over. Built as a single-file React app with a Firebase backend, styled with a glassmorphic dark-mode UI.

## Core Value

Accountable equipment handover — track who borrowed what, when, and whether it was returned, with an approval trail (approver name, passed status) so nothing gets lost or unaccounted.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

- ✓ Staff can submit a borrow request (name, phone, email, equipment type, date, time, purpose) — existing
- ✓ Equipment type dropdown (DJI drone, Vehicles, Other) with custom "Other" input — existing
- ✓ Vehicle-specific fields (fuel level, borrow time) shown when Vehicles selected — existing
- ✓ Email field hidden when Vehicles selected — existing
- ✓ Managers can approve or reject requests — existing
- ✓ Approver name recorded on approval (dropdown: Shafiq) — existing
- ✓ Item Passed tracking (records passedAt, shown on Approved cards + detail modal) — existing
- ✓ Request list with status (Pending/Approved/Rejected) — existing
- ✓ Detail modal showing full request info — existing
- ✓ Manager filter bar (filter by status) — existing
- ✓ CSV export of requests — existing
- ✓ Copy request details to clipboard (3-tier mobile fallback for non-secure contexts) — existing
- ✓ Dark mode toggle (persisted in localStorage) — existing
- ✓ Passcode-protected manager actions (HTML constant DEFAULT_PASSCODE, edit to rotate) — existing · Validated in Phase 2
- ✓ Plat Number dropdown for vehicles (WRD 5900) — existing
- ✓ "Requested By" label (was "Staff Name") — existing
- ✓ Enter key submits passcode modal — existing · Validated in Phase 2
- ✓ Copied details include approver name/date — existing
- ✓ Decompose the monolithic single `App` component into a maintainable 7-block structure — v1.0 (Phase 1)
- ✓ Passcode stored as configurable HTML constant, Firestore passcode logic removed — v1.0 (Phase 2)
- ✓ Plat Number and Approver as config-driven selects from editable constant arrays — v1.0 (Phase 3)

### Active

<!-- Current scope. Building toward these. -->

- (None — next milestone requirements to be defined via `/gsd-new-milestone`)

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Email notifications (EmailJS) — decided against; in-app status display is sufficient for this internal tool
- User accounts / per-user login — passcode gate is enough for an internal team tool
- Multi-language support — internal Malaysian team, single language
- Native mobile app — web app is mobile-responsive and sufficient
- ES modules / build step — violates single-file CDN deployment constraint
- Firebase modular SDK migration — highest behavior-change risk; separate milestone
- React 19 upgrade — requires ESM loading; separate milestone

## Context

- **Current state:** v1.0 MVP shipped 2026-09-15. The app is fully functional and in daily use at KWM Logistics. Built as a single `index.html` file (CDN React 18, Tailwind with `darkMode: 'class'`, Babel, Lucide icons, Firebase compat v11.6.1 namespaced API) organized into 7 ordered `text/babel` blocks (Constants → Config → Firebase Init → Utilities → Shared UI → App Shell → Render). Phase 2 (2026-09-14): the manager passcode is a config-driven constant — Firestore passcode read/write path removed, rotation is a one-line edit. Phase 3 (2026-09-14): Plat Number and Approver are config-driven selects from `PLAT_OPTIONS`/`APPROVER_OPTIONS` constants with "Other" free-text reveal.
- **Firebase:** Project `kwm-logistics-camera`. Requests stored at `artifacts/camera-borrow-wiramas/public/data/borrowing_requests`. The manager passcode is the `DEFAULT_PASSCODE` constant in index.html Block 2 — rotating it means editing that constant and reloading. (The old `app_config` doc in Firestore still exists as harmless dead data the app never reads — D-02: leave it untouched, no cleanup.)
- **Deployment:** Served locally via `python -m http.server 8080`; staff access over LAN at `http://10.0.6.12:8080` (non-secure context — hence the clipboard fallback).
- **Git:** Repo `https://github.com/abdilhaiyan/kwm-log`, branch `main`. v1.0 pushed (commit `cd2e0ed`). GitHub Pages: `https://abdilhaiyan.github.io/kwm-log/`.
- **Data conventions:** Internal value `'Car'` kept in code/data; display label is "Vehicles". Plat numbers and approver names are config-driven constants in Block 2 (`PLAT_OPTIONS`, `APPROVER_OPTIONS`) — adding one is a one-line insert before "Other".

## Constraints

- **Tech stack**: Single-file HTML with CDN React 18 + Tailwind + Babel + Lucide + Firebase compat v11.6.1 — no build step, no npm, no bundler. Keep it that way.
- **Firebase API style**: Namespaced compat API (`firebase.auth()`, `db.collection()`, `.add()`, `.doc().update()`) — not the modular v9+ API.
- **Mobile-first**: Staff use phones over LAN HTTP (non-secure context). Clipboard, layout, and touch interactions must work there.
- **Security**: Manager actions gated by the passcode constant in HTML; rotate by editing `DEFAULT_PASSCODE`. No user accounts.
- **Compatibility**: Must keep working as a single file that can be opened/served anywhere without a build step.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single-file CDN app (no build step) | Simple deployment, no toolchain for a small internal tool | ✓ Good |
| Firebase compat namespaced API | Matches existing code, stable | ✓ Good |
| No EmailJS / in-app status only | Internal tool, status list is enough | ✓ Good |
| Passcode as HTML constant (DEFAULT_PASSCODE) | Rotate by editing the constant; no Firestore dependency | ✓ Good (Phase 2) |
| Internal `'Car'` value, "Vehicles" label | Avoids data migration; user-facing clarity | ✓ Good |
| Config-driven dropdowns (PLAT_OPTIONS, APPROVER_OPTIONS) | One-line constant edit to add options; no JSX/logic changes | ✓ Good (Phase 3) |
| 7-block text/babel decomposition | Global scope via script order; zero import/export; single-file preserved | ✓ Good (Phase 1) |
| Prop drilling, not React Context | 19 useState below the Context threshold at this scale | ✓ Good (Phase 1) |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-09-15 after v1.0 milestone*