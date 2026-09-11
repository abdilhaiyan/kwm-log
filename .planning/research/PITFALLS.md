# Pitfalls Research

**Domain:** Behavior-preserving decomposition of a single-file CDN React + Firebase app
**Researched:** 2026-09-11
**Confidence:** HIGH (based on direct codebase analysis + domain knowledge)

## Critical Pitfalls

### Pitfall 1: Firebase Listener Leaks During Component Extraction

**What goes wrong:**
The `onSnapshot` listener on line 143 and `onAuthStateChanged` listener on line 142 are both returned as cleanup functions from `useEffect`. When extracting components, developers commonly move the auth effect or the Firestore subscription into a child component without preserving the cleanup return. Each mount without cleanup creates a second parallel listener. The old listener is never unsubscribed because the parent's `useEffect` cleanup is lost. Users see duplicate data updates, doubled network traffic, and eventually Firestore reads exhaust the free tier.

The current code has an additional subtlety: the `onSnapshot` effect depends on the `user` state (line 143: `}, [user]`), so React will re-run it and re-subscribe every time the auth state changes — but only because the cleanup from the previous run is returned. If a child component receives the listener setup and forgets the cleanup, you get compound leaks.

**Why it happens:**
Developers extract the "data loading" into a custom hook or child component and write `useEffect(() => { db.collection(...).onSnapshot(...) }, [])` — missing the `return () => unsub()`. Babel standalone transpilation does not catch this. There are no tests that would detect a leaked listener.

**How to avoid:**
- Before extracting, document every `useEffect` and its cleanup function. There are exactly 5 effects in the current `App` component (lines 141-151). Two have cleanup functions (lines 142, 143). Three do not (lines 141, 144, 146-151). This is the complete inventory.
- Every extraction that moves a `useEffect` must preserve the exact return value.
- After each extraction, add a console.log in the cleanup to verify it fires on unmount.
- Use the rule: if the original effect returns `unsub()`, the extracted version MUST return `unsub()`.

**Warning signs:**
- Network tab shows multiple simultaneous WebSocket connections to `firestore.googleapis.com`
- Console shows `Firestore (X.XX)` where X keeps incrementing across navigations
- Data appears to "update twice" when a document changes
- Firestore read counts in the console dashboard are 2-3x expected

**Phase to address:**
Phase 1 (Extract Components) — inventory all effects and their cleanup functions before any extraction begins.

---

### Pitfall 2: Babel Standalone Module Loading Incompatibility

