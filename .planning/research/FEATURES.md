# Feature Research

**Domain:** Behavior-preserving refactor of a single-file React CDN equipment-log app
**Researched:** 2026-09-11
**Confidence:** HIGH

## Context

This is NOT a new-feature milestone. The app is working and in daily use. The goal is to decompose a monolithic single `App` component (~532 lines, ~14 `useState` hooks, inline JSX) into smaller, maintainable components/modules — without changing any user-visible behavior. The "features" below define what must be preserved, what quality attributes the refactor should deliver, and what must NOT change.

---

## Table Stakes (Must Preserve or Users/Managers Lose Trust)

These are non-negotiable behavior-preservation guarantees. If ANY of these regress, the refactor has failed.

| Feature | What Must Be Preserved | Complexity | Notes |
|---------|----------------------|------------|-------|
| **Borrow request form submission** | Same fields, same conditional visibility (email hidden for Vehicles, plat number only for Vehicles, custom equipment field for Other), same Firebase write to `borrowing_requests` | LOW | Form data shape, validation (`required`), and Firestore document structure must be identical |
| **Equipment type dropdown** | Same three options: "DJI Osmo Action 6", "Vehicles" (internal value `Car`), "Other" with conditional custom input field | LOW | Internal value `Car` vs display label "Vehicles" mapping must not change — existing Firestore data uses `Car` |
| **Vehicle-specific fields** | Plat Number dropdown (WRD 5900), Borrow Time, Fuel Level (1 bar–Full) shown only when `cameraModel === 'Car'` | LOW | These are conditionally rendered based on `formData.cameraModel` |
| **Non-vehicle equipment condition** | Equipment Condition dropdown (Excellent/Good/Fair/Damaged) shown when NOT `Car` | LOW | Mutually exclusive with vehicle fields |
| **Manager authentication gate** | Passcode modal on Manager tab click, Enter key submits, passcode validated against the HTML constant `DEFAULT_PASSCODE = "1234"` (Block 2); no Firestore read, no rotation field | MEDIUM | Passcode is the `DEFAULT_PASSCODE` constant (rotate by editing it); must preserve the flow: modal → validate against constant → set `isManagerAuthenticated` → switch to manager view |
| **Request list with status filtering** | Status filter dropdown (All/Pending/Approved/Returned/Rejected), equipment filter (All/DJI Camera/Vehicles/Other), search by name, date range filter, CLEAR button, filtered count on EXPORT button | LOW | Filter logic lives in `filteredRequests` useMemo — must produce identical results |
| **Status counts dashboard** | Colored pill badges: Pending (amber), Active/Approved (emerald), Overdue (red), Returned (blue), Rejected (red) — counts computed from `statusCounts` useMemo with overdue logic (returnDate < today) | LOW | Overdue detection: `status === 'Approved'` AND `returnDate < today` |
| **Request cards** | Same layout: status label, borrower name, equipment + plat + phone, duration, return date + days info, fuel level (vehicles), passed indicator, return condition (returned), history count badge, left-border highlight for unseen/overdue | MEDIUM | Card rendering is ~40 lines of conditional JSX — must produce identical DOM |
| **Approve / Reject actions** | Approver name dropdown (Shafiq), APPROVE writes `approvedBy` + `approvedAt` to Firestore, REJECT writes status change, both show notification | LOW | Shared between card buttons and detail modal — same Firestore update shape |
| **Item Passed tracking** | "ITEM PASSED" button on Approved cards (only if not already passed), writes `passedAt` to Firestore, shows "✓ Passed on {date}" in card and detail modal | LOW | Only visible when `status === 'Approved'` AND `!passedAt` |
| **Mark as Returned** | Return condition modal (fuel level for vehicles, condition for equipment), CONFIRM RETURN writes `returnedAt` + `returnCondition` to Firestore, resets modal state | MEDIUM | Modal has two different field sets based on equipment type — must preserve both paths |
| **Detail modal** | Click card → full-screen modal with all request fields, conditional vehicle/equipment fields, approval info, return info, action buttons mirroring card buttons, dismiss on backdrop click or X | MEDIUM | Duplicate action logic between cards and detail modal — refactor must preserve both entry points |
| **UNDO REJECT** | "UNDO REJECT" button on Rejected cards and detail modal, resets status to "Pending Approval" | LOW | Simple Firestore update |
| **Copy to clipboard** | 3-tier fallback: Clipboard API → execCommand fallback → prompt fallback. Copies formatted text including approver name/date for approved, return info for returned, rejection notice | MEDIUM | Non-secure context (HTTP LAN) means Clipboard API likely fails — the fallback chain is critical |
| **CSV export** | Exports filtered requests with columns: Date Requested, Requested By, Email, Phone, Equipment, Plat Number, Purpose, Return Date, Status, Condition/Fuel, Borrow Time, Approved By. Filename: `kwm-requests-{date}.csv` | LOW | Column order and content must match exactly — users may import into spreadsheets |
| **Dark mode toggle** | Persists to `localStorage('kwm_dark_mode')`, toggles `dark` class on `<html>`, Sun/Moon icon swap, affects all glassmorphic styling | LOW | Must toggle same localStorage key and DOM class |
| **Notification toasts** | Bottom-fixed toast (mobile: full-width bottom, desktop: right-aligned), auto-dismiss after 4 seconds, "Processing..." state on submit button | LOW | Uses `scale-up-center` animation |
| **Unseen request badge** | Manager tab shows red badge with unseen count (max "99+"), persisted to `localStorage('kwm_seen_requests')` as Set of IDs, marks all seen when switching to manager view | LOW | Badge logic uses `seenRequestIds` Set — must preserve localStorage key and marking behavior |
| **Real-time Firestore sync** | `onSnapshot` listener on `borrowing_requests` collection, sorted by `createdAt` descending, triggers Lucide icon re-render after 100ms timeout | LOW | The 100ms `setTimeout` for `lucide.createIcons()` is a known quirk — preserve it to avoid icon rendering issues |
| **Firebase anonymous auth** | Auto sign-in anonymously on mount if no user, `onAuthStateChanged` listener | LOW | Must persist — Firestore rules likely require auth |
| **Responsive layout** | Mobile-first: form takes full width, nav pills flex on mobile, cards stack vertically, modals slide up from bottom on mobile / center on desktop | MEDIUM | Touch targets, safe area padding, `user-scalable=no` viewport — all must survive |
| **Enter key on passcode** | Pressing Enter in passcode input triggers the Unlock button click | LOW | Currently done via `onKeyDown` + `document.querySelector('[data-passcode-unlock]').click()` |

