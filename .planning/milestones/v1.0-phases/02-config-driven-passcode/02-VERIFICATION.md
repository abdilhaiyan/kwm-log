---
phase: 02-config-driven-passcode
verified: 2026-09-14T12:22:31Z
status: passed
score: 2/5 must-haves verified
behavior_unverified: 3 # SC-1, SC-2, SC-5: structure present + wired, runtime behavior/visual identity unexercised (no test runner; Babel cannot run in preview env — browser UAT required)
overrides_applied: 0
re_verification:
  previous_status: null
  previous_score: null
  gaps_closed: []
  gaps_remaining: []
  regressions: []

# No `gaps` key: zero failed truths / missing artifacts / broken key links were found.

# Blocked from `passed` by: (a) MVP goal-format guard — ROADMAP goal is not a user story, (b) runtime behaviors awaiting browser UAT.

behavior_unverified_items:

  - truth: "SC-1: Manager unlocks manager actions by entering the passcode from the HTML constant (default 1234) — same modal, same Enter-key submit"
    test: "Serve the app and type 1234 into the passcode modal, then click Unlock (and again via Enter key)"
    expected: "Manager view unlocks: isManagerAuthenticated true, modal closes, view switches to manager, requests marked seen"
    why_human: "Unlock is a runtime state transition; the handler is present and wired (passcodeAttempt === DEFAULT_PASSCODE → set isManagerAuthenticated → setView('manager')) but no test runner exists and Babel standalone cannot execute in the preview environment"

  - truth: "SC-2: Entering a wrong passcode is rejected with the same error behavior and does not unlock"
    test: "Type a wrong value and submit; then type a wrong value and press Enter"
    expected: "Input clears (setPasscodeAttempt(\"\")), no toast, modal stays open, manager view does not unlock"
    why_human: "Wrong-branch clear is a runtime branch; the else branch is byte-identical (silent clear, no handleNotify) but runtime rejection cannot be exercised without a browser"

  - truth: "SC-5: The passcode modal looks and behaves identically (input, Unlock button, Enter submit, error state)"
    test: "Open the modal from the Manager tab and visually compare to pre-phase appearance"
    expected: "Exactly one password field, Unlock button with data-passcode-unlock, Cancel button, same glassmorphic chrome, Enter submits; wrong input clears silently"
    why_human: "Modal chrome and single-input structure are present byte-identical, but visual identity and look-and-feel require human eyes in a real browser"
human_verification:

  - test: "MVP goal-format decision: ROADMAP.md Phase 2 has mode: mvp but its goal is NOT a user story (user-story.validate → false). Decide how to proceed per the guard"
    expected: "Either run /gsd mvp-phase 02 to set a proper User Story goal (candidate: 'As a manager, I want to configure the passcode by editing a constant, so that the approval gate stays under operator control without a Firestore round-trip.') and re-verify with a User Flow Coverage section, or explicitly direct standard (non-MVP) verification against the ROADMAP goal + 5 Success Criteria"
    why_human: "The MVP-mode user-story-format guard requires a human decision; the verifier must not fabricate user-flow coverage against a goal that is not in the ROADMAP"

  - test: "Browser UAT (plan 02-01 Task 1 human-check): serve `python -m http.server 8080` and open http://10.0.6.12:8080"
    expected: "Correct passcode 1234 unlocks the manager view; wrong passcode clears the input with no toast; the modal shows exactly ONE password field; Unlock button AND Enter key both work; request list, dark-mode toggle, form submit render identically; no console errors"
    why_human: "Babel standalone cannot run in the preview environment (Phase 1 finding); runtime behavior requires a real browser"

  - test: "WR-01 ops check: after deploying this phase, read the live Firestore doc artifacts/camera-borrow-wiramas/public/data/app_config in the Firestore console"
    expected: "Either the doc has no passcode field, its passcode field equals 1234, or the doc is deleted — so the effective gate matches the deployed DEFAULT_PASSCODE constant and no installation silently reverts to 1234 against operator intent"
    why_human: "Requires Firestore console access and operator judgment; D-02 deliberately leaves the doc untouched, so the doc state is an ops decision, not code"
---

# Phase 2: Config-Driven Passcode — Verification Report

