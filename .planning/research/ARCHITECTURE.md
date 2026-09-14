# Architecture Research

**Domain:** Single-file React CDN equipment-log app — decomposition into maintainable structure
**Researched:** 2026-09-11
**Confidence:** HIGH (stack constraints well-documented; decomposition patterns well-understood)

## System Overview — Current State

The entire app lives in a single 532-line `index.html` with one monolithic `App` component containing ~14 `useState` hooks, 4 `useEffect` blocks, 7 utility functions, 1 inline sub-component (`Icon`), and all UI rendering (form, dashboard, 3 modals, notifications, header/nav). Firebase is initialized as globals (`window.firebase`) via compat UMD scripts. React is loaded as UMD globals. Babel standalone transpiles a single `<script type="text/babel">` block containing everything.

```
┌─────────────────────────────────────────────────────────────┐
│                        index.html                            │
├─────────────────────────────────────────────────────────────┤
│  <script> CDN: React UMD, ReactDOM UMD, Babel, Lucide,     │
│              Firebase compat (app, firestore, auth)          │
├─────────────────────────────────────────────────────────────┤
│  <script type="text/babel">  ← SINGLE BLOCK                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Constants: firebaseConfig, STATUS_COLORS, CLS vars │    │
│  │  Firebase Init: app, auth, db globals                │    │
│  │  Icon component                                      │    │
│  │  App component (monolith):                           │    │
│  │    14 × useState   (lines 69-92)                     │    │
│  │    4  × useEffect  (lines 141-151)                   │    │
│  │    4  × useMemo    (lines 95-122)                    │    │
│  │    7  × helper fn  (lines 124-139)                   │    │
│  │    10 × async fn   (lines 156-224)                   │    │
│  │    renderFormFields() (lines 228-287)                │    │
│  │    main JSX return  (lines 289-527)                  │    │
│  └─────────────────────────────────────────────────────┘    │
│  ReactDOM.createRoot → render(<App />)                      │
└─────────────────────────────────────────────────────────────┘
```

## Component Boundary Analysis

After reading every line of the existing code, these are the **natural component boundaries** — distinct functional units that already exist as named sections within the monolith:

### Layer 1: Data / Service Layer

| Unit | Lines | State | Purpose |
|------|-------|-------|---------|
| Firebase config & init | 45-57 | — (globals) | `firebaseConfig`, `appId`, `DEFAULT_PASSCODE` (sole passcode source — edit to rotate), `app/auth/db` globals |
| Firestore reads/writes | 143, 146-151, 156-174 | — | `onSnapshot` listener, `submitRequest`, `updateStatus`, `markAsReturned` |
| CSV export | 212-224 | — | Pure data transform + download |
| Clipboard copy | 178-210 | — | Pure data transform + clipboard API with 3-tier fallback |

### Layer 2: Derived Data / Business Logic

| Unit | Lines | State | Purpose |
|------|-------|-------|---------|
| `statusCounts` | 97-107 | useMemo | Counts by status (pending, approved, overdue, returned, rejected) |
| `filteredRequests` | 109-122 | useMemo | Filter requests by status, equipment, search, date range |
| `borrowerCounts` | 95 | useMemo | Request count per borrower email |
| `isOverdue` / `getDaysInfo` / `getRequestDuration` / `getEquipmentLabel` / `getEquipmentIcon` | 124-139 | Pure functions | Date arithmetic, equipment label resolution |

### Layer 3: UI Components (currently inline JSX sections)

