---
phase: 02-config-driven-passcode
reviewed: 2026-09-14T04:30:00Z
depth: standard
files_reviewed: 1
files_reviewed_list:
  - index.html
findings:
  critical: 0
  warning: 3
  info: 3
  total: 6
status: issues_found
---

# Phase 2: Code Review Report

**Reviewed:** 2026-09-14T04:30:00Z
**Depth:** standard
**Files Reviewed:** 1
**Status:** issues_found

## Summary

Reviewed the current state of `index.html` (525 lines, 7 `text/babel` blocks) against the Phase 2 plan (02-01-PLAN.md) and its threat register. The three surgical edits are executed exactly as planned and verified:

- **Correct**: `app_config` fetch effect, rotation `.set()` write, `currentPasscode`/`newPasscode` state, rotation input, and "Passcode updated" toast are all gone — zero occurrences of `app_config` / `currentPasscode` / `newPasscode` / "Passcode updated" / "New passcode (optional)" remain in `index.html` (verified by grep; the diff `e56c294..HEAD` is 2 insertions / 18 deletions, all confined to the passcode path).
- **Correct**: Unlock validates `passcodeAttempt === DEFAULT_PASSCODE` (block-scope `const` from Block 2, shared global scope — strict string compare, no coercion). `data-passcode-unlock`, the Enter-key `querySelector(...).click()` pattern, the wrong-passcode silent clear, `handleNotify`, and all other effects are preserved byte-identical. 7 blocks intact.
- **Correct**: The `onSnapshot` listener (line 149) is the only remaining Firestore read.

The passcode-path *implementation* is sound. The findings below are about (a) a security-behavior regression the phase's own threat register does not enumerate, (b) project-standard violations that remain in the file, and (c) a documentation drift the phase's D-03 scope deliberately left behind. No BLOCKERs: the plaintext constant is an accepted, documented risk per the plan, and the manager gate remains a UI gate (Firestore rules are unchanged).

## Warnings

### WR-01: Silent passcode downgrade to `"1234"` if the leftover `app_config` doc holds a rotated value — new surface not in the threat register

**File:** `index.html:420` (interacts with leftover Firestore data, not a code defect per se)

**Issue:** Before this phase, the effective gate was `currentPasscode`, initialized to `DEFAULT_PASSCODE` but **overwritten by `doc.data().passcode` when the `app_config` doc had the field**. Any manager who ever used the old "New passcode" rotation UI (which wrote that field) left a rotated value ≠ `"1234"` in the live Firestore doc. This phase deletes the fetch and validates only `passcodeAttempt === DEFAULT_PASSCODE`, while D-02 deliberately leaves the doc untouched. Net effect: **for any installation whose `app_config` doc contains a rotated passcode, the deployed gate silently reverts to the publicly documented default `"1234"`** — anyone who reads this repo or the page source can unlock, regardless of the operator's rotation. The threat register (T-02-01 / T-02-03) asserts "the value was already readable as the Firestore fallback before this phase" — but the fallback only applied when the doc had *no* passcode field; it never applied to a rotated doc. The register does not enumerate the rotated-doc scenario, so this is new surface beyond the accepted-risk list.

**Fix:** One-time ops check at rollout (not code): read `artifacts/camera-borrow-wiramas/public/data/app_config` in the Firestore console and either delete the doc, delete the `passcode` field, or set it to `"1234"`, so the effective gate matches the deployed source. Optionally add this step to the phase's handoff/verify docs so the dead-data state can never silently override operator intent during a later re-deploy.

### WR-02: Unpinned CDN ranges remain in the file, violating the project's own STACK.md "What NOT to Use" rule

**File:** `index.html:11-14`

**Issue:** Lines 11-14 load `react@18`, `react-dom@18`, bare `@babel/standalone/babel.min.js`, and `lucide@latest`. AGENTS.md / `.planning/research/STACK.md` line 81 explicitly prohibits these exact ranges ("Unpinned ranges drift when a browser/CDN cache is busted — the app's render output can change under you"), and the recommended layout pins `react@18.3.1`, `react-dom@18.3.1`, `@babel/standalone@8.0.5`, `lucide@1.44.0` (STACK.md lines 74-77, HEAD-verified 200). `lucide@latest` is the highest-risk offender: a lucide release changes icon output and could break the `lucide.createIcons()` / `lucide.icons[name]` contract this app depends on (the `Icon` component and the D-04 100ms re-materialization hack). Under the phase's own "behavior-preserving changes, verified by reload" standard, an unpinned icon library can silently change render output after the reviewer's verification passes. Pre-existing, but still present in the reviewed file and contradicted by the project's own written standard.