**Phase Goal (as recorded in ROADMAP.md):** "Manager unlock works from a passcode constant in the HTML (`DEFAULT_PASSCODE = "1234"`); Firestore is no longer involved in passcode validation or rotation"
**Verified:** 2026-09-14T12:22:31Z
**Status:** human_needed
**Re-verification:** No — initial verification

## MVP Mode: User Flow Coverage — BLOCKED by goal-format guard

**The phase is marked `mode: mvp` in ROADMAP.md, but its goal is NOT in user-story format.** The MVP-mode guard (`references/verify-mvp-mode.md`) requires a user-story goal ("As a [role], I want to [capability], so that [outcome].") before a User Flow Coverage section can be produced. This report therefore does NOT fabricate user-flow coverage; it surfaces the discrepancy (Escalation Gate) and reports the objective roadmap Success-Criteria evidence instead.

**Facts:**

- `user-story.validate` on the ROADMAP goal → `false` (not a user story: "Manager unlock works from a passcode constant in the HTML..."). Run: `node gsd-tools.cjs query user-story.validate --story "<ROADMAP goal>" --pick valid`
- The user story carried in the verify context — "As a manager, I want to configure the passcode by editing a constant, so that the approval gate stays under operator control without a Firestore round-trip." — validates `true` but appears **nowhere** in `.planning/ROADMAP.md`, `2-CONTEXT.md`, or `2-DISCUSSION-LOG.md`. It is a faithful paraphrase of the phase intent, not a recorded goal.
- Both plans in this phase were executed against the ROADMAP goal text + Success Criteria, so the codebase evidence below is unaffected by the format issue.

**Options for the developer (see Human Verification Required, item 1):**

1. Run `/gsd mvp-phase 02` to set a proper User Story goal, then re-run verification to add the User Flow Coverage section.
2. Direct standard-mode verification against the ROADMAP goal + 5 Success Criteria (evidence below already covers these).

## Goal Achievement

### Observable Truths (ROADMAP Success Criteria — the roadmap contract)

| #   | Truth | Status | Evidence |
| --- | ----- | ------ | -------- |
| SC-1 | Manager unlocks manager actions by entering the passcode from the HTML constant (default "1234") — same modal, same Enter-key submit | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Unlock handler compares `passcodeAttempt === DEFAULT_PASSCODE` (node check PASS). Success branch order preserved: `setIsManagerAuthenticated(true); setShowPasscodeModal(false); setView('manager'); markRequestsAsSeen();`. Enter-key `document.querySelector('[data-passcode-unlock]').click()` intact. State transition itself needs a browser — see Human Verification |
| SC-2 | Entering a wrong passcode is rejected with the same error behavior and does not unlock | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | `else { setPasscodeAttempt(""); }` byte-identical — silent clear, **no** `handleNotify` in the wrong branch (node check PASS), no toast string present. Runtime rejection needs a browser — see Human Verification |
| SC-3 | No `app_config` document reads or writes appear in the network panel during unlock, page reload, or a full session — Firestore passcode logic is gone | ✓ VERIFIED | Identifier `app_config` absent from the entire file (node check PASS). Firestore surface = 3 `db.collection(...)` calls, ALL on `borrowing_requests`: one `onSnapshot` (only read, line ~149), one `.add()` (submitRequest), one `.doc(id).update()` (updateStatus/markAsReturned). **Zero `.get()`, zero `.set()`.** It is structurally impossible for an `app_config` read/write to occur — the string does not exist in the bundle |
| SC-4 | The change-passcode control is removed; rotating the passcode now means editing the `DEFAULT_PASSCODE` constant at the top of index.html | ✓ VERIFIED | "New passcode (optional)" input and "Passcode updated" toast absent (node check PASS). D-01 comment `// Manager passcode — edit to rotate` sits directly above `const DEFAULT_PASSCODE = "1234";` in Block 2 (node check: comment precedes constant, value unchanged) |
| SC-5 | The passcode modal looks and behaves identically (input, Unlock button, Enter submit, error state) | ⚠️ PRESENT_BEHAVIOR_UNVERIFIED | Single password input (autoFocus, `INPUT_CLS`, Enter onKeyDown), Unlock button with `data-passcode-unlock`, Cancel button, "Admin Access" title, glassmorphic chrome all present byte-identical. Exactly 7 `text/babel` blocks (node check PASS). Visual identity + look-and-feel require a real browser — see Human Verification |

