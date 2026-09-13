# Phase 1 — Multi-Block Decomposition: Context

## Phase Goal
Decompose the monolithic single-file `index.html` into 7 ordered `<script type="text/babel">` blocks while preserving all 17 existing behaviors, zero user-visible change.

## Locked Decisions

### 1. File Structure: Inline Blocks
All 7 blocks remain inside `index.html` as separate `<script type="text/babel">` tags. No external `.jsx` files. Preserves the single-file constraint.

### 2. Checkpoint Strategy: One Commit Per Block
Each of the 7 block extractions gets its own commit. Enables easy rollback if a specific extraction breaks something.

### 3. Verification: Test After Each Block
Quick reload + spot check after each extraction (~30 seconds). Catches issues immediately and isolates which extraction caused any problem.

### 4. Known Hacks: Preserve As-Is
- `data-passcode-unlock` DOM query: copy-paste exactly, no cleanup
- `lucide.createIcons()` 100ms timeout: copy-paste exactly, no cleanup
- All field names, status strings, and payload construction: byte-identical

## Build Order (from ARCHITECTURE.md research)

| Order | Block | Content | Depends On |
|-------|-------|---------|------------|
| 1 | Constants | `EQUIPMENT_OPTIONS`, `CONDITION_OPTIONS`, `CAMERA_MODELS`, `darkModeListenerAdded` | CDN globals |
| 2 | Utilities | `copyDetails()`, `formatDate()`, `truncate()`, `getReturnRequirement()` | Constants |
| 3 | Firebase Service | `firebaseConfig`, `app`, `auth`, `db`, `signIn()` | — |
| 4 | Shared UI | `Icon`, `Notification`, `PasscodeModal` | Firebase (db ref for passcode save) |
| 5 | View Components | `EquipmentForm`, `ManagerView` (incl. inline request cards, modals) | Shared UI, Constants, Utilities |
| 6 | App Shell | `App` component — all state, effects, handlers | Everything above |
| 7 | Render | `ReactDOM.createRoot(...).render(<App />)` | App Shell |

## Gray Area Resolutions
1. **File structure** → Inline blocks (not external files)
2. **Checkpoint strategy** → One commit per block
3. **Verification** → Test after each block extraction
4. **Known hacks** → Preserve `data-passcode-unlock` and `lucide.createIcons()` 100ms timeout exactly as-is

## Behavior Contract
- Every extraction is a **pure move** — copy code, don't edit it
- All 14 `useState` hooks stay in the App component
- All 5 `useEffect` hooks stay in the App component (with their cleanup functions)
- All event handlers stay in the App component
- Components receive state + callbacks via props (prop drilling, not Context)
- After each extraction: reload → verify UI identical → spot check one interaction

## Verification Checklist (post-phase)
- [ ] App loads without console errors
- [ ] Dark mode toggle works
- [ ] Equipment form submits (DJI, Car, Other variants)
- [ ] Manager passcode gate works (Enter key submits)
- [ ] Request cards render with correct status colors
- [ ] Approve/Reject flow works
- [ ] "ITEM PASSED" button works
- [ ] Return flow works
- [ ] Detail modal shows all fields
- [ ] Filter + search works
- [ ] CSV export works
- [ ] Clipboard copy works on phone (HTTP, non-secure)
- [ ] Icons render in all views
- [ ] Unseen badge count works

## Scope Boundaries
- **In scope:** Extract code from one block into 7 blocks, commit each extraction
- **Out of scope:** State management changes, config-driven features, passcode persistence changes, icon timing cleanup