| Component | Lines | Props needed | Purpose |
|-----------|-------|-------------|---------|
| `Header` / Nav | 291-309 | `view`, `setView`, `isManagerAuthenticated`, `isDarkMode`, `setIsDarkMode`, `unseenCount`, `markRequestsAsSeen` | Brand, tab nav (Form/Manager), dark mode toggle, badge count |
| `RequestForm` | 314-317 + `renderFormFields` 228-287 | `submitRequest`, `isSending`, `formData`, `updateForm`, `platNumber`, `setPlatNumber` | Equipment borrow form with conditional vehicle fields |
| `FilterBar` | 321-346 | `filters`, `setFilters`, `exportCSV`, `filteredRequests` | Status/equipment/search/date filters + export button |
| `StatusPills` | 348-354 | `statusCounts` | Color-coded count badges |
| `RequestCard` | 362-406 | `req`, `isOverdue`, `getDaysInfo`, `getRequestDuration`, `borrowerCounts`, `setDetailModal`, `updateStatus`, `approverName`, `setApproverName`, `copyDetails`, `setReturnConditionModal` | Individual request card with action buttons |
| `EmptyState` | 356-361 | `requests.length`, `filters` | "No requests" placeholder |
| `PasscodeModal` | 413-441 | `showPasscodeModal`, `setShowPasscodeModal`, `passcodeAttempt`, `setPasscodeAttempt`, `setIsManagerAuthenticated`, `setView`, `markRequestsAsSeen`, `handleNotify` | Admin gate — validation compares against the `DEFAULT_PASSCODE` constant; no rotation control |
| `ReturnConditionModal` | 443-465 | `returnConditionModal`, `setReturnConditionModal`, `returnCondition`, `setReturnCondition`, `markAsReturned`, `getEquipmentLabel` | Mark equipment as returned with condition |
| `DetailModal` | 467-524 | `detailModal`, `setDetailModal`, `setReturnConditionModal`, `setReturnCondition`, `getEquipmentLabel`, `updateStatus`, `copyDetails`, `approverName`, `setApproverName`, `handleNotify` | Full request detail view with inline actions |
| `NotificationToast` | 411 | `showNotification` | Bottom toast notification |

### Layer 4: App Shell (orchestrator)

| Unit | Lines | Purpose |
|------|-------|---------|
| `App` component | 68-527 | Owns all state, wires everything together, renders view switch (request vs manager) |
| View switch | 313, 318 | Conditional: form view or dashboard view |

## Recommended Architecture

### Phase 1: Decompose within the single file (recommended — zero-risk refactor)

Split the single `<script type="text/babel">` into **ordered inline blocks**. Each block defines functions/components that later blocks can reference via global scope. Babel standalone transpiles each block independently and synchronously, so function declarations in earlier blocks are available as globals in later blocks.

```
┌──────────────────────────────────────────────────────────────┐
│  index.html (still one file, still no build step)            │
├──────────────────────────────────────────────────────────────┤
│  <script> CDN libs (React, ReactDOM, Firebase, Lucide)       │
│  <style> ... </style>                                        │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 1: CONSTANTS & CONFIG                               │
│      firebaseConfig, STATUS_COLORS, INPUT_CLS, SELECT_CLS    │
│      Firebase init (app, auth, db)                            │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 2: UTILITY FUNCTIONS                                │
│      isOverdue, getDaysInfo, getRequestDuration               │
│      getEquipmentLabel, getEquipmentIcon, exportCSV           │
│      copyToClipboard, handleNotify                            │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 3: FIREBASE SERVICE (exposed as window.DB)          │
│      subscribeRequests(onUpdate) → returns unsubscribe        │
│      submitRequest(payload)                                   │
│      updateRequestStatus(id, status, extra)                   │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 4: SHARED UI COMPONENTS                              │
│      Icon, Notification, StatusPills, EmptyState              │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 5: MODAL COMPONENTS                                  │
│      PasscodeModal, ReturnConditionModal, DetailModal         │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 6: VIEW COMPONENTS                                   │
│      RequestForm, FilterBar, RequestCard, Header              │
├──────────────────────────────────────────────────────────────┤
│  <script type="text/babel">                                  │
│    Block 7: APP SHELL + RENDER                                │
│      App component (orchestrator with useState/useEffect)     │
│      ReactDOM.createRoot → render(<App />)                    │
└──────────────────────────────────────────────────────────────┘
```

**Key properties of this approach:**

| Property | Value |
|----------|-------|
| Deployment change | **Zero** — still one index.html, still python http.server |
| Behavior change | **Zero** — same JSX, same Babel, same React, same Firebase |
| Module isolation | Global scope (components are functions on `window`) |
| Coupling reduction | Each block has a focused responsibility |
| Risk | **Minimal** — moving code between blocks, not rewriting it |

**Critical implementation details:**