**Fix:** Pin the four URLs to the exact versions in STACK.md:
```html
<script src="https://unpkg.com/react@18.3.1/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/@babel/standalone@8.0.5/babel.min.js"></script>
<script src="https://unpkg.com/lucide@1.44.0/dist/umd/lucide.min.js"></script>
```

### WR-03: AGENTS.md (the project instruction file every agent reads) still describes the passcode as "Firestore-backed, changeable"

**File:** `AGENTS.md:16` (GSD:project-start block; the code file itself is unaffected)

**Issue:** The phase's D-03 scope updated `.planning/PROJECT.md` — line 65 now correctly reads "gated by the passcode constant in HTML; rotate by editing `DEFAULT_PASSCODE`" — but `AGENTS.md` embeds a snapshot of PROJECT.md (`<!-- GSD:project-start source:PROJECT.md -->`) that was **not regenerated**: line 16 still says "Manager actions gated by passcode (Firestore-backed, changeable)". This is the primary instruction file for every future agent and this review. It now actively contradicts implemented behavior (`PASS-02`, which removed all Firestore passcode logic). Any agent planning future passcode work from AGENTS.md will act on a false security premise. The refactor milestone's next phase reads this file before every code change.

**Fix:** Regenerate/update the `GSD:project-start` block in `AGENTS.md` to match the new `.planning/PROJECT.md` line 65 phrasing (one-line edit), or add `AGENTS.md` regeneration to the next docs plan so the instruction file tracks the source of truth.

## Info

### IN-01: Dead code — `getEquipmentIcon` defined, never called

**File:** `index.html:82`

**Issue:** `const getEquipmentIcon = (r) => ...` is extracted as a "utility" (Block 4) but has no call site anywhere in the file (grepped; all planning-doc references are prose). UI status/equipment icons are hardcoded lucide names elsewhere. Dead export surviving the Phase 1 extraction.

**Fix:** Either delete the function, or use it as intended, e.g. `<Icon name={getEquipmentIcon(req)} />` in the request card where the equipment type is displayed.

### IN-02: `getDaysInfo` renders "NaN days left" for a malformed `returnDate`

**File:** `index.html:68-79`

**Issue:** If a Firestore record carries a malformed `returnDate` string (hand-edited data, legacy import), `new Date(r.returnDate)` is `Invalid Date`; `(rd - t)` is `NaN`; all three branches fail (`NaN < 0`, `NaN === 0`), and line 79 returns `"NaN days left"` into the card UI. `isOverdue` degrades safely to `false` for the same input, so only the label is wrong. Data-dependent edge case, pre-existing, not introduced by this phase; low probability given the form constrains `type="date"`.

**Fix:** Guard with a validity check before arithmetic:
```javascript
const rd = new Date(r.returnDate);
if (isNaN(rd.getTime())) return null;
```

### IN-03: Manager-view icons rely on the `onSnapshot` `createIcons` call — a snapshot after unlock never comes

**File:** `index.html:149` (single `lucide.createIcons()` call site, inside the `onSnapshot` callback)

**Issue:** The `Icon` component renders `<i data-lucide>` placeholders; SVGs only materialize when `lucide.createIcons()` re-scans the DOM. That scan happens only inside the `onSnapshot` effect. After a manager unlocks (view switches to `manager`), the manager-view icons (Download, Inbox, etc.) are newly committed `<i data-lucide>` nodes that no snapshot re-render will re-scan until *new* request data arrives — so the Export button / empty-state icons can remain blank for the session. Not introduced by this phase (the unlock handler's success branch was already missing a `createIcons()` call before Phase 2), and Phase 1's locked D-04 explicitly preserved the hack — flagging as a known-remaining defect for a future phase, e.g. adding `setTimeout(() => lucide.createIcons(), 0)` after `setView(...)` in the unlock handler.

**Fix (future phase):** `setIsManagerAuthenticated(true); ...; setTimeout(() => lucide.createIcons(), 100);` inside the unlock success branch, mirroring the D-04 pattern.

---

_Reviewed: 2026-09-14T04:30:00Z_
_Reviewer: the agent (gsd-code-reviewer)_
_Depth: standard_