**Score:** 2/5 truths verified (3 present, behavior-unverified)

## Required Artifacts

| Artifact | Expected | Status | Details |
| -------- | -------- | ------ | ------- |
| `index.html` | Constant gate + Firestore passcode path removed + modal preserved (plan 02-01) | ✓ VERIFIED | All four plan automated checks exit 0, re-run during verification: removed identifiers absent (`app_config`/`currentPasscode`/`newPasscode`), handler shape correct (`passcodeAttempt === DEFAULT_PASSCODE`, `data-passcode-unlock`, `setPasscodeAttempt(`, no stale toast, no rotation input), 7 blocks intact, comment precedes constant. Diff `e56c294..HEAD` = 2 insertions / 18 deletions confined to the passcode path (per 02-REVIEW.md) |
| `.planning/PROJECT.md` | Passcode described as HTML constant, edit-to-rotate, no Firestore claim (plan 02-02) | ✓ VERIFIED | "Firestore-backed, changeable" / "changeable, stored in Firestore" absent; `DEFAULT_PASSCODE` + "edit to rotate" present (node check PASS). Line 54 notes the dead `app_config` doc per D-02 |
| `.planning/research/FEATURES.md` | Constant-based validation; rotation row replaces change-passcode row (plan 02-02) | ✓ VERIFIED | "Change passcode" / "validated against Firestore" absent; `DEFAULT_PASSCODE` + "Rotation" scenario present (node check PASS). Row 23: "validated against the HTML constant `DEFAULT_PASSCODE = "1234"` (Block 2); no Firestore read, no rotation field" |
| `.planning/research/ARCHITECTURE.md` | No Firestore passcode-flow references; constant is sole source (plan 02-02) | ✓ VERIFIED | `fetchPasscode`/`updatePasscode`/`currentPasscode`/`newPasscode` absent; `DEFAULT_PASSCODE` present; "passcode rotation" phrasing gone (node check PASS). Line 68: PasscodeModal "no rotation control"; line 371: constant read directly by App Shell |

### Key Link Verification

| From | To | Via | Status | Details |
| ---- | -- | -- | ------ | ------- |
| Unlock handler (Block 6) | `DEFAULT_PASSCODE` (Block 2) | `passcodeAttempt === DEFAULT_PASSCODE` in the Unlock button `onClick` | ✓ WIRED | Strict string compare, no coercion, no React state mirror of the passcode |
| Enter key (password input) | Unlock handler | `onKeyDown` → `document.querySelector('[data-passcode-unlock]').click()` | ✓ WIRED | Phase 1 locked decision D-04 preserved byte-identical |
| Unlock success | Manager gate flow | `setIsManagerAuthenticated(true)` → `setShowPasscodeModal(false)` → `setView('manager')` → `markRequestsAsSeen()` | ✓ WIRED | Four statements in documented order |
| Firestore | App state | Single `onSnapshot` on `borrowing_requests` → `setRequests(docs.sort(...))` + `setTimeout(() => lucide.createIcons(), 100)` | ✓ WIRED | Only remaining Firestore read; D-04 100ms icon hack intact |

### Data-Flow Trace (Level 4)

| Artifact | Data Variable | Source | Produces Real Data | Status |
| -------- | ------------- | ------ | ------------------ | ------ |
| Passcode gate (manager view) | `passcodeAttempt` / unlock | Block 2 `DEFAULT_PASSCODE` constant (inline, intended) | Yes — the constant IS the configured data source | ✓ FLOWING |
| Request list | `requests` | `onSnapshot` → `setRequests(docs.sort((a,b) => b.createdAt - a.createdAt))` | Yes — real Firestore documents map `{id, ...data()}` | ✓ FLOWING |
| Notifications | `handleNotify` | Request submit / status update success branches | Yes — retained for non-passcode toasts (9 references) | ✓ FLOWING |

### Behavioral Spot-Checks

