# Phase 3: Configurable Plat & Approver Dropdowns - Context

**Gathered:** 2026-09-14
**Status:** Ready for planning

<domain>
## Phase Boundary

Plat Number and Approver become `<select>` dropdowns populated from editable constant arrays at the top of index.html (Block 2, Config). Both dropdowns include an "Other" option (last entry) that reveals a free-text input for a custom value, following the existing equipment "Other" pattern. Adding a new plate or approver is a one-line constant edit — no JSX or logic changes. Plat number and approver continue to display correctly in request cards, detail modal, copied details, and CSV export.

</domain>

<decisions>
## Implementation Decisions

### Option Constants
- **D-01:** Add two simple string arrays to Block 2 (Config), adjacent to `DEFAULT_PASSCODE`:
  - `const PLAT_OPTIONS = ['WRD 5900', 'Other'];` — with comment `// Vehicle plates — edit to add options`
  - `const APPROVER_OPTIONS = ['Shafiq', 'Other'];` — with comment `// Approvers — edit to add options`
  - "Other" is the **last entry** — adding a new option is a one-line insert before it. Simple strings (not value/label objects) per the "one-line constant edit" success criterion.

### "Other" → Free-Text Reveal
- **D-02:** Selecting "Other" in either dropdown reveals a free-text input, mirroring the equipment pattern (line 236: `eq === 'Other'` → `customEquipment` input). New App Shell state: `customPlat` (string) and `customApprover` (string). The typed value is what gets recorded.

### Recording
- **D-03:**
  - Plat: `finalPlat = platNumber === 'Other' ? customPlat : platNumber`; payload uses `finalPlat` when `cameraModel === 'Car'` (extends the line 160 pattern: `platNumber: formData.cameraModel === 'Car' ? platNumber : ''`).
  - Approver: `finalApprover = approverName === 'Other' ? customApprover : approverName`; `updateStatus(..., { approvedBy: finalApprover || 'N/A', ... })` (extends the line 390/502 pattern).

### Rendering
- **D-04:** Both selects map their constant array to `<option>`s via `.map()`. Plat select stays conditional on `eq === 'Car'` (line 242). Approver select appears in **two places** (request card line 386, detail modal line 498) — both map `APPROVER_OPTIONS`. Custom inputs: plat "Other" input renders when `eq === 'Car' && platNumber === 'Other'`; approver "Other" input renders when `approverName === 'Other'` (in both card and modal).

### Agent Discretion
- Custom inputs use `required` when shown, matching the equipment pattern (line 236).
- `approverName` stays shared state between card and modal (existing behavior preserved).
- Verification: programmatic checks (constants present, both selects map arrays, no hardcoded option values left in JSX) + real-browser UAT at `http://10.0.6.12:8080` (preview env cannot run Babel standalone — confirmed in Phase 1).

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Requirements & Roadmap
- `.planning/REQUIREMENTS.md` §Plat Number — PLAT-01, PLAT-02, PLAT-03; §Approver — APPR-01, APPR-02 (the phase's requirements)
- `.planning/ROADMAP.md` §Phase 3 — Goal, 4 success criteria, mode (mvp), dependency on Phase 2

### Prior Phase Context (carried-forward decisions)
- `.planning/phases/01-multi-block-decomposition/1-CONTEXT.md` — locked decisions D-01..D-04 (inline blocks, one commit per change, verify after each change, preserve `data-passcode-unlock` + `lucide.createIcons()` hacks as-is)
- `.planning/phases/02-config-driven-passcode/2-CONTEXT.md` — D-01..D-03 (rotation comment, dead `app_config` doc, doc refresh) — establishes the constant-in-Block-2 pattern and verification approach

### Existing Code Reference
- `index.html` — Block 2 (Config): `DEFAULT_PASSCODE` (~line 58) — new constants go here
- `index.html` — App Shell: `platNumber` state (line 114), `approverName` state (line 115); payload construction (line 160); `updateStatus` (lines 170-172)
- `index.html` — Plat select (lines 242-245): placeholder + single hardcoded option; conditional on `eq === 'Car'`
- `index.html` — Approver selects (lines 386-389 card, 498-501 modal): placeholder + single hardcoded option
- `index.html` — Equipment "Other" free-text pattern (line 236): the template for D-02
- `index.html` — Display surfaces (success criterion #4): card equipment label (line 377), detail modal platNumber (line 474), copied details (line 180), CSV export (lines 212-215)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- Equipment "Other" free-text pattern (line 236): `{eq === 'Other' && <input required ... value={formData.customEquipment} ...>}` — the exact template for plat/approver "Other" inputs.
- `SELECT_CLS` / `INPUT_CLS` constants (Block 1): reused as-is for the new selects/inputs.
- Block 2 (Config) location: `DEFAULT_PASSCODE` already lives here — the new arrays sit beside it.

### Established Patterns
- Pure-move discipline from Phase 1: edits are surgical; no unrelated changes.
- One commit per logical change (Phase 1 locked decision D-02).
- Programmatic verification scripts (node, written to temp) + real-browser UAT checklist at `http://10.0.6.12:8080`.

### Integration Points
- Block 2 (Config): add `PLAT_OPTIONS` + `APPROVER_OPTIONS` (D-01).
- App Shell (Block 6): add `customPlat` + `customApprover` state; compute `finalPlat` in `submitRequest` (line 160) and `finalApprover` in the two approve handlers (lines 390, 502).
- Form fields (Block 6): plat select maps `PLAT_OPTIONS`; "Other" input renders when `eq === 'Car' && platNumber === 'Other'`.
- Request card + detail modal (Block 6): approver selects map `APPROVER_OPTIONS`; "Other" input renders when `approverName === 'Other'`.

</code_context>

<specifics>
## Specific Ideas

- Success criterion #4 (display preservation) is satisfied by construction: custom "Other" values flow through the same `platNumber` / `approvedBy` fields, so cards (line 377), detail modal (line 474), copied details (line 180), and CSV export (lines 212-215) need zero changes.
- The card approver area is compact (`flex md:flex-col gap-2 shrink-0`, line 384) — the "Other" input must fit the existing layout; the modal (line 496) has more room. Plan phase should confirm the input classes (`INPUT_CLS + " text-sm"` style, matching the select).

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 3-Configurable Plat & Approver Dropdowns*
*Context gathered: 2026-09-14*