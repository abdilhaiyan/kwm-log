# Project Research Summary

**Project:** KWM Logistics Equipment Log
**Domain:** No-build single-file CDN React app — behavior-preserving decomposition
**Researched:** 2026-09-11
**Confidence:** HIGH

## Executive Summary

KWM Equipment Log is a working, daily-use single-file React app (`index.html`, ~532 lines) with a monolithic `App` component holding 14 `useState` hooks, 5 `useEffect` blocks, and all UI inline as JSX. The Active requirement is to decompose this monolith into maintainable structure **without changing any user-visible behavior** and **without introducing a build step**. The app is deployed via `python -m http.server` over a local LAN (`http://10.0.6.12:8080`) — no HTTPS, no npm, no bundler, no toolchain.

The recommended approach is **multi-inline-block decomposition**: split the single `<script type="text/babel">` block into 7 ordered `<script type="text/babel">` blocks (Constants → Utilities → Firebase Service → Shared UI → Modals → View Components → App Shell). This is a pure code-movement refactor — same JSX, same Babel standalone, same React 18.3.1 UMD, same Firebase compat 11.6.1, same global scope sharing. ESM modules are definitively blocked (Babel standalone transforms `import` → `require()`, and Python serves `.jsx` as `octet-stream`). The decomposition is bottom-up: extract leaf utilities first, then service layer, then UI components, then the app shell orchestrator last.

The top risks are Firebase listener leaks (the `onSnapshot` and `onAuthStateChanged` effects both have cleanup-returning subscriptions that must be preserved during extraction), behavioral drift in the borrow/return flow (the `cameraModel` field stores `'Other'` not custom text, `'Approved'` status covers both approved and passed states, clipboard has a 3-tier fallback critical for non-secure HTTP), and the Lucide icon rendering hack (`lucide.createIcons()` via 100ms `setTimeout` must survive extraction). All 17 behaviors and 12 quality attributes are catalogued; 12 anti-features explicitly prohibit scope creep during this milestone.

## Key Findings

### Recommended Stack

The stack is frozen. Every CDN script stays exactly where it is. The only structural change is splitting one inline `<script type="text/babel">` block into seven ordered blocks. Babel standalone 8.0.5 auto-compiles `text/babel` scripts via XHR (MIME-agnostic — `.jsx` files served as `octet-stream` by Python still compile), executes in document order, and defaults to the `react` preset with `runtime: 'classic'`. Classic scripts share global lexical scope, so constants → utils → views → App reference each other naturally without `import`/`export`.

**Core technologies:**
- **React 18.3.1 UMD** — UI framework; React 19 removed UMD builds entirely, so upgrading forces ESM migration (out of scope)
- **Babel standalone 8.0.5** — In-browser JSX transform AND the decomposition engine; supports external `src=` scripts via XHR
- **Tailwind CSS 3.4.17 CDN** — Styling; v4 browser runtime cannot read `darkMode: 'class'` JS config
- **Firebase compat 11.6.1** — Firestore + anonymous auth via namespaced API; modular rewrite = highest behavior-change risk
- **Lucide 1.44.0 UMD** — Icons; DOM-scanning `createIcons()` pattern must be preserved as-is
- **No new dependencies** — The entire point is the existing CDN set is sufficient

### Expected Features

This is NOT a new-feature milestone. 17 behaviors must be preserved identically; 12 quality attributes define success; 12 anti-features are explicitly prohibited.