| Behavior | Command | Result | Status |
| -------- | ------- | ------ | ------ |
| Removed identifiers absent | `node -e "...['app_config','currentPasscode','newPasscode']..."` | OK removed identifiers absent | ✓ PASS (structural) |
| Handler shape (constant compare, unlock attr, wrong-code reset, no stale toast/input) | `node -e "...handler shape..."` | all expected states confirmed | ✓ PASS (structural) |
| 7 text/babel blocks intact | `node -e "...(s.match(/text\/babel/g)||[]).length..."` | 7 | ✓ PASS (structural) |
| D-01 comment precedes constant | `node -e "...comment-before-constant..."` | OK | ✓ PASS (structural) |
| Firestore surface audit | `node -e "...db.collection / .get / .set / onSnapshot counts..."` | 3×borrowing_requests, 0×get, 0×set, 1×onSnapshot | ✓ PASS (structural) |
| No debt markers in index.html | `node -e "TBD|FIXME|XXX|TODO|PLACEHOLDER|..."` | OK no debt markers | ✓ PASS (structural) |
| Docs negatives (PROJECT/FEATURES/ARCHITECTURE) | `node -e` per-plan one-liners | all OK | ✓ PASS (structural) |
| Browser unlock / wrong-code / visual parity | `python -m http.server` + manual UAT | not run — Babel standalone cannot execute in preview env; requires human browser session | ? SKIP → human |

### Probe Execution

No probes exist for this phase. Plan verification uses single-command `node -e` one-liners (Phase 1 pattern), all re-run and passed during this verification. Section N/A.

### Requirements Coverage

| Requirement | Source Plan | Description | Status | Evidence |
| ----------- | ----------- | ----------- | ------ | -------- |
| PASS-01 | 02-01, 02-02 | Manager passcode stored as a configurable constant in the HTML code (e.g. `const DEFAULT_PASSCODE = "1234"`), not integrated with Firebase | ✓ SATISFIED | `const DEFAULT_PASSCODE = "1234";` in Block 2 (with D-01 rotation comment); `passcodeAttempt === DEFAULT_PASSCODE`; zero Firebase passcode references |
| PASS-02 | 02-01, 02-02 | Remove Firestore passcode fetch and update logic (no `app_config` doc reads/writes for passcode) | ✓ SATISFIED | `app_config` absent from the file; zero `.get()`/`.set()`; only `borrowing_requests` touches Firestore (onSnapshot read, add, update) |
| PASS-03 | 02-01, 02-02 | Keep the passcode modal UI (input + Unlock button + Enter key submit) — just validate against the local constant | ✓ SATISFIED (structural) | Single password input, Unlock (`data-passcode-unlock`), Enter submit, Cancel, chrome preserved byte-identical; 7 blocks intact. Visual parity pending browser UAT (human item) |

### Anti-Patterns & Review Notes

| File | Pattern | Severity | Impact |
| ---- | ------- | -------- | ------ |
| index.html | No debt markers (`TBD`/`FIXME`/`XXX`/`TODO`/`PLACEHOLDER`) — scan clean | — | none |
| index.html | WR-01 — leftover `app_config` doc with a rotated passcode would silently downgrade the gate to `"1234"` (per 02-REVIEW.md) | ⚠️ Warning (pre-existing data state, not a code defect) | Does NOT fail any Phase 2 must-have: the constant gate IS this phase's promised behavior and D-02 documents the dead doc. Ops reconciliation recorded as human item 3 — the code never reads or writes the doc, so the doc cannot affect runtime; only operator expectation can drift |
| index.html:11-14 | WR-02 — unpinned CDN ranges (`react@18`, `lucide@latest`, bare babel), contradicts STACK.md | ℹ️ Info (pre-existing, out of phase scope) | Phase 2 `files_modified` = index.html passcode path only; CDN pinning is a separate refactor |
| AGENTS.md:16 | WR-03 — project instruction file still says "Firestore-backed, changeable" | ℹ️ Info (D-03 scope covered PROJECT.md/FEATURES.md/ARCHITECTURE.md only) | Future agents reading AGENTS.md get a stale security premise; recommend regenerating the `GSD:project-start` block in a later docs plan |
| index.html:82 | IN-01 — `getEquipmentIcon` defined, never called | ℹ️ Info (pre-existing) | Dead utility surviving Phase 1 extraction |
| index.html:68-79 | IN-02 — malformed `returnDate` renders "NaN days left" | ℹ️ Info (pre-existing, data-dependent edge case) | Not introduced by this phase |
| index.html:149 | IN-03 — manager-view icons may not materialize until the next `onSnapshot` (createIcons only inside the listener) | ℹ️ Info (pre-existing; D-04 locked the hack as-is) | Unlock success branch historically lacked createIcons; flagged for a future phase |

