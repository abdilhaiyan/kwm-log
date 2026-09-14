---
phase: 3
phase_name: "Configurable Plat & Approver Dropdowns"
project: "KWM Logistics Equipment Log"
generated: "2026-09-14"
counts:
  decisions: 5
  lessons: 2
  patterns: 3
  surprises: 2
missing_artifacts:
  - "03-VERIFICATION.md"
  - "03-UAT.md"
---

# Phase 3 Learnings: Configurable Plat & Approver Dropdowns

## Decisions

### Option lists as plain string constants in Block 2 with rotation comments
Plat and approver options became simple string arrays (`PLAT_OPTIONS`, `APPROVER_OPTIONS`) placed beside `DEFAULT_PASSCODE` in Block 2 (Config), each with a `// edit to add options` comment and "Other" as the last entry.

**Rationale:** Adding a plate or approver becomes a one-line insert before "Other" with zero JSX or logic changes (success criterion #3). Simple strings rather than value/label objects keep the edit trivial. Co-locating with `DEFAULT_PASSCODE` follows the Phase 2 constant-in-Block-2 pattern.
**Source:** 03-01-PLAN.md (D-01), 03-CONTEXT.md (D-01)

### "Other" reveals a required free-text input (equipment pattern generalization)
Selecting "Other" in either dropdown reveals a required free-text input bound to new `customPlat` / `customApprover` state, mirroring the existing equipment "Other" pattern.

**Rationale:** Reuses a proven interaction already in the codebase (line 236 equipment pattern) instead of inventing a new one; the typed value is what gets recorded, so custom plates/approvers are first-class data.
**Source:** 03-01-PLAN.md (D-02), 03-CONTEXT.md (D-02)

### Recorded-value ternary computed before payload/handler writes
`finalPlat = platNumber === 'Other' ? customPlat : platNumber` and `finalApprover = approverName === 'Other' ? customApprover : approverName` are computed just before the payload construction / `updateStatus` call, feeding `payload.platNumber` and `approvedBy` (with `|| 'N/A'` fallback preserved).

**Rationale:** Keeps the recorded value logic in one visible place per write path; the `|| 'N/A'` fallback semantics are preserved in both approve handlers.
**Source:** 03-01-PLAN.md (D-03), 03-CONTEXT.md (D-03)

### Both selects map their constant array via `.map()`
The plat select (Vehicles only) and both approver selects (card + modal) render options from `PLAT_OPTIONS.map(...)` / `APPROVER_OPTIONS.map(...)` with placeholder first and "Other" last by construction.

**Rationale:** Option order and membership come from the constant array order — no hardcoded `<option>` elements remain in JSX, so the one-line-edit success criterion holds.
**Source:** 03-01-PLAN.md (D-04), 03-CONTEXT.md (D-04)

### Research deliberately skipped; one commit per logical change
The orchestrator skipped research for this phase because 03-CONTEXT.md decisions D-01..D-04 fully specified the implementation. Execution used two atomic commits (Task 1 plat path, Task 2 approver path) per the Phase 1 locked decision D-02.

**Rationale:** Research is only worth its cost when the phase is underspecified; a fully-specified phase goes straight to planning. Atomic commits keep each logical change independently revertable.
**Source:** 03-01-PLAN.md (objective, verification), 03-01-SUMMARY.md (Task Commits)

---

## Lessons

### Verification regexes with character classes can false-positive on unrelated text
Plan verification check 2's regex `/option value=.[WRD 5900]/` matched the `D` in the camera option `<option value="DJI Osmo Action 6">` because the character class `[WRD 5900]` is a set of single characters, not a literal string.

**Context:** The caret-escaped regexes were designed to avoid shell-quoting hazards, but the character-class trick traded one hazard for another — a false positive that initially suggested the hardcoded plat option was still present when it was actually gone.
**Source:** 03-01-SUMMARY.md (Issues Encountered)

### PowerShell quoting mangles `node -e` one-liners with embedded quotes
Two `node -e` verification one-liners failed under PowerShell because embedded `\"` inside double-quoted strings got mangled. The plan's documented fallback — writing the same script to the temp dir and running `node <script>` — worked cleanly.

**Context:** The plan anticipated quoting hazards and documented the fallback in advance; the fallback was needed despite the caret-escaped regexes. Verification scripts with complex regexes should go straight to temp files on Windows.
**Source:** 03-01-SUMMARY.md (Issues Encountered)

---

## Patterns

### Config-driven dropdown pattern
Constant array in Block 2 → `.map()` into `<option>`s → "Other" as last entry → required free-text input revealed on selection → ternary-recorded value written to the payload/handler.

**When to use:** Any future option list (equipment, locations, departments, statuses) that needs to be editable without code changes and must support custom values beyond the preset list.
**Source:** 03-01-SUMMARY.md (patterns-established), 03-CONTEXT.md (D-01..D-04)

### Recorded-value ternary
`selectValue === 'Other' ? customValue : selectValue` computed immediately before the payload/handler write, so the recorded field always holds the literal value the user chose or typed.

**When to use:** Any select-with-custom-input pair where the stored value must be the user's actual choice, not the "Other" sentinel.
**Source:** 03-01-SUMMARY.md (tech-stack patterns), 03-01-PLAN.md (D-03)

### Display preservation by construction
Custom "Other" values flow through the same recorded fields (`platNumber`, `approvedBy`) that all four display surfaces (cards, detail modal, copied details, CSV export) already read — so zero display-surface edits are needed.

**When to use:** When adding configurable inputs, route the recorded value through existing fields rather than adding parallel display logic; verify display surfaces are untouched by construction.
**Source:** 03-01-PLAN.md (success criterion #4), 03-CONTEXT.md (specifics)

---

## Surprises

### Verification regex false positive on the camera option
The plan's own check 2 regex falsely flagged the hardcoded plat option as present because `[WRD 5900]` matched the `D` in "DJI Osmo Action 6". The hardcoded option was actually gone — confirmed via a precise check.

**Impact:** Cost a few minutes of re-verification and required a manual precise check to resolve; no code change needed, but the plan regex was imprecise.
**Source:** 03-01-SUMMARY.md (Issues Encountered)

### PowerShell quoting bit despite caret-escaped regexes
The plan's regexes were deliberately caret-escaped to survive shell quoting, yet two `node -e` one-liners still got mangled by PowerShell's handling of embedded `\"`.

**Impact:** Verification had to fall back to temp script files (the documented plan fallback); the checks themselves passed once run correctly. Lesson: on Windows, prefer temp-file scripts for anything beyond trivial regexes.
**Source:** 03-01-SUMMARY.md (Issues Encountered)