---

## Differentiators (Quality Attributes the Refactor Should Deliver)

These are the reasons to do the refactor. They don't change user-visible behavior but determine whether the refactor was worthwhile.

| Attribute | Value Proposition | Complexity | Notes |
|-----------|-------------------|------------|-------|
| **Component decomposition** | Break the monolithic `App` into logical sub-components (e.g., `RequestForm`, `ManagerView`, `RequestCard`, `DetailModal`, `PasscodeModal`, `ReturnConditionModal`, `FilterBar`, `StatusPills`, `Notification`). Each component owns its own concern. | MEDIUM | Target: no component exceeds ~100 lines. Each should have clear props interface |
| **State locality** | Move state closer to where it's used. Form state → `RequestForm`. Filter state → `FilterBar`. Modal state → respective modal components. Only shared state (requests, auth, dark mode) stays in `App`. | MEDIUM | Reduces prop drilling and makes state flow traceable |
| **Extracted utility functions** | Move `isOverdue`, `getDaysInfo`, `getRequestDuration`, `getEquipmentLabel`, `getEquipmentIcon`, `copyDetails`, `exportCSV` out of the component into a shared utils module (as `<script>` blocks in the same HTML file — no build step) | LOW | These are pure functions or side-effect-only functions with no React dependency |
| **Extracted constants** | `STATUS_COLORS`, `INPUT_CLS`, `SELECT_CLS`, `DEFAULT_PASSCODE`, equipment option lists, approver options, fuel levels, condition options — all into a constants section at the top | LOW | Makes future expansion (e.g., adding equipment types) a one-line change |
| **Reduced useState count in App** | The root `App` component should hold only cross-cutting state: `user`, `isManagerAuthenticated`, `isDarkMode`, `requests`, and top-level `view`. Everything else moves to child components. | HIGH | This is the primary structural goal — reduces cognitive load when modifying any feature |
| **Self-documenting data flow** | With smaller components and localized state, it becomes obvious which component reads/writes which Firestore fields. No more hunting through 530 lines to trace a data path. | LOW | Natural consequence of good decomposition |
| **Lower regression risk per change** | Editing `RequestForm` cannot accidentally break `DetailModal` rendering. Each component is a smaller blast radius. | MEDIUM | Key business value — the app is in daily use, regressions disrupt operations |
| **Easier onboarding** | New developer (or AI agent) can understand the app by reading component files in order, not by parsing one massive function | LOW | Critical if the team grows or if future milestones add features |
| **Preserved single-file deployment** | Despite decomposition into components, the app MUST remain a single `index.html` file served via `python -m http.server` — no build step, no npm, no bundler | MEDIUM | Components are defined as `<script type="text/babel">` functions in the same file, or split into multiple `<script>` blocks within the same HTML. CDN React + Babel in-browser compilation is the deployment model |
| **Identical Firestore data shape** | Every Firestore document written by the refactored app must be structurally identical to what the current app writes — field names, types, nesting. No schema migration needed. | LOW | This is a constraint, not a feature, but it's a critical quality attribute |

