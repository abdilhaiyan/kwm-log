---
status: complete
phase: 02-config-driven-passcode
source: [02-01-SUMMARY.md, 02-02-SUMMARY.md]
started: 2026-09-14T12:35:00Z
updated: 2026-09-14T12:50:00Z
---

## Current Test

[testing complete]

## Tests

### 1. Manager unlock with correct passcode
expected: Type 1234 into the passcode modal and click Unlock. Manager view unlocks: modal closes, view switches to manager, requests marked seen.
result: pass

### 2. Wrong passcode rejected silently
expected: Type a wrong value and submit. Input clears silently (no toast), modal stays open, manager view does not unlock.
result: pass

### 3. Single password field in modal
expected: Open the modal from the Manager tab. Exactly one password field, Unlock button, Cancel button, same glassmorphic chrome as before.
result: pass

### 4. Enter key submits unlock
expected: Type 1234 and press Enter. Same unlock behavior as clicking Unlock.
result: pass

### 5. App regression — request list, dark mode, form
expected: Request list renders, dark-mode toggle works, submit form works identically; no console errors.
result: pass

### 6. WR-01 Firestore ops check (deployment-time)
expected: Inspect the live Firestore doc artifacts/camera-borrow-wiramas/public/data/app_config. Doc deleted, or its passcode field deleted, or set to "1234" — so the effective gate matches the deployed DEFAULT_PASSCODE constant.
result: skipped
reason: "Deferred follow-up: skip, check back later"

## Summary

total: 6
passed: 5
issues: 0
pending: 0
skipped: 1

## Gaps

[none yet]

## Deferred Follow-Ups

- test: 6
  idea: "skip, check back later"
  deferred_at: 2026-09-14