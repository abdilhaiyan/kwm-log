---
phase: 4
slug: sortable-list-reordering
# status lifecycle: draft (seeded by plan-phase) → validated (set by validate-phase §6)
# audit-milestone §5.5 distinguishes NOT-VALIDATED (draft) from PARTIAL (validated + nyquist_compliant: false) (#2117)
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-09-16
---

# Phase 4 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | none — this repo has no package.json / test runner (no-build single-file app; RESEARCH Validation Architecture § NG: manual + console is the architecture) |
| **Config file** | none — verification is per-task `node -e` regex sweeps against `index.html` + browser DevTools assertions on the served app (localhost:8080, live Firestore data per RESEARCH A1) |
| **Quick run command** | the `<automated>` node -e sweeps embedded in 04-01-PLAN.md tasks (Task 1: 2 sweeps; Task 2: 2 sweeps) — run after each task commit |
| **Full suite command** | Wave-0 DevTools scaffold (Pitfall-1 icon repro + live select-order baseline) + S1–S6 acceptance + regression UAT list (STACK.md) at end of phase |
| **Estimated runtime** | ~5 seconds per node -e sweep (single-file read); ~5–10 min for the browser/UAT pass |

---

## Sampling Rate

- **After every task commit:** Run that task's `<automated>` node -e sweep(s) against `index.html` — each exits 1 on assertion failure, matching the task `<done>` gate.
- **After every plan wave (execution of 04-01-PLAN):** Run the Wave-0 DevTools scaffold once (Settings mount icon assertion `i[data-lucide]` count 0; select-order baseline of the three consumed selects against LIVE doc data).
- **Before `/gsd-verify-work`:** All node -e sweeps green; DevTools matrix assertion (otherUp/otherDown ≥ 3, disabled booleans true) green; S1–S6 acceptance pass in browser.
- **Max feedback latency:** ~5 seconds per automated sweep; the DevTools/reload checks are the long pole at end of phase.

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Threat Ref | Secure Behavior | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|------------|-----------------|-----------|-------------------|-------------|--------|
| 04-01-1 | 01 | 1 | SORT-01, SORT-02, SORT-03, SORT-04, SORT-05 | T-04-01 | moveOption bound-guards `toIdx` (no out-of-bounds write — `if (toIdx < 0 \|\| toIdx >= arr.length) return;`), never setState's the option arrays (Firestore-echo only), addOption inserts before `'Other'` (Q2) | node -e regex sweep | 2 commands: (1) signature/guard/insert-before/const-last counts === 1 each; (2) zero `set(Plat\|Approver\|Equipment)Options` in the moveOption body slice + `saveOptions(next.plat, next.approver, next.equipment);` as final statement | ✅ index.html | ⬜ pending |
| 04-01-2 | 01 | 1 | SORT-01..06, UI E1–E5 | T-04-01 | Down disabled at true last index (`i >= items.length - 1`, incl. legacy non-Other tail) and above `'Other'`; across both buttons the amended boundary matrix holds; no arrows in edit mode (Pitfall 5) | node -e regex sweep + browser DevTools | 2 commands: (1) onMoveUp/onMoveDown == 5, ChevronUp/ChevronDown == 1, createIcons == 3, Reorder legend 1× & before Edit; (2) `i >= items.length - 1` == 3, `items[i+1] === 'Other'` == 2, `disabled={i === 0 \|\| o === 'Other'}` == 1. DevTools: `i[data-lucide]` count 0 + disabled matrix booleans | ✅ index.html | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] (Task 2 verify, DevTools on served page) Pitfall-1 repro: after opening Settings, `[...document.querySelectorAll('i[data-lucide]')]` must be `[]` — was 18 unmaterialized before this phase.
- [ ] (Task 2 verify, live data) DevTools disabled-matrix assertion: `otherUp >= 3`, `otherDown >= 3`, and `otherUpDisabled`/`otherDownDisabled`/`firstUpDisabled`/`bottomDownDisabled` all true (amended boundary rules incl. Q1 last-index clause).
- [ ] (pre-move UAT) Select-order baseline: record equal text of the request-form equipment select, manager filter select, and both approver selects from LIVE doc data (never DEFAULT_* — RESEARCH A1).

*Existing node -e + DevTools infrastructure covers all phase requirements; no test framework install is possible or wanted in this no-build repo.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| S1–S3 live reorder: click Up/Down → request-form equipment/plat selects and both approver selects re-render immediately (no reload) | SORT-01/02/03 | Needs a rendered browser + live Firestore doc; no DOM test framework exists | Settings → Equipment/Vehicle Plates/Approvers card → click an enabled arrow → check the consuming select(s) show the swapped order |
| S4 Other pin: immovable (both arrows disabled, tooltip Cannot reorder Other); Q4 legacy fix (Equipment Laptop ▲ once → `['DJI Osmo Action 6','Car','Laptop','Other']`); Q2 add lands above Other | SORT-04 | Live-data + visual tooltip verification; the legacy tail is a one-time manual repair, not load-time normalization (Q4) | Per-list inspect the Other row; Equipment: move Laptop up once; add a new item and confirm it lands above Other; every select re-renders Other last |
| S5 persistence: reload → same order; second tab/browser → same order; Network tab shows exactly one options/lists write per move | SORT-05 | Cross-device/cross-tab + Network inspection | After moves, reload, open second tab, compare select orders; watch Network for one write per move |
| S6 reload consistency + no localStorage order state | SORT-06 | localStorage inspection is manual console | After reload compare orders; confirm only `kwm_dark_mode` / `kwm_seen_requests` in localStorage |
| Regression UAT (submit request, manager passcode unlock, approve w/ approver select, reject, filter, CSV export, copy details, dark toggle, return + item-passed flows) | Regression (STACK.md) | Behavior-preservation check by human | Exercise each flow exactly as in STACK.md; all must behave as before the phase |
| Mobile viewport: four-button row fits with truncating label, disabled arrows read faded vs enabled glass, p-2 tap targets | UI E1/E2 (human-check) | Visual/UX judgment | Phone viewport or responsive preview in Settings; confirm layout + legend reads Reorder, Edit, Delete, Save, Cancel, ADD |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 60s
- [ ] `nyquist_compliant: true` set in frontmatter (by validate-phase §6)

**Approval:** pending