---

## Anti-Features (Things to Deliberately NOT Do)

These are temptations to resist during the refactor. Doing any of them increases risk and scope without delivering the milestone goal.

| Anti-Feature | Why It Seems Tempting | Why It's Problematic | What to Do Instead |
|-------------|----------------------|---------------------|-------------------|
| **Add new features** | "While we're refactoring, let's add X" | Scope creep. Every new feature is an untested behavior change mixed with structural changes. Bisecting regressions becomes impossible. | Finish the refactor first. New features go in the NEXT milestone with their own plan. |
| **Change UI styling or layout** | "The button looks slightly off, let me fix it" | Visual changes are behavior changes — users notice. Even subtle CSS tweaks can break mobile touch targets or safe-area padding. | Copy-paste exact same CSS classes. Verify visually before and after. |
| **Migrate Firebase data** | "Let's clean up the old documents while we're at it" | Data migration on a live production system with daily users = potential data loss. Also changes the Firestore contract. | Do not touch existing Firestore documents. The refactor is code-only. |
| **Switch to modular Firebase SDK** | "The compat API is deprecated, let's use v9+" | Changes every Firebase call in the app. Mixing SDK migration with component decomposition doubles the blast radius. | Keep compat v11.6.1 namespaced API. SDK migration is a separate future milestone. |
| **Introduce a build step** | "We need proper modules / TypeScript / imports" | Violates the single-file CDN constraint. Breaks deployment (`python -m http.server`). Requires toolchain the team doesn't have. | Use `<script>` blocks and Babel in-browser compilation. Functions are global-scope within the file. |
| **Extract to separate .js files** | "Let's use ES modules" | Non-secure HTTP context on LAN may not serve correct MIME types for JS modules. CORS issues with `file://` protocol. Breaks the single-file deployment model. | Keep everything in `index.html`. Use `<script type="text/babel">` for JSX components, `<script>` for pure JS utilities. |
| **Refactor Firebase paths or collection structure** | "The path `artifacts/camera-borrow-wiramas/public/data/...` is verbose" | Changes the Firestore contract. Existing data, security rules, and any external integrations depend on these paths. | Use the same paths as string constants. Do not change them. |
| **Add state management library** | "Redux / Zustand / Jotai would manage this better" | Adds a CDN dependency, increases bundle complexity, violates the minimal-dependency philosophy of this internal tool. | Prop drilling through component tree is fine at this scale (~10 components). Context API if needed for truly global state (dark mode, auth). |
| **Change the passcode authentication model** | "Let's add proper auth / roles" | Out of scope per PROJECT.md. Passcode gate is the decided approach. | Keep the passcode modal as-is. |
| **Optimize performance** | "Let's add memoization everywhere / virtualize the list" | The list is small (internal team, maybe dozens of requests). Premature optimization adds complexity without measurable benefit. | Only memoize where the current code already does (useMemo for `filteredRequests`, `statusCounts`, `borrowerCounts`). Don't add new memoization. |
| **Refactor the Lucide icon approach** | "Let's use React Lucide components instead of `data-lucide` attributes" | Changes the icon rendering model. The 100ms setTimeout hack for `lucide.createIcons()` exists for a reason — switching to React Lucide components requires a different CDN package and different rendering approach. | Keep the existing `Icon` component wrapper and `lucide.createIcons()` pattern. |
| **Add error boundaries** | "We should add React error boundaries" | Useful in general, but this is a refactor milestone, not a reliability milestone. Adding error boundaries changes the failure mode of the app (shows fallback UI instead of crashing). | Defer to a future milestone if needed. |
| **Rename internal values** | "Let's rename `'Car'` to `'Vehicle'` for consistency" | Existing Firestore documents use `'Car'` as the `cameraModel` value. Renaming breaks filter logic, equipment mapping, and conditional rendering for all existing data. | Keep `'Car'` as internal value, `'Vehicles'` as display label. This is a documented key decision. |

