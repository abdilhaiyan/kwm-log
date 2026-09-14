---
phase: 02
slug: config-driven-passcode
status: verified
threats_open: 0
asvs_level: 1
created: 2026-09-14
---

# Phase 02 — Security

> Per-phase security contract: threat register, accepted risks, and audit trail.

---

## Trust Boundaries

| Boundary | Description | Data Crossing |
|----------|-------------|---------------|
| Browser ↔ Firebase (Firestore) | Borrow-request CRUD via anonymous auth; rules unchanged this phase | Request data (staff name, equipment, dates, notes) |
| Manager gate (client-side) | `passcodeAttempt === DEFAULT_PASSCODE` unlocks manager view | None — in-app state only, no data crossing |

---

## Threat Register

| Threat ID | Category | Component | Severity | Disposition | Mitigation | Status |
|-----------|----------|-----------|----------|-------------|------------|--------|
| T-02-01 | Spoofing | Unlock handler (`passcodeAttempt === DEFAULT_PASSCODE`) | medium | accept | Client-side constant gate is the locked design (ROADMAP goal + PASS-01); value was already readable as the Firestore fallback before this phase; no data-access boundary crossed (anonymous auth only, rules unchanged) | closed |
| T-02-02 | Tampering | Removed rotation write (app config doc `.set`) | low | mitigate | Firestore write path deleted outright — no code path can mutate a passcode field (verified: no `app_config` / `.set(` in index.html) | closed |
| T-02-03 | Information Disclosure | Block 2 constant `"1234"` in source | low | accept | Internal LAN tool; value was already exposed as the fallback and via the Firestore doc before; D-01 rotation comment makes the operator-visible rotation model explicit | closed |
| T-02-04 | Elevation of Privilege | Manager view gate | medium | accept | Anyone reading the source can unlock — unchanged in substance from the pre-change design (single passcode, no accounts per PROJECT.md); this phase introduces no new privilege boundary | closed |
| T-02-05 | Denial of Service | Firestore request surface | low | accept | Removing the app config fetch + write reduces Firestore operations; no new unauthenticated path added | closed |
| T-02-06 | Integrity | .planning/PROJECT.md, research/FEATURES.md, research/ARCHITECTURE.md | low | mitigate | Acceptance-criteria negative greps + read-through human-check keep the docs from re-asserting the removed Firestore passcode path (verified: PROJECT.md:65, FEATURES.md:165-166 describe constant-based rotation) | closed |
| T-02-SC | Tampering | npm/pip/cargo installs | high | accept | No package installations this phase (single-file HTML, no build step) — nothing to vet | closed |

*Status: open · closed · open — below high threshold (non-blocking)*
*Severity: critical > high > medium > low — only open threats at or above workflow.security_block_on count toward threats_open*
*Disposition: mitigate (implementation required) · accept (documented risk) · transfer (third-party)*

---

## Accepted Risks Log

| Risk ID | Threat Ref | Rationale | Accepted By | Date |
|---------|------------|-----------|-------------|------|
| AR-02-01 | T-02-01 | Client-side constant gate is the locked design (ROADMAP goal + PASS-01); value was already readable as the Firestore fallback; no data-access boundary crossed | Phase 2 plan (02-01-PLAN.md) | 2026-09-14 |
| AR-02-02 | T-02-03 | Internal LAN tool; value already exposed as fallback and via Firestore doc before; D-01 rotation comment makes rotation explicit | Phase 2 plan (02-01-PLAN.md) | 2026-09-14 |
| AR-02-03 | T-02-04 | Anyone reading source can unlock — unchanged in substance from pre-change design; no new privilege boundary | Phase 2 plan (02-01-PLAN.md) | 2026-09-14 |
| AR-02-04 | T-02-05 | Removing app config fetch + write reduces Firestore operations; no new unauthenticated path | Phase 2 plan (02-01-PLAN.md) | 2026-09-14 |
| AR-02-05 | T-02-SC | No package installations this phase — nothing to vet | Phase 2 plan (02-01-PLAN.md) | 2026-09-14 |

*Accepted risks do not resurface in future audit runs.*

---

## Security Audit Trail

| Audit Date | Threats Total | Closed | Open | Run By |
|------------|---------------|--------|------|--------|
| 2026-09-14 | 7 | 7 | 0 | OpenAgent (L1 grep verification, short-circuit: threats_open 0 + register at plan time + ASVS L1) |

---

## Sign-Off

- [x] All threats have a disposition (mitigate / accept / transfer)
- [x] Accepted risks documented in Accepted Risks Log
- [x] `threats_open: 0` confirmed
- [x] `status: verified` set in frontmatter

**Approval:** verified 2026-09-14