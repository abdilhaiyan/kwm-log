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

### Active

<!-- Current scope. Building toward these. -->

- [ ] Fix fragility: decompose the monolithic single `App` component (~14 `useState` hooks) into a cleaner, maintainable structure without changing behavior

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- Email notifications (EmailJS) — decided against; in-app status display is sufficient for this internal tool
- User accounts / per-user login — passcode gate is enough for an internal team tool
- Multi-language support — internal Malaysian team, single language
- Native mobile app — web app is mobile-responsive and sufficient

## Context

- **Current state:** The app is fully functional and in daily use at KWM Logistics. It was built incrementally in a single `index.html` file (CDN React 18, Tailwind with `darkMode: 'class'`, Babel, Lucide icons, Firebase compat v11.6.1 namespaced API). Phase 2 complete (2026-09-14): the manager passcode is now a config-driven constant — the Firestore passcode read/write path was removed entirely, rotation is a one-line edit, and the planning docs (PROJECT/FEATURES/ARCHITECTURE) were refreshed to match.
- **Firebase:** Project `kwm-logistics-camera`. Requests stored at `artifacts/camera-borrow-wiramas/public/data/borrowing_requests`. The manager passcode is the `DEFAULT_PASSCODE` constant in index.html Block 2 — rotating it means editing that constant and reloading. (The old `app_config` doc in Firestore still exists as harmless dead data the app never reads — D-02: leave it untouched, no cleanup.)
- **Deployment:** Served locally via `python -m http.server 8080`; staff access over LAN at `http://10.0.6.12:8080` (non-secure context — hence the clipboard fallback).
- **Known issue:** The app "works but feels fragile" — everything lives in one giant React component with ~14 `useState` hooks and inline JSX. Adding features is getting riskier. This is the main Active requirement.
- **Data conventions:** Internal value `'Car'` kept in code/data; display label is "Vehicles". Plat numbers and approver names are hardcoded dropdowns (expandable later).
- **Git:** Repo `https://github.com/abdilhaiyan/kwm-log`, branch `main`. The 8 most recent UI/UX changes are in the working tree, not yet pushed (user testing first).

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
| Hardcoded dropdowns (WRD 5900, Shafiq) | Fast to ship; expandable to Firestore later | — Pending |
| Decompose monolithic component | Fragility concern — main Active requirement | — Pending |

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
*Last updated: 2026-09-14 after Phase 2 completion*