**Must preserve (table stakes — any regression = failure):**
- Borrow request form with equipment-conditional fields (email hidden for Vehicles, plat number only for Vehicles, custom input for Other)
- Equipment type dropdown with internal `'Car'` value for Vehicles (Firestore data depends on this)
- Vehicle-specific fields (Plat Number dropdown, Borrow Time, Fuel Level) shown only when `cameraModel === 'Car'`
- Manager passcode gate with Enter key submit, Firestore-backed validation, optional passcode rotation
- Request list with status/equipment/search/date-range filtering + CLEAR button
- Status counts dashboard (Pending amber, Active/Approved emerald, Overdue red, Returned blue, Rejected red)
- Request cards with approve/reject/passed/return/undo-reject/copy-info actions
- "ITEM PASSED" tracking (writes `passedAt`, status stays `'Approved'` — do NOT change to `'Passed'`)
- Return condition modal with equipment-type-specific fields (fuel for vehicles, condition for equipment)
- Detail modal with all fields and action buttons mirroring card actions
- 3-tier clipboard fallback (Clipboard API → execCommand → prompt) — critical for non-secure HTTP LAN
- CSV export with exact column order (Date Requested, Requested By, Email, Phone, Equipment, Plat Number, Purpose, Return Date, Status, Condition/Fuel, Borrow Time, Approved By)
- Dark mode toggle persisted to `localStorage('kwm_dark_mode')`
- Notification toasts with auto-dismiss
- Unseen request badge with `localStorage('kwm_seen_requests')` persistence
- Real-time Firestore sync via `onSnapshot` with 100ms Lucide re-render timeout
- Firebase anonymous auto sign-in on mount

**Quality attributes (differentiators — why the refactor is worthwhile):**
- Component decomposition (no component >100 lines, clear props interface)
- State locality (form state in RequestForm, filter state in FilterBar, modal state in respective modals)
- Reduced useState count in App (only cross-cutting state: requests, auth, dark mode, view)
- Lower regression risk per change (smaller blast radius per component)
- Preserved single-file deployment (no build step, no npm, `python -m http.server`)
- Identical Firestore data shape (no schema migration)

**Defer to v2+:**
- ES modules + import maps migration (needs htm syntax conversion, import maps, separate `.js` files)
- Firebase modular SDK migration (v9+ API)
- New features, error boundaries, performance optimization, TypeScript, tests
- React 19 upgrade (requires ESM loading)

### Architecture Approach

Four clean layers, extracted bottom-up in 7 ordered `<script type="text/babel">` blocks within the single `index.html`:

1. **Block 1: Constants & Config** — `firebaseConfig`, `STATUS_COLORS`, `INPUT_CLS`, `SELECT_CLS`, Firebase init (`app`, `auth`, `db` globals). No dependencies.
2. **Block 2: Utilities** — Pure functions: `isOverdue`, `getDaysInfo`, `getRequestDuration`, `getEquipmentLabel`, `getEquipmentIcon`, `copyDetails`, `exportCSV`, `handleNotify`. No React dependency.
3. **Block 3: Firebase Service** — Firestore CRUD: `subscribeRequests(callback)`, `submitRequest(payload)`, `updateRequestStatus(id, status, extra)`, `fetchPasscode()`, `updatePasscode(new)`. Depends on Block 1 globals (`db`, `appId`).
4. **Block 4: Shared UI** — `Icon`, `NotificationToast`, `StatusPills`, `EmptyState`. Depends on Block 1 (`STATUS_COLORS`).
5. **Block 5: Modals** — `PasscodeModal`, `ReturnConditionModal`, `DetailModal`. Each receives show state + data + callbacks as props.
6. **Block 6: View Components** — `Header`, `RequestForm`, `FilterBar`, `RequestCard`. Each receives relevant state slice + specific callbacks.
7. **Block 7: App Shell + Render** — `App` component (all `useState`, `useEffect`, `useMemo`), `ReactDOM.createRoot → render`. Depends on all above.

