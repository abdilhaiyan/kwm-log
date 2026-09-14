# Phase 2: Config-Driven Passcode - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-09-14
**Phase:** 2-Config-Driven Passcode
**Areas discussed:** Rotation annotation, Firestore app_config doc, Stale research docs

---

## Rotation Annotation

| Option | Description | Selected |
|--------|-------------|----------|
| Add comment (Recommended) | Add `// Manager passcode — edit to rotate` above `DEFAULT_PASSCODE` in Block 2 (Config). Makes rotation discoverable for anyone opening index.html. Zero runtime impact. | ✓ |
| No comment | Leave `const DEFAULT_PASSCODE = "1234";` exactly as-is with no comment. Minimal diff, but rotation is undiscoverable without reading docs. | |

**User's choice:** Add comment (Recommended)
**Notes:** None — straightforward selection.

---

## Firestore app_config Doc

| Option | Description | Selected |
|--------|-------------|----------|
| Leave untouched (Recommended) | Don't touch the existing `app_config` doc in Firestore. It becomes harmless dead data — the app never reads it after this phase. Zero risk of deleting something in use. | ✓ |
| Leave + note cleanup | Note in the phase summary that the `app_config` doc can be manually deleted from the Firestore console after rollout — but don't delete it programmatically. | |
| Delete the doc | Delete the `app_config` doc as part of this phase (via Firestore console or script). Riskier — if any other app/process reads it, that breaks. | |

**User's choice:** Leave untouched (Recommended)
**Notes:** None — straightforward selection.

---

## Stale Research Docs

| Option | Description | Selected |
|--------|-------------|----------|
| Update in this phase (Recommended) | Update PROJECT.md (Firebase section), FEATURES.md (passcode feature row), and ARCHITECTURE.md (passcode flow) to reflect constant-based passcode. Keeps planning docs truthful for future phases. | ✓ |
| Defer to later cleanup | Skip doc updates now; note them as a deferred item for a later docs-cleanup pass. Code changes land first, docs stay stale until then. | |

**User's choice:** Update in this phase (Recommended)
**Notes:** None — straightforward selection.

---

## the agent's Discretion

- Wrong-passcode behavior: keep exactly as-is (`setPasscodeAttempt("")` — input clears, no toast). Locked by ROADMAP success criterion #2.
- `currentPasscode` state: remove entirely; validate `passcodeAttempt === DEFAULT_PASSCODE` directly.
- `newPasscode` state: remove entirely (control is gone).
- "Passcode updated" toast: removed with the control (no longer reachable).
- Verification: programmatic checks + real-browser UAT at `http://10.0.6.12:8080`.

## Deferred Ideas

None — discussion stayed within phase scope.