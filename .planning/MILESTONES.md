# Milestones

## v1.0 MVP (Shipped: 2026-09-15)

**Phases completed:** 3 phases, 6 plans, 14 tasks

**Closeout type:** override_closeout — Phases 1 & 3 lack formal `*-VERIFICATION.md` reports (user-tested via UAT instead); 5 requirements (REF-01..05) marked complete by override.

**Key accomplishments:**

- Manager unlock now validates against the `DEFAULT_PASSCODE` HTML constant with the Firestore passcode read/write path deleted and a rotate-by-editing comment added — zero Firestore passcode operations remain in the app
- PROJECT.md, FEATURES.md, and ARCHITECTURE.md rewritten to describe the constant-based passcode (`DEFAULT_PASSCODE` in index.html Block 2, edit to rotate) — zero Firestore passcode-flow claims remain in the planning docs
- Config-driven plat and approver dropdowns with "Other" free-text reveal — PLAT_OPTIONS/APPROVER_OPTIONS constants in Block 2, mapped selects at all three sites, typed custom values recorded via finalPlat/finalApprover

**Known verification overrides:** 2 (Phases 1 & 3 — see STATE.md Deferred Items)

---
