---
phase: 06-form-tweaks-and-borrower-return-flow
plan: 02
subsystem: ui
tags: [vehicle, fuel-bar, settings-lock, form]

dependency:
  requires: [01]
  provides: [VEH-01, VEH-02, VEH-03, VEH-04, VEH-05]
  affects: []

tech-stack:
  added: []
  patterns: [react-conditional-rendering, locked-list-items, required-select]

key-files:
  created: []
  modified:
    - index.html

decisions:
  - "Vehicle plates removed from the borrow form — plate is embedded in the equipment name (e.g., 'WRD 5900 (Myvi)')."
  - "Email input hidden when a vehicle equipment option is selected (WRD 5900 (Myvi), NDC 264 (Motor), Vehicle)."
  - "Fuel bar labels are exactly '1 Bar', '2 Bar', '3 Bar', '4 Bar', 'Full'."
  - "Vehicle equipment items locked in Settings (no reorder/edit); deletable via passcode modal showing red warning: '!! Warning: if this item is deleted, the form will revert back to the default and the fuel bar status can no longer be used. !!'"
  - "Return modal for vehicles requires an active fuel bar choice — CONFIRM RETURN disabled until a bar is selected; cancelling writes nothing."

metrics:
  duration: 45m
  completed_date: 2026-10-13
  status: complete
  plan_head_before: f1bc125
  commits: 1

actuals:
  tokens: 12000
  tasks: 5
  commits: 1
---

# Phase 06 Plan 02: Vehicle Fuel-Bar Mode Summary

Vehicle fuel-bar mode: plates removed, email hidden for vehicles, exact fuel labels, locked vehicle items in Settings, and required fuel-bar choice on return.

## Performance Metrics

- **Duration:** 45m
- **Tasks Completed:** 5/5
- **Commits:** 1 (`fcd9b36`)

## Accomplishments

- **VEH-01**: Removed vehicle plates from the borrow form — the plate is embedded in the equipment name (e.g., `WRD 5900 (Myvi)`), so no separate plate field renders.
- **VEH-02**: Email input hidden when a vehicle equipment option is selected (`WRD 5900 (Myvi)`, `NDC 264 (Motor)`, `Vehicle`); non-vehicle equipment keeps the email field.
- **VEH-03**: Fuel bar labels exactly `1 Bar / 2 Bar / 3 Bar / 4 Bar / Full` in both the borrow form and the return modal.
- **VEH-04**: Vehicle equipment items locked in Settings — no reorder/edit; deletable only via the passcode modal, which shows the red warning about reverting to defaults and losing fuel-bar status.
- **VEH-05**: Return modal for vehicles requires an active fuel bar choice — CONFIRM RETURN disabled until a bar is selected; cancelling the modal writes no return.

## Deviations from Plan

None - plan executed exactly as written.

## Self-Check: PASSED

- `index.html` modified successfully.
- Commit `fcd9b36` verified in git log.
- Full E2E UAT passed 15/15 (see 06-VERIFICATION.md).