1. Each `<script type="text/babel">` block is transpiled independently by Babel standalone and executed synchronously in document order
2. Function declarations (not `const` arrow functions) in earlier blocks are hoisted to `window` and visible in later blocks
3. The `Icon` component and constants like `STATUS_COLORS` must be declared with `function` keyword (not `const =`) to be available as globals in later blocks
4. `const` and `let` declarations are block-scoped and NOT visible across `<script>` tags — use `window.X = ...` explicitly or declare with `function`/`var`
5. Firebase `db` is already a global (`window.db`) from Block 1, accessible everywhere

### Phase 2 (optional, strategic): ES Modules + Import Maps

If the app grows past ~800 lines or the team wants true module encapsulation, migrate to ES modules. This requires:

1. **JSX → htm migration:** Replace JSX syntax with htm tagged template literals
2. **Import maps** for React dependency resolution
3. **Firebase stays as UMD globals** (esm.sh/firebase compat has known "Service not available" issues — issue #1075; official Firebase docs recommend `<script>` tags for no-build)
4. **Separate .js files** for each component

**Why htm and not "keep JSX with Babel standalone + data-type=module":**

Babel standalone with `data-type="module"` has documented, unfixed issues:
- Default presets transform `import` → `require()`, breaking ES modules (babel/babel#12059)
- Babel standalone does NOT transpile ES modules imported by the entry script — only the entry file is processed
- External `.jsx` files loaded via `src` attribute are fetched but NOT transformed for their own imports
- Requires a service-worker hack to intercept and transform imported modules at runtime

htm is a 1KB library (1247 bytes) that provides JSX-like syntax using tagged template literals. The syntax differences are small:

```jsx
// JSX (current)
<div className="text-white">
  <input className={INPUT_CLS} value={name} onChange={e => setName(e.target.value)} />
  <button onClick={() => submit()}>Submit</button>
</div>

// htm equivalent
html`<div class="text-white">
  <input class=${INPUT_CLS} value=${name} onInput=${e => setName(e.target.value)} />
  <button onClick=${() => submit()}>Submit</button>
</div>`
```

**htm syntax rules:**
- `className` → `class` (htm uses real HTML attributes)
- `onChange` → `onInput` for text inputs (htm preserves native events)
- Dynamic values: `{expr}` → `${expr}`
- Component references: `<Component>` → `<${Component}>`
- Closing tags: `</Component>` → `<//>`
- Self-closing: `<Component />` → `<${Component} />`

**ES Module file structure (Phase 2):**

```
index.html                          ← import map + tiny bootstrap
├── <script type="importmap">
│     { "react": "https://esm.sh/react@18.2.0",
│       "react-dom/client": "https://esm.sh/react-dom@18.2.0/client",
│       "htm": "https://esm.sh/htm@3.1.1/react?external=react" }
│
├── <script> CDN: Firebase compat (UMD globals)
│
└── <script type="module">
      import { createRoot } from "react-dom/client";
      import { html } from "htm";
      import { App } from "./app.js";
      createRoot(document.getElementById("root")).render(html`<${App} />`);

app.js                              ← App shell (orchestrator)
├── import { Header } from "./components/header.js";
├── import { RequestForm } from "./components/request-form.js";
├── import { Dashboard } from "./components/dashboard.js";
├── import { PasscodeModal } from "./components/passcode-modal.js";
├── import { ReturnModal } from "./components/return-modal.js";
├── import { DetailModal } from "./components/detail-modal.js";
└── App component: useState/useEffect, wires sub-components

components/header.js                ← Brand + nav + dark mode toggle
components/request-form.js          ← Borrow form with conditional fields
components/dashboard.js             ← FilterBar + RequestCard list + EmptyState
components/filter-bar.js            ← Status/equipment/search/date filters
components/request-card.js          ← Individual request card with actions
components/passcode-modal.js        ← Admin gate (constant passcode)
components/return-modal.js          ← Mark returned with condition
components/detail-modal.js          ← Full detail view with inline actions
components/notification.js          ← Toast notification
components/shared.js                ← Icon helper, STATUS_COLORS, CSS class constants

services/firebase.js                ← Firebase CRUD wrapper (uses window.firebase)
  export function subscribeRequests(onUpdate)
  export async function submitRequest(payload)
  export async function updateRequestStatus(id, status, extra)

utils/helpers.js                    ← Date/label/clipboard utilities
  export function isOverdue(r)
  export function getDaysInfo(r)
  export function getEquipmentLabel(r)
  export function copyToClipboard(r)
  export function exportCSV(data)
```

**Phase 2 import map for this project:**

```html
<script type="importmap">
{
  "imports": {
    "react": "https://esm.sh/react@18.2.0",
    "react/": "https://esm.sh/react@18.2.0/",
    "react-dom/client": "https://esm.sh/react-dom@18.2.0/client?external=react",
    "htm": "https://esm.sh/htm@3.1.1/react?external=react"
  }
}
</script>
```

Firebase stays as UMD globals — no import map entry needed. Access via `window.firebase` in service files.

## Data Flow

### Current Data Flow (monolith)

```
Firebase Firestore (cloud)
    ↕ onSnapshot real-time listener
App component (single)
    ├── useState: requests, filters, view, modals, form, auth state
    ├── useMemo: filteredRequests, statusCounts, borrowerCounts
    ├── useEffect: Firestore subscription, dark mode
    ├── submitRequest → Firestore.add
    ├── updateStatus → Firestore.update
    └── render: form | dashboard (filter → card list → modals)
```

### Decomposed Data Flow (Phase 1)

```
Firebase Firestore (cloud)
    ↕ onSnapshot (in Block 3: Firebase Service)
Block 3: FirebaseService
    ├── subscribeRequests(callback) → sets requests[]
    ├── submitRequest(payload)
    └── updateRequestStatus(id, status, extra)
Block 7: App Shell
    ├── useState: requests ← from subscribeRequests
    ├── useState: filters, view, modals, form, auth
    ├── useMemo: filteredRequests, statusCounts, borrowerCounts
    ├── passes data + callbacks as props to:
Block 6: View Components
    ├── Header ← view, setView, auth, darkMode, unseenCount
    ├── RequestForm ← formData, updateForm, submitRequest, isSending
    ├── Dashboard → FilterBar ← filters, setFilters
    │              → RequestCard ← request, onApprove, onReject, onReturn, onCopy
    │              → EmptyState ← empty
Block 5: Modals ← show state + data + callbacks
Block 4: Shared UI ← Icon, Toast
```

### Data Flow Direction (Rule)

```
Firebase ──read──→ App Shell (useState) ──props──→ UI Components
                    ↑                              ↓
              Firebase Service ←──callback──── UI Actions (onClick)
```

**Direction is strictly: UP (read) and DOWN (props + callbacks).**

No Context needed. No Redux. No external state library. For this app (~200 lines of state, ~14 useState hooks, single-user internal tool), **prop drilling is the correct pattern** — it makes data flow explicit and debuggable. React Context would add indirection without benefit at this scale.

### State Ownership After Decomposition

| State | Owner | Passed To | Notes |
|-------|-------|-----------|-------|
| `requests` | App Shell | Dashboard, RequestCard, FilterBar | Source of truth from Firestore |
| `filters` | App Shell | FilterBar (reads+writes), Dashboard (reads) | Local UI state |
| `view` | App Shell | Header (reads+writes) | Tab selection |
| `formData` | App Shell | RequestForm (reads+writes) | Form field values |
| `isManagerAuthenticated` | App Shell | Header, PasscodeModal | Auth gate |
| `showPasscodeModal` | App Shell | PasscodeModal | Modal toggle |
| `detailModal` | App Shell | DetailModal | Selected request for detail view |
| `returnConditionModal` | App Shell | ReturnConditionModal | Selected request for return |
| `showNotification` | App Shell | NotificationToast | Toast message + type |
| `isDarkMode` | App Shell | Header (reads+writes) | Persisted in localStorage |

## Anti-Patterns to Avoid

### Anti-Pattern 1: React Context for This App

**What people do:** Wrap everything in a `RequestsProvider` / `AuthProvider` context to "avoid prop drilling."

**Why it's wrong:** At 14 useState hooks and ~5 child levels of nesting, prop drilling is trivial and explicit. Context adds indirection that makes the data flow harder to trace. Context re-renders ALL consumers when any value changes — with `requests` as context, a filter change re-renders every card.

**Do this instead:** Pass props explicitly. If a component doesn't use a value, don't pass it. If drilling exceeds 3 levels, extract a child component closer to the data.

### Anti-Pattern 2: Converting Constants to ES Modules When They're Just Strings

**What people do:** Create `constants/STATUS_COLORS.js` exporting `STATUS_COLORS` as a module.

**Why it's wrong:** For Phase 1 (inline blocks), constants are just `var STATUS_COLORS = {...}` declared in Block 1. Over-modularizing simple constants adds file noise. For Phase 2 (ES modules), inline them in the component file that uses them — they're not shared widely enough to warrant a separate module.

**Do this instead:** Keep constants at the top of the block/file that uses them. Only extract to a shared file if 3+ components reference the same constant.

### Anti-Pattern 3: Babel Standalone + data-type="module" for Multi-File JSX

**What people do:** Load external `.jsx` files via `<script type="text/babel" data-type="module" src="App.jsx">` expecting `import` to work.

**Why it's wrong:** Babel standalone transforms `import` to `require()` by default. Even with custom presets, imported modules are NOT themselves transpiled — JSX syntax in imported files causes syntax errors. This pattern is broken by design (babel/babel#12059).

**Do this instead:** Use the multi-inline-block approach (Phase 1) or htm + ES modules (Phase 2). Both are documented, working patterns.

### Anti-Pattern 4: Decomposing Without Identifying Natural Boundaries First

**What people do:** Split code into files/components at arbitrary line counts ("split every 100 lines").

**Why it's wrong:** The decomposition boundary should follow data flow and responsibility, not line count. The natural boundaries in this app are: data service → business logic → UI components → app shell — not arbitrary chunks.

**Do this instead:** Follow the 4-layer architecture identified above (Service → Logic → Components → Shell). Each layer has clear input/output contracts.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Firebase Firestore | UMD global `window.firebase`, accessed as `db.collection(...)` | Compat namespaced API — NOT modular v9+ API. Keep as globals. |
| Firebase Auth (anonymous) | UMD global, `firebase.auth().signInAnonymously()` | Single use on mount. No complex auth flow. |
| Lucide Icons | UMD global `window.lucide`, re-init with `lucide.createIcons()` after DOM updates | Needs 100ms timeout after Firestore snapshot updates. |
| Tailwind CSS | CDN `<script>` with config, global utility classes | No module integration needed. |

### Internal Boundaries (after decomposition)

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Block 1 (Config) → Block 3 (Firebase Service) | Shared globals: `db`, `appId`, `DEFAULT_PASSCODE` | FirebaseService reads `db`/`appId` from global config; `DEFAULT_PASSCODE` is read directly by the App Shell |
| Block 3 (Firebase Service) → Block 7 (App Shell) | `subscribeRequests(callback)` pattern | Real-time subscription via callback, not return |
| Block 7 (App Shell) → Block 6 (View Components) | Props: data in, callbacks out | Standard React one-way data flow |
| Block 7 (App Shell) → Block 5 (Modals) | Props: show state + data + onConfirm | Modals are controlled components |
| Block 6 (RequestCard) → Block 7 (App Shell) | Callbacks: onApprove(), onReject(), onReturn() | Events bubble up to App Shell which calls Firebase Service |
| Block 2 (Utilities) → All Blocks | Pure function imports (no state, no side effects) | Stateless helpers called by any block |

## Scaling Considerations

| Scale | Architecture Adjustment |
|-------|------------------------|
| Current (internal LAN, 5-20 users) | Phase 1 multi-block decomposition is sufficient |
| Growth (50+ requests/day, 5+ managers) | Consider Phase 2 ESM migration for real code organization |
| Scale (multi-team, external users) | Add proper auth, move to a real build (Vite), consider React 19 + modular Firebase |

### Scaling Priorities

1. **First bottleneck:** The monolith makes adding new features risky (current pain point — addressed by Phase 1)
2. **Second bottleneck:** Global state namespace collisions if more than ~20 components are added (addressed by Phase 2 ESM)
3. **Third bottleneck:** Babel standalone startup time becomes noticeable (2-3s on mobile) — addressed by removing Babel (Phase 2 htm approach eliminates this)

## Build Order (Implementation Dependencies)

### Phase 1 Build Order

The decomposition must follow dependency order — each block can only reference functions/components from blocks above it:

```
Step 1: Block 1 — Constants & Config (no dependencies)
         Extract firebaseConfig, COLORS, CLASSES, firebase init
         ↓
Step 2: Block 2 — Utilities (depends on Block 1: no actual deps, but logically follows)
         Extract isOverdue, getDaysInfo, getRequestDuration, getEquipmentLabel,
         copyToClipboard, exportCSV, handleNotify
         ↓
Step 3: Block 3 — Firebase Service (depends on Block 1: db, appId)
         Extract subscribeRequests, submitRequest, updateStatus
         Expose as window.DB or plain global functions
         ↓
Step 4: Block 4 — Shared UI (depends on Block 2: none; Block 1: STATUS_COLORS)
         Extract Icon, Notification, StatusPills, EmptyState
         ↓
Step 5: Block 5 — Modals (depends on Block 1: CLASSES, Block 2: getEquipmentLabel, CLASSES)
         Extract PasscodeModal, ReturnConditionModal, DetailModal
         Each receives: show state + data + callbacks as props
         ↓
Step 6: Block 6 — View Components (depends on Block 1: CLASSES, Block 2: utilities, Block 4: shared UI)
         Extract Header, RequestForm, FilterBar, RequestCard
         Each receives: relevant slice of state + specific callbacks
         ↓
Step 7: Block 7 — App Shell (depends on ALL above blocks)
         App component: all useState, all useEffect, all useMemo
         Passes props down to child components
         ReactDOM.createRoot → render
```

**Each step should be independently verifiable:** After extracting a block, the app should still render and work identically. The extraction is pure code movement — no logic changes.

### Phase 2 Build Order (if pursued)

```
Step 1: Create import map in index.html, load htm, verify React loads
Step 2: Create utils/helpers.js (pure functions, no React)
Step 3: Create services/firebase.js (uses window.firebase global)
Step 4: Create components/shared.js (Icon, constants, htm binding)
Step 5: Create each component file (one at a time, convert JSX → htm)
Step 6: Create app.js shell (orchestrator)
Step 7: Create index.html bootstrap (import map + module entry)
Step 8: Remove Babel standalone + all text/babel blocks
```

**Critical Phase 2 detail:** htm syntax conversion is mechanical but must be exact. The `className` → `class` change affects every element. Use find-and-replace for bulk conversion, then manual review for edge cases (dynamic className with template literals, spread props, etc.).

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Component boundaries | **HIGH** | Extracted from actual code analysis — every boundary maps to an identifiable section in the monolith |
| Phase 1 feasibility | **HIGH** | Multi-inline-block is a documented Babel standalone pattern; no behavior changes, only code movement |
| Phase 2 ESM feasibility | **HIGH** | htm + esm.sh + import maps is the official no-build React pattern (react.dev docs, esm.sh docs) |
| Firebase ESM risk | **MEDIUM** | esm.sh/firebase compat has known issues (#1075); keep UMD globals to avoid this entirely |
| Data flow correctness | **HIGH** | Prop drilling is the correct pattern for this app size; Context would be an anti-pattern |
| Build order | **HIGH** | Dependencies follow from block ordering — straightforward |

## Sources

- esm.sh documentation: import maps, `?external` for singleton deps (https://esm.sh)
- htm library: developit/htm on GitHub — tagged templates as JSX alternative
- Babel standalone docs: `data-type="module"` and external `src` attribute behavior (babeljs.io/docs/babel-standalone)
- Babel issue #12059: ES modules with babel-standalone — import→require bug, confirmed broken by design
- Firebase JS SDK: compat library from window, ESM import compatibility (firebase.google.com/docs/web/modular-upgrade)
- Firebase esm.sh issue #1075: "Service firestore is not available" when using compat via ESM
- React docs: "React without a build step" (react.dev) — official no-build recommendation
- Preact guide: no-build workflows with import maps + htm (preactjs.com/guide/v11/no-build-workflows)

---
*Architecture research for: KWM Equipment Log — monolith decomposition*
*Researched: 2026-09-11*