**Key architectural decisions:**
- **Prop drilling, NOT React Context** — 14 useState, 5 nesting levels is below Context threshold. Context re-renders all consumers; at this scale props are explicit and debuggable.
- **Global scope sharing via script order** — `function` declarations in earlier blocks are visible to later blocks. `const`/`let` are block-scoped and NOT visible across `<script>` tags — use `function` keyword or `window.X = ...`.
- **No import/export** — Babel standalone transforms `import` → `require()` (babel/babel#12059). ES modules are a dead end for this stack.
- **Data flow is strictly UP (read) and DOWN (props + callbacks)** — Firebase → App Shell (useState) → UI Components (props) → Firebase Service (callbacks).

### Critical Pitfalls

1. **Firebase Listener Leaks (#1 risk)** — `onSnapshot` (line 143) and `onAuthStateChanged` (line 142) both return cleanup functions from `useEffect`. If extraction loses the `return () => unsub()`, duplicate listeners accumulate — doubled network traffic, doubled Firestore reads, free-tier exhaustion. *Prevention: inventory all 5 effects before extraction; every moved effect must preserve exact cleanup return.*

2. **Babel Standalone Module Incompatibility** — ES modules (`import`/`export`) are fundamentally broken with `type="text/babel"`. Babel transforms `import` → `require()`, and `<script type="module">` is incompatible with `<script type="text/babel">`. *Prevention: zero `import`/`export` statements; use global scope via script order.*

3. **Passcode Gate State Coupling** — The passcode flow touches 5 state atoms atomically (`isManagerAuthenticated`, `showPasscodeModal`, `view`, `markRequestsAsSeen`, `currentPasscode`). The `document.querySelector('[data-passcode-unlock]')` cross-component DOM query breaks if the button moves. *Prevention: keep auth state in parent; pass `onUnlock` callback; replace DOM query with React ref.*

4. **Borrow/Return Behavioral Drift** — `cameraModel` stores `'Other'` not custom text; `'Approved'` covers both approved and approved+passed; clipboard 3-tier fallback is actively used on non-secure HTTP; `platNumber` is separate state that must reset with form. *Prevention: copy-paste extraction only, zero logic changes; run the full borrow→approve→pass→return flow on a real device after each step.*

5. **Lucide Icon Timing** — `lucide.createIcons()` via 100ms `setTimeout` after snapshot updates is a DOM-scanning hack. Icons won't render in newly extracted components if the call isn't preserved. *Prevention: verify icons render in all views after every extraction; do NOT "fix" the icon approach.*

## Implications for Roadmap

Based on research, this is a **single-phase, behavior-preserving decomposition** — one milestone, 7 ordered extraction steps, each independently verifiable. The roadmap should NOT split this into multiple phases; it's one logical unit of work with a clear build order.

### Phase 1: Multi-Block Decomposition

**Rationale:** This IS the milestone. The 7 extraction steps follow strict dependency order — each block can only reference functions/components from blocks above it. Every step is independently verifiable (reload + full UAT checklist). The decomposition is pure code movement, not rewriting.

**Delivers:** A maintainable `index.html` with 7 ordered `<script type="text/babel">` blocks, a `js/` directory with external `.jsx` files (optional second step), and a reduced `App` component holding only cross-cutting state.

**Addresses:** All 17 table-stakes behaviors (preserved identically), all 12 quality attributes (component decomposition, state locality, reduced useState count, lower regression risk), and the single Active requirement from PROJECT.md.

**Avoids:** Firebase listener leaks (effect inventory before extraction), Babel module incompatibility (zero imports), passcode gate coupling (keep auth state in parent), borrow/return drift (copy-paste only), Lucide timing (preserve setTimeout hack).

**Step order (each step = one extraction):**
1. Constants & Config (firebaseConfig, COLORS, CLASSES, Firebase init)
2. Utilities (pure functions — isOverdue, getDaysInfo, copyDetails, exportCSV, etc.)
3. Firebase Service (subscribeRequests, submitRequest, updateStatus, fetchPasscode, updatePasscode)
4. Shared UI (Icon, Notification, StatusPills, EmptyState)
5. Modals (PasscodeModal, ReturnConditionModal, DetailModal)
6. View Components (Header, RequestForm, FilterBar, RequestCard)
7. App Shell + Render (App component with all useState/useEffect/useMemo, ReactDOM.createRoot)

**Step 2 (optional):** External file split — move each block's code into separate `js/*.jsx` files loaded via `<script type="text/babel" src="js/...">`. Same Babel standalone, same global scope, same deployment model. This gives editors real `.jsx` syntax highlighting and prepares for future ESM migration.

### Research Flags

Phases likely needing deeper research during planning:
- **None** — This is a well-documented, mechanical decomposition. All stack constraints are verified with primary sources, all pitfalls are catalogued from direct codebase analysis.

Phases with standard patterns (skip research-phase):
- **Phase 1 (entire milestone)** — Multi-inline-block decomposition is a documented Babel standalone pattern. The 7-step build order is fixed by dependency analysis. No API research needed. No new integrations.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | **HIGH** | Every CDN URL HEAD-verified live; Babel standalone `src=` XHR behavior confirmed from source; React 19 UMD removal verified; Tailwind v4 incompatibility documented |
| Features | **HIGH** | All 17 behaviors extracted from direct `index.html` analysis; 12 anti-features cross-referenced with PROJECT.md constraints; 20-scenario validation checklist covers all user flows |
| Architecture | **HIGH** | Component boundaries extracted from actual code (not speculative); Phase 1 feasibility confirmed by Babel standalone docs; Phase 2 ESM feasibility confirmed by esm.sh/htm docs |
| Pitfalls | **HIGH** | All pitfalls from direct codebase analysis; Firebase listener inventory complete (2 cleanup-returning effects); passcode gate coupling traced through 5 state atoms; borrow/return conventions documented from code |

**Overall confidence:** HIGH — This is a mechanical decomposition of a small, well-understood codebase with documented constraints and verified tooling. No unknowns remain.

### Gaps to Address

- **External file split timing:** The research recommends both inline-block decomposition (Phase 1, Step 1-7) and external file split (Phase 1, Step 2). The roadmapper should decide whether these are one phase or two — inline-block is lower risk, external files give better editor experience.
- **`data-passcode-unlock` DOM query:** The research flags this as a Phase 2 concern, but if Step 5 (Modals) extracts PasscodeModal, the DOM query must be handled during extraction. Decision: replace with React ref or callback prop during Step 5 extraction, not deferred.
- **Validation execution:** The 20-scenario checklist requires manual testing on the actual LAN deployment (`http://10.0.6.12:8080`) with staff phones — this cannot be automated in the current stack.

## Sources

### Primary (HIGH confidence)
- `index.html` (532 lines) — direct analysis of all user-visible behaviors, state hooks, effects, and JSX
- React 19 Upgrade Guide (react.dev/blog/2024/04/25/react-19-upgrade-guide) — UMD builds removed confirmed
- @babel/standalone source (`transformScriptTags.ts`) — XHR loading, document-order execution, `runtime: 'classic'`
- Babel 8.0.0 release blog (babeljs.io/blog/2026/06/16/8.0.0) — ESM-only for Node, not browser
- Tailwind CSS docs — Play CDN page, dark-mode page, v4 `@tailwindcss/browser` limitations
- Firebase docs — compat API still shipped, modular upgrade path documented
- esm.sh README — `esm.sh/run` inline-only, 60 builds/min, no `src` handling verified
- Lucide getting-started docs — UMD CDN pattern, version pinning recommendation
- npm registry live queries 2026-09-11 — all version numbers verified
- unpkg + gstatic HEAD checks — all pinned URLs return 200 text/javascript

### Secondary (MEDIUM confidence)
- Babel issue #12059 — ES modules with babel-standalone confirmed broken by design
- Firebase esm.sh issue #1075 — "Service firestore not available" with compat via ESM
- Python 3.12 `mimetypes` verification — `.jsx` returns `(None, None)` → served as `octet-stream`

### Tertiary (LOW confidence)
- None — all research is from primary or verified secondary sources

---
*Research completed: 2026-09-11*
*Ready for roadmap: yes*
