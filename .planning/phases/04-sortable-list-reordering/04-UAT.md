---
status: complete
phase: 04-sortable-list-reordering
source: [04-VERIFICATION.md]
started: 2026-09-16T17:35:00Z
updated: 2026-09-17

## Current Test

[testing complete]

## Tests

### 1. Mobile Visual/UX Confirmation
expected: Open Settings on a phone viewport (or responsive preview in DevTools). The four-button row fits mobile width; the truncating label clips long names; disabled arrows read as faded (bg-white/5) vs enabled glass arrows; hover/tap targets feel consistent (p-2, ~30x30px); the Settings legend reads Reorder, Edit, Delete, Save, Cancel, ADD with the neutral dot for Reorder.
result: pass

### 2. S1 Equipment Reorder Live (SORT-01)
expected: Settings → Equipment List → click Up on a non-first row → the request form's equipment select shows the swapped order immediately (no reload).
result: issue
reported: "The user hard to know which ones is up and which one is down. Is a little up arrow and down arrow is okay to make it much friendly and easier for the user?"
severity: minor

### 3. S2 Plat Reorder Live (SORT-02)
expected: Vehicle Plates card → move a row → request form plat select (form with cameraModel = Car) reflects the same order as the Settings card.
result: issue
reported: "The user hard to know which ones is up and which one is down at the card. Is a little up arrow and down arrow is okay to make it much friendly and easier for the user?"
severity: minor

### 4. S3 Approver Reorder Live (SORT-03)
expected: Approvers card → move a row → manager card approver select AND detail-modal approver select both reflect the new order.
result: issue
reported: "The user hard to know which ones is up and which one is down in the card. Is a little up arrow and down arrow is okay to make it much friendly and easier for the user?"
severity: minor

### 5. S4 Other Pin + Q4 Legacy Fix + Q2 Add Observation (SORT-04)
expected: For EVERY list: Other row has both arrows disabled with tooltip "Cannot reorder Other". In Equipment: click Laptop Up once → order becomes ['DJI Osmo Action 6','Car','Laptop','Other']. ADD a new equipment item → it appears ABOVE Other. New item lands above Other (Q2 observable); every select re-renders with Other last.
result: pass

### 6. S5 Persistence + Cross-Tab (SORT-05)
expected: After the moves in S1-S4, reload the page → order persists. Open a second tab/browser → same saved order. Network tab → exactly one options/lists write per move.
result: pass

### 7. S6 Reload Consistency + No localStorage Order (SORT-06)
expected: After S5 reload, order is identical on both surfaces. DevTools console: Object.keys(localStorage).filter(k => k.startsWith('kwm_')) shows only kwm_dark_mode and kwm_seen_requests; no option-order keys.
result: pass

### 8. Regression UAT (STACK.md)
expected: Submit request form, manager passcode unlock, approve with approver select, reject, filter, CSV export, copy details, dark-mode toggle, return flow (mark returned + condition), item-passed flow. All behave exactly as before the phase — zero new error states, zero layout regressions.
result: pass

### 9. DevTools Icon Assertion (Pitfall 1 Reproduction)
expected: Open Settings → DevTools console → [...document.querySelectorAll('i[data-lucide]')] → navigate back to Form, open Settings again, re-run. Both runs return []; all icons materialized to svg.lucide-*.
result: pass

### 10. DevTools Disabled-Matrix Assertion
expected: Open Settings → DevTools console → evaluate the matrix assertion from PLAN Task 2 verify: icons === 0, otherUp >= 3, otherDown >= 3, otherUpDisabled === true, otherDownDisabled === true, firstUpDisabled === true, bottomDownDisabled === true.
result: pass

### 11. Post-Fix Re-verification (9bfede4, 2026-09-17)
expected: After the Icon rewrite + settings-button restyle (commit 9bfede4), re-run the interactive row operations against live Firestore — all reversible: (a) reorder Down then back Up on an Equipment row (order restored), (b) Edit → save a temporary label then restore it, (c) ADD a temp item via the "+" circle, (d) delete the temp item via the list-delete modal (passcode 1234 → CONFIRM DELETE), (e) confirm zero residue in every consuming select and the list returns to its exact pre-test state.
result: pass

## Summary

total: 11
passed: 8
issues: 3
pending: 0
skipped: 0

## Gaps

- gap_id: G-04-2
  truth: "Settings reorder controls clearly distinguish Up from Down with friendly icons"
  status: resolved
  resolved_by: 04-02-PLAN.md (04-02-SUMMARY.md correction: deployed fix is 9bfede4)
  resolved_at: 2026-09-17
  reason: "User reported: The user hard to know which ones is up and which one is down. Is a little up arrow and down arrow is okay to make it much friendly and easier for the user?"
  severity: minor
  test: 2
  artifacts: []
  missing: []
  also_reported_on: [3, 4]
  root_cause: "Initial diagnosis (036594f): index.html lines 117-118 render ChevronUp/ChevronDown at size={14} with identical styling (bg-white/20 text-white/80) - no size/color/direction distinction and no tooltip on enabled buttons, so the affordance relies solely on tiny chevron orientation. DEEPER ROOT CAUSE (refined at 9bfede4, confirmed live after a full reload): the Icon component called name.toLowerCase() when setting the data-lucide attribute, producing unresolvable keys like 'arrowup'/'chevronup'; Lucide only resolves PascalCase or kebab-case names, so lucide.createIcons() could never materialize those icon nodes - rendered icons were window-blind/blank regardless of glyph name, and createIcons() imperatively replacing React-owned <i> nodes with <svg> also broke React reconciliation (edit-mode Save/Cancel would show stale arrow glyphs)."
  fix_applied: "04-02 (e67a7cc): ChevronUp/ChevronDown size 14 -> ArrowUp/ArrowDown size 16, lines 117-118 only - NECESSARY but NOT sufficient. Real fix (9bfede4): rewrote the Icon component to render a React-owned <svg> directly from lucide.icons[name] node data (flat [[tag, attrs], ...] pairs) with a kebab-case fallback for unknown names; createIcons() call sites remain but are now no-ops; the same commit applied the user-approved restyle (24px circular row buttons, Save=Check/Cancel=X, edit/delete glyphs, ADD+= circle, compact inputs, legend circle). Re-verified live in test 11."