---

## Feature Dependencies

```
Component Decomposition
    ├──requires──> State Locality (state must move with components)
    │                  └──requires──> Preserved Firestore Data Shape (state must match existing data)
    ├──requires──> Extracted Utilities (pure functions out of component)
    │                  └──requires──> Preserved Function Signatures (same inputs/outputs)
    ├──requires──> Extracted Constants (shared config at top)
    └──conflicts──> New Features (adding features during decomposition = untestable)

Preserved Single-File Deployment
    ├──conflicts──> ES Modules / Separate .js Files
    └──conflicts──> Build Step / Bundler

Lower Regression Risk
    └──depends-on──> Component Decomposition (smaller blast radius per component)
```

### Dependency Notes

- **Component Decomposition requires State Locality:** You can't have clean components if all state lives in `App` and is passed down as props. State must move to the component that owns it.
- **State Locality requires Preserved Firestore Data Shape:** When state moves to a child component, the Firestore read/write calls move too. The document structure (field names, types) must stay identical.
- **Extracted Utilities requires Preserved Function Signatures:** Moving `isOverdue(req)` to a utility module means it must accept the same request object shape and return the same boolean. No refactoring the data model while extracting.
- **Component Decomposition conflicts with New Features:** Every new feature changes the behavior being verified. Decomposition + new features = untestable changeset. Do one, verify, then the other.
- **Preserved Single-File Deployment conflicts with ES Modules/Build Step:** The deployment model is `python -m http.server` serving a single HTML file. No MIME type issues, no CORS, no toolchain.

---

## Refactor Scope Definition

### In Scope (What "Done" Looks Like)