**What goes wrong:**
The app uses `<script type="text/babel">` which Babel standalone transpiles at runtime. Developers try to split into multiple files and use ES module imports (`import X from './components/X.js'`). This does not work. Babel standalone does not process `import`/`export` statements in `type="text/babel"` scripts — it only handles JSX/class-properties within a single script block. ES modules also require `<script type="module">` which is incompatible with `type="text/babel"`. The result is either a silent failure (components don't load) or an explicit `SyntaxError: Cannot use import statement in a module`.

The no-build constraint means ES modules are off the table entirely. Every approach to file splitting must stay within `<script>` tag ordering.

**Why it happens:**
Developers assume "modern React = ES modules" and reach for `import`/`export` by default. The CDN+Babel approach predates widespread module adoption and uses a fundamentally different loading model.

**How to avoid:**
- Split into multiple `<script type="text/babel">` tags in dependency order within `index.html`. Each script block shares the same global scope.
- Components are defined as `const MyComponent = () => { ... }` in later script blocks, referencing earlier definitions.
- Alternative: keep all component code in one `<script type="text/babel">` block but organize with clear section comments and helper functions defined above the main `App`.
- NEVER use `import`/`export`. NEVER add `<script type="module">`.
- Consider using `window.MyComponent = MyComponent` to make inter-block dependencies explicit (defensive, but prevents accidental ordering bugs).

**Warning signs:**
- Any `import` or `export` keyword in the codebase
- `<script type="module">` tags
- Components not rendering (blank areas) with no console errors
- `ReferenceError: X is not defined` when navigating between views

**Phase to address:**
Phase 1 (Extract Components) — the component splitting strategy must be defined BEFORE any extraction starts.

---

### Pitfall 3: Passcode Gate State Coupling Breakage

**What goes wrong:**
The passcode verification flow (lines 421-437) touches 5 pieces of state atomically: `setIsManagerAuthenticated(true)`, `setShowPasscodeModal(false)`, `setView('manager')`, `markRequestsAsSeen()`, and optionally `setCurrentPasscode()` after the Firestore write. When this logic is extracted into a `PasscodeModal` component, developers commonly:
1. Move the passcode state (`passcodeAttempt`, `showPasscodeModal`, `newPasscode`, `currentPasscode`) into the child component, losing the parent's ability to read `isManagerAuthenticated`.
2. Split the success callback so that `setView('manager')` fires before `setIsManagerAuthenticated(true)`, creating a race where the manager view renders but the auth gate hasn't opened yet.
3. Lose the `data-passcode-unlock` button selector pattern (lines 419-421) where the Enter key handler reaches outside the component via `document.querySelector('[data-passcode-unlock]').click()`. This is a cross-component DOM query that breaks if the button's component tree moves.

**Why it happens:**
The passcode modal is conceptually a "separate concern" so it's a natural first extraction target. But it is deeply coupled to the parent's `isManagerAuthenticated` state and `view` state. The `data-passcode-unlock` DOM query is an invisible cross-component dependency.

**How to avoid:**
- Keep `isManagerAuthenticated` and `showPasscodeModal` as parent-level state during initial decomposition.
- Pass `onUnlock` callback from parent to child. The parent owns the atomic state transition.
- Replace `document.querySelector('[data-passcode-unlock]')` with a React ref or callback prop.
- The passcode verification + Firestore update + view switch must remain a single synchronous callback in the parent.

**Warning signs:**
- Manager view renders without auth gate being checked
- `data-passcode-unlock` appears in `querySelector` calls
- `isManagerAuthenticated` set in a child component instead of parent
- Passcode modal visible but manager content not loading (or vice versa)

**Phase to address:**
Phase 2 (State Management) — passcode gate is one of the highest-risk state extractions.

---

### Pitfall 4: Behavorial Drift in the Borrow/Return Flow

**What goes wrong:**
The borrow flow has subtle behavior that is easy to "clean up" into brokenness:
- `submitRequest` (line 156) merges `formData` with computed fields (`cameraModel` override for "Other", `platNumber` conditional, `status: 'Pending Approval'`, `requestedAt`, `createdAt`). When extracting the form, developers simplify the payload construction and drop the `cameraModel` override — silently storing "Other" as the equipment type instead of the user's custom text.
- `markAsReturned` (line 176) reads `returnConditionModal.id` to determine which request to update. If the modal state is extracted without its dependency on `returnConditionModal`, the function operates on `null` and silently does nothing.
- The "ITEM PASSED" action (line 395) sends `updateStatus(req.id, 'Approved', { passedAt: ... })` — it passes the status `'Approved'` (not `'Passed'`). This is intentional: the item is still "Approved", just marked as physically handed over. A developer cleaning this up might "fix" it to `'Passed'`, which would break the filter logic and status display.
- `copyDetails` (line 178) has a 3-tier clipboard fallback (secure context → textarea → prompt). This is deployed on `http://10.0.6.12:8080` (non-secure context), so the fallback tiers are actively used. "Simplifying" to just `navigator.clipboard.writeText()` would break clipboard on every staff phone.

**Why it happens:**
The "obvious" cleanup is often wrong. The `cameraModel` field serves double duty (stored value vs display value). The status `'Approved'` covers both "approved" and "approved + passed" states. The clipboard has a non-obvious production dependency. These are domain-specific conventions that only make sense with full context.

**How to avoid:**
- Before any extraction, write a behavior contract for each flow: given input X, expect output Y and side effect Z.
- Do NOT rename, restructure, or "improve" field values during decomposition. Copy-paste the exact payload construction.
- Run the actual borrow → approve → pass → return flow on a real device before and after each phase.
- Keep `copyDetails` exactly as-is with all 3 fallback tiers. Do not simplify.
- The `cameraModel` field must store the original value (`'Other'`), not the custom text. The custom text is in `customEquipment`.

**Warning signs:**
- Any field name changes in the Firestore payload
- Clipboard API used without fallback
- Status values changed from the exact strings: `'Pending Approval'`, `'Approved'`, `'Rejected'`, `'Returned'`
- `getEquipmentLabel` function modified or bypassed

**Phase to address:**
Phase 1 (Extract Components) — copy-paste extraction only, zero logic changes.

---

### Pitfall 5: `lucide.createIcons()` Timing and Icon Rendering

**What goes wrong:**
The app uses `lucide@latest` (CDN) which renders icons by scanning the DOM for `<i data-lucide="name">` elements. The `Icon` component (line 59) renders `<i data-lucide={name.toLowerCase()}>`. After every state change that re-renders the `Icon` component, `lucide.createIcons()` must be called to process the new DOM nodes. Currently this is done via `setTimeout(() => lucide.createIcons(), 100)` inside the `onSnapshot` callback (line 143).

When components are extracted, developers typically move the `Icon` component into its own file/block but forget that the `lucide.createIcons()` call must run AFTER every render that includes `Icon` components. If the call is only in the `onSnapshot` handler, icons will not render after form submissions, modal opens, or filter changes.

**Why it happens:**
Lucide's CDN mode is a DOM-scanning approach — it's fundamentally different from how React components normally work. The `createIcons()` call is an imperative side effect that doesn't fit React's declarative model. It's already a hack in the current code (the 100ms timeout is a race condition workaround).

**How to avoid:**
- After each component extraction, verify icons render in all views (Form, Manager, Modals, Detail).
- Consider adding a `useEffect` that calls `lucide.createIcons()` after every render of the `Icon` component. This is more robust than the current `setTimeout` in `onSnapshot`.
- Do NOT attempt to "fix" the icon approach during decomposition — that's a behavior change.

**Warning signs:**
- Blank squares or empty spaces where icons should appear
- Icons appear after a delay (the 100ms timeout is too short)
- Icons missing entirely in newly extracted components

**Phase to address:**
Phase 1 (Extract Components) — icon rendering must work in every extracted component.

---

### Pitfall 6: Loss of the `data-babel-preserve` Global Scope Contract

**What goes wrong:**
Babel standalone transpiles every `<script type="text/babel">` block independently. All blocks share the same global scope (`window`), so components defined in earlier blocks are available to later blocks. But if a developer creates a second `<script type="text/babel">` block for a component file, and that block runs BEFORE the React globals are destructured (`const { useState, useEffect, useMemo } = React` on line 44), the component will fail with `useState is not defined`.

Currently there is exactly one `<script type="text/babel">` block (line 43-530). All state hooks, effects, and components share this scope. Splitting into multiple blocks requires ordering: CDN globals first, then shared utilities, then leaf components, then the main App, then the render call.

**Why it happens:**
Developers assume all `<script>` tags are guaranteed to execute in order. They are — but the transpilation is async and timing-dependent. Babel standalone processes each `type="text/babel"` script asynchronously, and a later script may begin transpilation before an earlier one finishes. This creates non-deterministic ordering.

**How to avoid:**
- Option A (recommended): Keep one `<script type="text/babel">` block but organize with clear sections and helper functions. This avoids the ordering problem entirely.
- Option B: If splitting into multiple `<script type="text/babel">` blocks, add `data-plugins="transform-modules-commonjs"` and verify execution order. Better yet, test by adding `console.log('Block N loaded')` at the top of each block.
- NEVER use ES module `<script type="module">`.
- The `ReactDOM.createRoot` and `root.render(<App />)` call (lines 528-529) must be in the LAST script block.

**Warning signs:**
- `ReferenceError: useState is not defined` or similar
- Components rendering as empty
- Inconsistent behavior depending on network load order (some scripts load before others)

**Phase to address:**
Phase 1 (Extract Components) — script ordering strategy must be decided before any splitting.

---

## Moderate Pitfalls

### Pitfall 7: Dark Mode CSS Class Toggling Lost

**What goes wrong:**
Line 141: `useEffect(() => { localStorage.setItem('kwm_dark_mode', isDarkMode); document.documentElement.classList.toggle('dark', isDarkMode); }, [isDarkMode]);` This effect directly manipulates `document.documentElement.classList`. If the dark mode state is extracted into a context provider or separate component, the effect may fire before the DOM is ready, or the `document.documentElement` reference may be unavailable.

**How to avoid:** Keep the dark mode effect in the root `App` component. Pass `isDarkMode` as a prop or context value to children. Do NOT move the `classList.toggle` into a child component.

**Phase to address:** Phase 2 (State Management)

---

### Pitfall 8: `seenRequestIds` localStorage Sync Race

**What goes wrong:**
Lines 84 and 144: `seenRequestIds` is initialized from `localStorage` and synced back on every change. The `markRequestsAsSeen()` function (line 154) creates a new `Set` with all current request IDs. If this function is extracted into a hook or context, the closure over `requests` may become stale — the function reads `requests` from the enclosing scope, which may not be the latest value if `requests` was updated by `onSnapshot` between the render and the call.

**How to avoid:** Keep `markRequestsAsSeen` in the same component as the `requests` state. Use the functional form of `setSeenRequestIds(prev => ...)` (which the current code already does). Do NOT extract this into a separate hook without passing `requests` as a parameter.

**Phase to address:** Phase 2 (State Management)

---

### Pitfall 9: Form State Reset Silently Drops Vehicle-Specific Fields

**What goes wrong:**
Line 165: The form reset sets `formData` to a clean object, but `platNumber` is a SEPARATE state variable (line 89, `useState('')`). The reset also calls `setPlatNumber('')`. If the form component is extracted and the `platNumber` state lives in the wrong component, the reset won't clear it — leading to a stale plat number appearing on the next non-vehicle submission.

**How to avoid:** When extracting the form, keep `platNumber` in the same scope as `formData`. The reset in `submitRequest` must clear both `formData` AND `platNumber`. Verify by submitting a Vehicle request, then submitting a DJI Camera request — the plat number should not appear.

**Phase to address:** Phase 1 (Extract Components)

---

### Pitfall 10: `approverName` State Shared Across Multiple Request Cards

**What goes wrong:**
Line 90: `approverName` is a single `useState('')` shared across ALL pending request cards. This means selecting an approver for one card pre-fills the same value for all cards. This is existing behavior (not a bug introduced by decomposition), but when extracting the pending request card into its own component, developers might move `approverName` into each card — which would change behavior (each card having independent approver selection).

**How to avoid:** Preserve the shared `approverName` state in the parent during decomposition. This is a known design choice (or limitation) — do not "fix" it during a behavior-preserving refactor. Document it as a future improvement.

**Phase to address:** Phase 1 (Extract Components) — preserve existing behavior exactly.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| One giant component | Zero extraction risk | Increasingly hard to maintain; any change risks everything | Currently (but actively working to fix) |
| `document.querySelector('[data-passcode-unlock]')` cross-component DOM query | Quick Enter-key handling | Invisible coupling between components; breaks if DOM structure changes | Until Phase 2 extracts passcode modal |
| `setTimeout(() => lucide.createIcons(), 100)` | Avoids deep React+Lucide integration work | Race condition; icons may flicker or not render | Until a proper icon rendering solution is implemented |
| Shared `approverName` state across cards | Simple implementation | All cards show same approver selection | For now (small team, one approver) |
| Firebase compat namespaced API | Matches existing code, stable | Deprecated by Firebase; will eventually need migration | Entire project lifespan |
| Hardcoded `'Car'` internal value, `'Vehicles'` display | Avoids data migration | Confusing for new developers | Until data is migrated (may never be) |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Firebase `onSnapshot` | Not returning cleanup function | Always `return () => unsub()` from the effect |
| Firebase `onAuthStateChanged` | Forgetting anonymous sign-in | The effect must call `auth.signInAnonymously()` if `!u` before setting user |
| Firebase `.set()` for passcode | Using `.update()` instead | `.set()` creates the doc if it doesn't exist; `.update()` throws if missing |
| Lucide CDN icons | Assuming React state change re-renders icons | Must call `lucide.createIcons()` after DOM changes |
| Babel standalone | Using `import`/`export` syntax | Only `type="text/babel"` scripts; no ES modules |
| Clipboard API | Using only `navigator.clipboard` | Must fall back to textarea + `execCommand` + prompt for non-secure HTTP contexts |
| Tailwind CDN | Adding custom `@apply` rules | Tailwind CDN does not support `@apply`; use inline classes |
| Firestore security rules | Assuming client-side code is secure | Firestore rules are the real security boundary; client code is visible to users |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Multiple `onSnapshot` listeners | Doubled network traffic, doubled Firestore reads | Audit cleanup functions after every extraction | As soon as a listener leaks |
| Unnecessary re-renders from state proximity | UI jank on mobile phones (older Android devices at KWM) | Keep `filteredRequests` memoized; avoid passing entire `requests` array to child components that only need one request | When >50 requests exist |
| `lucide.createIcons()` on every render | Layout thrashing, flickering icons | Call only when new DOM nodes with `data-lucide` are added, not on every state change | When more than ~10 icons on screen |
| Large localStorage `seenRequestIds` Set | Slow JSON serialization on every change | Use a simple max-size (keep last 500 IDs) | When >1000 requests accumulate |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Storing passcode in client-side state only | Passcode visible in React DevTools | Current approach stores in Firestore — good; but `currentPasscode` is in React state, visible in DevTools. Consider comparing in Firestore only. |
| Firebase API key in HTML | API key is public by design for Firebase; security depends on Firestore rules, not the key | Ensure Firestore rules deny unauthenticated writes to `app_config` |
| `http://` deployment (no TLS) | Passcode sent in cleartext over LAN | Acceptable for internal LAN tool; document the risk |
| Firebase anonymous auth with no additional verification | Any browser visitor gets authenticated | Acceptable for this internal use case; all sensitive ops are passcode-gated |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Passcode modal Enter key uses `document.querySelector` | May fail if button hasn't rendered yet | Use React ref or `onKeyDown` on the input that calls the unlock handler directly |
| Dark mode toggle doesn't persist across tabs | Setting one tab's dark mode doesn't update other open tabs | Add a `storage` event listener (future improvement, not for decomposition phase) |
| Notification toast disappears after 4 seconds | Users may miss success/error messages on slow connections | Keep current behavior; document for future improvement |

## "Looks Done But Isn't" Checklist

- [ ] **Component extraction complete:** All 5 `useEffect` hooks verified with cleanup logging
- [ ] **Passcode gate works:** Enter key submits, wrong code clears, correct code unlocks, new passcode persists to Firestore
- [ ] **Form submission works:** All equipment types submit correct payload (DJI → `cameraModel: 'DJI Osmo Action 6'`, Car → `cameraModel: 'Car'` + `platNumber`, Other → `cameraModel: <custom text>`)
- [ ] **Return flow works:** `markAsReturned` sends correct `returnCondition` (fuel level for vehicles, condition for equipment)
- [ ] **Icons render in all views:** Form header icon, Manager nav icons, filter icons, card icons, modal icons, empty state icons, notification icons
- [ ] **Clipboard copy works on HTTP (non-secure):** Test on actual staff phone over LAN, not localhost
- [ ] **Dark mode persists:** Toggle dark mode, refresh page, verify state persists
- [ ] **Filter + export works:** Filter by status, export CSV, verify CSV contents match filtered view
- [ ] **Detail modal shows all fields:** Test with each equipment type (DJI, Car, Other)
- [ ] **Unseen badge works:** Approve a request, reload, verify badge shows correct count

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Firebase listener leak | LOW | Add cleanup function to the leaky `useEffect`. Remove duplicate listeners by reloading page. |
| Babel standalone module error | LOW | Remove `import`/`export` statements. Revert to single `<script type="text/babel">` block. |
| Passcode gate broken | MEDIUM | Restore the atomic state transition in parent component. Verify `isManagerAuthenticated` is only set in parent. |
| Behavior drift in form payload | HIGH | Compare current Firestore writes against expected payload structure. Fix and manually correct any wrong entries in Firestore. |
| Icons not rendering | LOW | Add `lucide.createIcons()` call after render. Verify all `<i data-lucide>` elements are processed. |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Firebase listener leaks | Phase 1: Extract Components | Console.log cleanup functions; verify single WebSocket in Network tab |
| Babel module incompatibility | Phase 1: Extract Components | Zero `import`/`export` statements; app loads without console errors |
| Passcode gate state coupling | Phase 2: State Management | Manual test: wrong code → clear, correct code → unlock, Enter key → submit |
| Borrow/return behavioral drift | Phase 1: Extract Components | Submit real request → approve → pass → return on staff phone |
| Lucide icon timing | Phase 1: Extract Components | Visual check: all icons render in all views |
| Script block ordering | Phase 1: Extract Components | Each script block logs on load; verify sequential execution |
| Dark mode CSS toggling | Phase 2: State Management | Toggle dark mode, refresh, verify persistence |
| seenRequestIds race | Phase 2: State Management | Submit request, verify it appears as "unseen" (amber left border) |
| Form state reset (platNumber) | Phase 1: Extract Components | Submit vehicle request, then submit DJI request — no plat number shown |
| Shared approverName | Phase 1: Extract Components | Two pending cards show same approver selection (preserve existing behavior) |

## Sources

- Direct analysis of `index.html` (532 lines, 14 useState hooks, 5 useEffect hooks)
- React 18 + Babel standalone compatibility documentation
- Firebase compat SDK documentation (v11.6.1)
- Lucide CDN integration patterns
- Common pitfalls in React component decomposition (experience-based)
- KWM Logistics deployment context: `http://10.0.6.12:8080` (non-secure, LAN)

---
*Pitfalls research for: single-file CDN React + Firebase equipment log app decomposition*
*Researched: 2026-09-11*
