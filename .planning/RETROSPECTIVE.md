# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.0 — MVP

**Shipped:** 2026-09-15
**Phases:** 3 | **Plans:** 6 | **Sessions:** ~6

### What Was Built
- Decomposed the monolithic single-file React app into 7 ordered `text/babel` blocks (Constants → Config → Firebase Init → Utilities → Shared UI → App Shell → Render) with zero user-visible behavior change
- Manager passcode became a config-driven HTML constant (`DEFAULT_PASSCODE` in Block 2) — Firestore passcode read/write path removed entirely, rotation is a one-line edit
- Plat Number and Approver became config-driven selects from `PLAT_OPTIONS`/`APPROVER_OPTIONS` constants with "Other" free-text reveal — adding an option is a one-line insert
- Planning docs (PROJECT.md, FEATURES.md, ARCHITECTURE.md) refreshed in-phase to describe the constant-based passcode (D-03 doc-truthfulness discipline)

### What Worked
- Pure-move extraction: byte-identical moves of constants/utilities/components into new blocks, verified by diff — zero behavior change, low risk
- Config-driven dropdown pattern (constant → `.map()` → "Other" last → required free-text reveal → ternary-recorded value) generalized the existing equipment pattern cleanly
- In-phase doc refresh (D-03): planning docs updated in the same phase as the code change they describe, so future planners read the actual implemented state
- Estimate calibration: realized diffs measured (chars/4) and recorded in SUMMARY frontmatter — feeding a calibration factor (0.5) for future estimates

### What Was Inefficient
- PowerShell quoting mangled `node -e` one-liners with embedded double quotes (Phase 2/3) — required documented fallback patterns (dot-wildcard regex, temp script files)
- Plan regex imprecision (Phase 3 check 2): `/option value=.[WRD 5900]/` false-positive matched the `D` in "DJI Osmo Action 6" — needed a precise re-check
- Phase 1 plans initially referenced fictional symbols (EQUIPMENT_OPTIONS, CAMERA_MODELS, etc.) that didn't exist in the code — required plan rewrite after grep = 0 matches
- Phases 1 & 3 never got formal `*-VERIFICATION.md` reports (user-tested via UAT instead) — milestone closed with verification overrides

### Patterns Established
- Multi-block text/babel decomposition: inline blocks share global scope; dependency order enforced by script order; zero import/export
- Constant-as-config validation source: passcode read from the Block 2 HTML constant; rotation = edit the constant
- Config-driven dropdown pattern: constant array in Block 2 → `.map()` options → "Other" last → required free-text reveal → ternary-recorded value
- Doc-truthfulness discipline (D-03): planning docs updated in the same phase as the code change they describe

### Key Lessons
1. Verify plan assumptions against the actual codebase before planning (Phase 1 plans referenced symbols that didn't exist — grep first)
2. Record realized diffs (chars/4) in SUMMARY frontmatter to calibrate estimates — initial estimates ran ~10x high
3. When a plan removes/changes runtime behavior, its own files_modified list must carry the doc updates so planning docs never drift from code
4. PowerShell quoting breaks `node -e` one-liners with embedded quotes — use dot-wildcard regex or temp script files

### Cost Observations
- Model mix: adaptive profile (opus planning/checking, haiku verification)
- Sessions: ~6
- Notable: 68 commits total; 3 feat() commits; most work was docs/planning commits (GSD discipline)

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Sessions | Phases | Key Change |
|-----------|----------|--------|------------|
| v1.0 | ~6 | 3 | Established 7-block decomposition, constant-as-config pattern, estimate calibration |

### Cumulative Quality

| Milestone | Tests | Coverage | Zero-Dep Additions |
|-----------|-------|----------|-------------------|
| v1.0 | 0 (no test framework — single-file CDN app) | n/a | 0 (no new CDN deps) |

### Top Lessons (Verified Across Milestones)

1. Verify plan assumptions against actual code before planning (grep first)
2. Record realized diffs to calibrate estimates
3. Keep planning docs truthful to implemented code (D-03)