- [ ] `App` component contains only: view routing, auth state, dark mode, request data, and top-level layout
- [ ] `RequestForm` component owns: form state, equipment-conditional fields, submit handler, form validation
- [ ] `ManagerView` component owns: filter bar, status pills, request card list, export button
- [ ] `FilterBar` component owns: filter state, search, status/equipment dropdowns, date range, clear
- [ ] `RequestCard` component owns: single card rendering, action buttons (approve/reject/passed/returned/copy/undo)
- [ ] `DetailModal` component owns: full request display, action buttons (duplicate of card actions), dismiss
- [ ] `PasscodeModal` component owns: passcode input, validation, new passcode field, Firestore update
- [ ] `ReturnConditionModal` component owns: condition/fuel selection, confirm return handler
- [ ] `Notification` component owns: toast display, auto-dismiss
- [ ] `StatusBar` (or `StatusPills`) component owns: count badges (pending/active/overdue/returned/rejected)
- [ ] Utility functions extracted: `isOverdue`, `getDaysInfo`, `getRequestDuration`, `getEquipmentLabel`, `getEquipmentIcon`, `copyDetails`, `exportCSV`, `markRequestsAsSeen`
- [ ] Constants extracted: `STATUS_COLORS`, `INPUT_CLS`, `SELECT_CLS`, `DEFAULT_PASSCODE`, equipment options, approver options, fuel levels, condition options
- [ ] All existing tests/behaviors pass (manual verification: submit form, approve, reject, return, copy, export, dark mode, passcode, filters, modals)
- [ ] Single `index.html` file, no build step, no new CDN dependencies
- [ ] Firestore documents written are structurally identical to current app

### NOT in Scope

- New features or UI changes
- Firebase SDK migration
- Build tooling or module system
- Performance optimization
- Error boundaries or new error handling
- Tests (unit/integration/E2E) — can be a follow-up milestone
- TypeScript or type annotations

---

## Validation Checklist (How to Verify Behavior Preservation)

| Scenario | Steps to Verify | Expected Result |
|----------|-----------------|-----------------|
| Submit borrow request | Fill form → Submit | Toast "Request Logged Successfully", form resets, new card appears in manager view |
| Submit vehicle request | Select Vehicles → fill fields → Submit | Email field hidden, plat number + borrow time + fuel level visible, document has `cameraModel: 'Car'` |
| Submit "Other" request | Select Other → enter custom name → Submit | Custom equipment field visible, document stores custom name in `cameraModel` |
| Approve request | Manager view → Select approver → APPROVE | Status changes to Approved, `approvedBy` + `approvedAt` written, card shows approver |
| Reject request | Manager view → REJECT | Status changes to Rejected, UNDO REJECT button appears |
| Undo reject | Click UNDO REJECT | Status reverts to Pending Approval |
| Item Passed | Approved card → ITEM PASSED | `passedAt` written, "✓ Passed on {date}" shown |
| Mark returned | Approved card → MARK RETURNED → select condition → CONFIRM | `returnedAt` + `returnCondition` written, card shows returned info |
| Copy details | Any card → COPY INFO | Clipboard contains formatted text, toast "Details Copied!" |
| Filter by status | Select "Pending" in filter | Only pending requests shown, export count updates |
| Filter by equipment | Select "Vehicles" in filter | Only vehicle requests shown |
| Search by name | Type in search box | Only matching names shown |
| Date range filter | Set from/to dates | Only requests in range shown |
| Clear filters | Click CLEAR | All filters reset, all requests shown |
| Export CSV | Click EXPORT | CSV downloads with correct columns and data |
| Passcode gate | Click Manager tab → enter wrong passcode | Modal stays, input clears. Enter correct → switches to manager view |
| Rotation | Edit the DEFAULT_PASSCODE constant in index.html and reload | New value is required to unlock the manager view |
| Dark mode toggle | Click Sun/Moon button | Theme toggles, persists across page reload |
| Detail modal | Click any card | Modal opens with all fields, correct conditional sections, action buttons |
| Mobile layout | Resize to phone width | Form full-width, nav flexes, cards stack, modals slide from bottom |
| Unseen badge | New request arrives while on form tab | Red badge shows on Manager tab with count |
| Mark seen | Switch to Manager tab | Badge clears, localStorage updated |

---

## Sources

- `index.html` (532 lines) — direct analysis of all user-visible behaviors
- `.planning/PROJECT.md` — requirements, constraints, key decisions, out-of-scope items
- PROJECT.md Active requirement: "Fix fragility: decompose the monolithic single `App` component (~14 `useState` hooks) into a cleaner, maintainable structure without changing behavior"

---
*Feature research for: behavior-preserving component decomposition*
*Researched: 2026-09-11*