### Human Verification Required

1. **MVP goal-format decision** (blocking `passed` under `mode: mvp` — see the User Flow Coverage section above): run `/gsd mvp-phase 02` to set a proper User Story goal, or explicitly direct standard-mode verification. Expected: the ROADMAP goal becomes a valid user story (candidate provided above), or the developer accepts standard-mode verification.
2. **Browser UAT** (plan 02-01 human-check): serve `python -m http.server 8080`, open http://10.0.6.12:8080. Expected: 1234 unlocks the manager view; wrong code clears the input with no toast; one password field; Unlock + Enter both work; request list, dark-mode toggle, form submit identical; no console errors.
3. **WR-01 ops check** (SC-3 follow-up): inspect the live `app_config` doc in the Firestore console. Expected: doc deleted, or its `passcode` field deleted, or set to `"1234"` — so the effective gate matches the deployed constant and no installation silently reverts against operator intent.
4. **SC-1 runtime unlock** (behavior-unverified): type 1234, click Unlock, repeat with Enter. Expected: manager view unlocks, requests marked seen.
5. **SC-2 wrong-passcode rejection** (behavior-unverified): type a wrong value, submit. Expected: input clears silently, no toast, no unlock.
6. **SC-5 visual parity** (behavior-unverified): compare the modal's look and feel to the pre-phase state. Expected: identical glassmorphic chrome, single input, Unlock/Cancel buttons, Enter submit.

### Gaps Summary

**No code gaps were found.** Every structural truth, artifact, key link, and requirement check passes against the actual codebase: the constant gate is implemented exactly as promised (`passcodeAttempt === DEFAULT_PASSCODE`), the Firestore passcode path is fully removed (`app_config` absent; zero `.get()`/`.set()`; only the `borrowing_requests` `onSnapshot` read remains), the rotation control is gone with the D-01 edit-to-rotate comment above the constant, the modal is preserved with byte-identical Enter/Unlock/wrong-code behavior and 7 blocks intact, and all three planning docs (PROJECT.md, FEATURES.md, ARCHITECTURE.md) match the new reality.

The phase is **not marked `passed`** for two reasons, both human-decision items rather than defects:

1. **MVP goal-format guard**: `mode: mvp` is set but the ROADMAP goal is not a user story — the verifier must not fabricate user-flow coverage; the developer must decide the goal format.
2. **Runtime behaviors await a real browser**: the unlock transition, wrong-code rejection, and visual parity (SC-1/2/5) are wired but unexercised — no test runner exists, and Babel standalone cannot run in the preview environment. The end-of-phase browser UAT fulfills these.

WR-01 (leftover rotated `app_config` value) is folded into the human checklist as an ops reconciliation, not a failed must-have: this phase's goal is the constant gate, and the constant gate is what the code delivers.

---

## Developer Decision (2026-09-14)

- **Human item 1 (MVP goal-format guard): RESOLVED** — developer chose **standard-mode verification** against the ROADMAP goal + 5 Success Criteria. The report above already covers all 5 SCs; no User Flow Coverage section required. (Candidate user story recorded for future reference: "As a manager, I want to configure the passcode by editing a constant, so that the approval gate stays under operator control without a Firestore round-trip.")
- **Human items 2/4/5/6 (browser UAT, SC-1/SC-2/SC-5 runtime): SATISFIED** — developer verified the 4-point UAT at the plan 02-01 tracer gate on http://10.0.6.12:8080 (1234 unlocks manager view; wrong code clears silently with no toast; exactly one password field; Unlock button and Enter key both work; no console errors).
- **Human item 3 (WR-01 Firestore ops check): PENDING** — deployment-time task: inspect the live `app_config` doc in the Firestore console; delete the doc or its `passcode` field, or confirm it equals `"1234"`.

---

_Verified: 2026-09-14T12:22:31Z_
_Verifier: the agent (gsd-verifier, Escalation Gate)_
