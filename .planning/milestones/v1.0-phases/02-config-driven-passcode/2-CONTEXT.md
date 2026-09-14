# Phase 2: Config-Driven Passcode - Context

**Gathered:** 2026-09-14
**Status:** Ready for planning

<domain>
## Phase Boundary

Manager unlock validates against the `DEFAULT_PASSCODE` constant in the HTML (Block 2, Config). Firestore `app_config` reads/writes for passcode are removed entirely. The change-passcode control (New passcode input + Firestore write) is removed — rotating the passcode now means editing the `DEFAULT_PASSCODE` constant at the top of index.html. The passcode modal UI (input + Unlock button + Enter-key submit + wrong-passcode behavior) stays visually and behaviorally identical.

</domain>

<decisions>
## Implementation Decisions

### Rotation Annotation
- **D-01:** Add a comment above `DEFAULT_PASSCODE` in Block 2 (Config): `// Manager passcode — edit to rotate`. Makes rotation discoverable for anyone opening index.html. Zero runtime impact.

### Firestore app_config Doc
- **D-02:** Leave the existing `app_config` doc in Firestore untouched. It becomes harmless dead data — the app never reads it after this phase. No programmatic deletion, no cleanup note required in the summary.

### Stale Research Docs
- **D-03:** Update PROJECT.md (Firebase section), FEATURES.md (passcode feature row), and ARCHITECTURE.md (passcode flow) in this phase to reflect the constant-based passcode. Keeps planning docs truthful for future phases.

### the agent's Discretion
- Wrong-passcode behavior: keep exactly as-is (`setPasscodeAttempt("")` — input clears, no toast). Locked by ROADMAP success criterion #2 ("rejected with the same error behavior").
- `currentPasscode` state: remove entirely; validate `passcodeAttempt === DEFAULT_PASSCODE` directly.
- `newPasscode` state: remove entirely (control is gone).
- "Passcode updated" toast: removed with the control (no longer reachable).
- Verification: programmatic checks (no `app_config` refs, modal structure, block integrity) + real-browser UAT at `http://10.0.6.12:8080` (preview env cannot run Babel standalone — confirmed in Phase 1).

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Requirements & Roadmap
- `.planning/REQUIREMENTS.md` §Passcode Configuration — PASS-01, PASS-02, PASS-03 (the phase's requirements)
- `.planning/ROADMAP.md` §Phase 2 — Goal, success criteria (5 items), mode, dependencies

### Phase 1 Context (carried-forward decisions)
- `.planning/phases/01-multi-block-decomposition/1-CONTEXT.md` — locked decisions D-01..D-04 (inline blocks, one commit per change, verify after each change, preserve `data-passcode-unlock` + `lucide.createIcons()` hacks as-is)

### Existing Code Reference
- `index.html` — Block 2 (Config): `DEFAULT_PASSCODE` (line 58); App Shell: `passcodeAttempt`/`showPasscodeModal`/`newPasscode`/`currentPasscode` state (lines 97-98, 115-116); Firestore fetch effect (lines 153-158); passcode modal (lines 420-448)

### Docs to Update (D-03)
- `.planning/PROJECT.md` §Firebase — describes `app_config` doc with changeable passcode
- `.planning/research/FEATURES.md` — "Manager authentication gate" feature row + "Change passcode" validation row
- `.planning/research/ARCHITECTURE.md` — `PasscodeModal` component table, `fetchPasscode`/`updatePasscode` references, `currentPasscode` state row

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `DEFAULT_PASSCODE` constant (Block 2, line 58): already exists — the phase's validation source. Only needs the D-01 comment.
- `data-passcode-unlock` DOM query + Enter-key `onKeyDown` handler (line 426): preserved as-is (locked decision D-04 from Phase 1).
- `handleNotify` (line 160): still used for other toasts; "Passcode updated" toast simply becomes unreachable.

### Established Patterns
- Pure-move discipline from Phase 1: edits are surgical; no unrelated changes.
- One commit per logical change (Phase 1 locked decision D-02).
- Programmatic verification scripts (node, written to temp) + real-browser UAT checklist.

### Integration Points
- Block 2 (Config): `DEFAULT_PASSCODE` — add comment only.
- App Shell (Block 6): remove `newPasscode` + `currentPasscode` state declarations; remove the Firestore fetch `useEffect` (lines 153-158); simplify the Unlock button handler to validate against `DEFAULT_PASSCODE` and drop the Firestore write block.
- Passcode modal (Block 6): remove the "New passcode (optional)" input (line 427).

</code_context>

<specifics>
## Specific Ideas

No specific requirements beyond the decisions above — the phase is a surgical removal + constant validation swap, fully specified by ROADMAP success criteria and PASS-01..03.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 2-Config-Driven Passcode*
*Context gathered: 2026-09-14*