# Phase 2: Config-Driven Passcode - Pattern Map

**Mapped:** 2026-09-14
**Files analyzed:** 4 (1 code file modified in place + 3 planning docs per D-03)
**Analogs found:** 4 / 4 (all exact — every analog is the file's own existing code being edited)

## File Classification

| New/Modified File | Role | Data Flow | Closest Analog | Match Quality |
|-------------------|------|-----------|----------------|---------------|
| `index.html` (passcode sections) | component (single-file App Shell) | request-response (UI event → state) + CRUD removal | `index.html` itself — existing passcode code (Block 2 line 58, Block 6 lines 97-98/115-116/153-158/420-448) | exact |
| `.planning/PROJECT.md` (D-03 doc) | config / doc | transform (doc edit) | itself — Firebase section lines 30, 33, 54, 65, 75 | exact |
| `.planning/research/FEATURES.md` (D-03 doc) | config / doc | transform (doc edit) | itself — passcode rows lines 23, 40, 165-166 | exact |
| `.planning/research/ARCHITECTURE.md` (D-03 doc) | config / doc | transform (doc edit) | itself — passcode rows lines 44-45, 68, 109-110, 231-232, 321-327, 378, 415 | exact |

**Scope note:** No new files are created. No RESEARCH.md exists for this phase (research skipped). The phase is a surgical removal + constant-validation swap inside the existing `index.html`, plus three doc edits.

---

## Pattern Assignments

### `index.html` — Passcode code (component, request-response + CRUD removal)

**Analog:** `index.html` itself. The executor edits the file's own passcode code in place. All excerpts below are the CURRENT pre-change code, with exact line numbers as of 2026-09-14. Preserve everything not listed for removal byte-identically.

**File context:** 7 `text/babel` blocks (Phase 1 decomposition, locked). Block 2 = Config (lines 48-59), Block 6 = App Shell (lines 89-448+, `App` component, 19 useState hooks). This phase touches only Block 2 line 58 and Block 6 passcode code.

---

#### Edit A — D-01: Add rotation comment above `DEFAULT_PASSCODE` (Block 2, line 58)

**Current code** (lines 48-59, byte-identical Block 2):
```html
    <script type="text/babel">
        const firebaseConfig = {
            apiKey: "AIzaSyCH8tnCz0sgMdElLkza0FuW5aM84GkhgUQ",
            authDomain: "kwm-logistics-camera.firebaseapp.com",
            projectId: "kwm-logistics-camera",
            storageBucket: "kwm-logistics-camera.firebasestorage.app",
            messagingSenderId: "393195572454",
            appId: "1:393195572454:web:42676f73a430b075157f27"
        };
        const appId = window.__app_id || 'camera-borrow-wiramas';
        const DEFAULT_PASSCODE = "1234";
    </script>
```

**Edit:** Insert `// Manager passcode — edit to rotate` above `const DEFAULT_PASSCODE = "1234";` (line 58). Nothing else in Block 2 changes; `firebaseConfig` and `appId` stay byte-identical.

**Post-edit target (line 58 area):**
```javascript
        const appId = window.__app_id || 'camera-borrow-wiramas';
        // Manager passcode — edit to rotate
        const DEFAULT_PASSCODE = "1234";
```

---

#### Edit B — Remove `newPasscode` + `currentPasscode` state (Block 6, lines 115-116)

**Current code** — passcode state declarations among the useState block (lines 97-98 KEEP, 115-116 REMOVE):
```javascript
            const [passcodeAttempt, setPasscodeAttempt] = useState("");          // line 97 — KEEP
            const [showPasscodeModal, setShowPasscodeModal] = useState(false);   // line 98 — KEEP
```
```javascript
            const [newPasscode, setNewPasscode] = useState('');                  // line 115 — REMOVE
            const [currentPasscode, setCurrentPasscode] = useState(DEFAULT_PASSCODE); // line 116 — REMOVE
```

**Edit:** Delete lines 115-116 entirely. Keep lines 97-98 (`passcodeAttempt`, `showPasscodeModal`) — they remain the modal's state. All other useState hooks (lines 93-114) untouched. After removal, validation happens against the module-scope `DEFAULT_PASSCODE` constant (Block 2) directly — no React state copy of the passcode.

---

#### Edit C — Remove the Firestore `app_config` fetch `useEffect` (Block 6, lines 153-158)

**Current code** (lines 152-158, the effect is the 5th useEffect, immediately after the seen-requests effect at line 151):
```javascript
            useEffect(() => { localStorage.setItem('kwm_seen_requests', JSON.stringify([...seenRequestIds])); }, [seenRequestIds]);   // line 151 — KEEP

            useEffect(() => {                                                                                                          // line 153 — REMOVE (whole block)
                if (!db) return;
                db.collection('artifacts/' + appId + '/public/data').doc('app_config').get().then(doc => {
                    if (doc.exists && doc.data().passcode) setCurrentPasscode(doc.data().passcode);
                }).catch(() => {});
            }, []);
```

**Edit:** Delete lines 153-158 (the entire effect). KEEP the other four effects: dark mode (line 148), `onAuthStateChanged` (line 149), requests `onSnapshot` (line 150), seen-requests persistence (line 151). The `onSnapshot` listener at line 150 is the ONLY remaining Firestore read — `app_config` must not appear anywhere in the file after this edit (ROADMAP success criterion #3 + PASS-02).

**Verification hook for planner:** after the edit, `grep -n "app_config" index.html` must return zero matches; `grep -n "currentPasscode\|newPasscode" index.html` must return zero matches.

---

#### Edit D — Passcode modal: remove "New passcode" input + simplify unlock handler (Block 6, lines 420-448)

**Current code** (complete modal, lines 420-448):
```jsx
                    {showPasscodeModal && (
                        <div className="fixed inset-0 z-50">
                            <div className="absolute inset-0 bg-black/50 backdrop-blur-sm" onClick={() => setShowPasscodeModal(false)} />
                            <div className="absolute bottom-0 left-0 right-0 md:bottom-auto md:top-1/2 md:left-1/2 md:-translate-x-1/2 md:-translate-y-1/2 glass-card bg-white/20 dark:bg-[#1e1e2a]/90 p-6 md:p-10 rounded-t-2xl md:rounded-3xl w-full max-w-sm mx-auto border border-white/30 dark:border-[#3a3a4a]">
                                <div className="w-10 h-1 bg-white/30 rounded-full mx-auto mb-4 md:hidden" />
                                <h3 className="text-lg md:text-xl font-black mb-4 md:mb-6 text-center text-white">Admin Access</h3>
                                <input type="password" autoFocus className={INPUT_CLS + " text-center text-xl md:text-2xl mb-4 placeholder-white/40"} value={passcodeAttempt} onChange={e => setPasscodeAttempt(e.target.value)} onKeyDown={e => { if (e.key === 'Enter') document.querySelector('[data-passcode-unlock]').click(); }} />          // line 426 — KEEP byte-identical
                                <input type="text" placeholder="New passcode (optional)" className={INPUT_CLS + " text-sm mb-4 placeholder-white/40"} value={newPasscode} onChange={e => setNewPasscode(e.target.value)} onKeyDown={e => { if (e.key === 'Enter') document.querySelector('[data-passcode-unlock]').click(); }} />          // line 427 — REMOVE
                                <button data-passcode-unlock onClick={() => {                                                                                                                     // line 428 — KEEP attribute, EDIT handler
                                    if (passcodeAttempt === currentPasscode) {                                                                                                                   // line 429 — EDIT: compare against DEFAULT_PASSCODE
                                        setIsManagerAuthenticated(true);                                                                                                                          // line 430 — KEEP
                                        setShowPasscodeModal(false);                                                                                                                              // line 431 — KEEP
                                        setView('manager');                                                                                                                                        // line 432 — KEEP
                                        markRequestsAsSeen();                                                                                                                                      // line 433 — KEEP
                                        if (newPasscode.trim() && db) {                                                                                                                            // line 434 — REMOVE (Firestore write block)
                                            db.collection('artifacts/' + appId + '/public/data').doc('app_config').set({ passcode: newPasscode.trim() }).then(() => {                              // line 435 — REMOVE
                                                setCurrentPasscode(newPasscode.trim());                                                                                                           // line 436 — REMOVE
                                                handleNotify("Passcode updated");                                                                                                                  // line 437 — REMOVE
                                            }).catch(() => {});                                                                                                                                    // line 438 — REMOVE
                                            setNewPasscode('');                                                                                                                                    // line 439 — REMOVE
                                        }                                                                                                                                                          // line 440 — REMOVE
                                    } else {                                                                                                                                                       // line 441 — KEEP (wrong-passcode behavior)
                                        setPasscodeAttempt("");                                                                                                                                    // line 442 — KEEP byte-identical
                                    }                                                                                                                                                              // line 443 — KEEP
                                }} className="w-full bg-white/80 dark:bg-white/20 text-slate-900 dark:text-white py-3.5 md:py-4 rounded-xl font-bold hover:bg-white dark:hover:bg-white/30 transition-all border border-white/50 dark:border-[#3a3a4a]">Unlock</button>   // line 444 — KEEP className byte-identical
                                <button onClick={() => setShowPasscodeModal(false)} className="w-full mt-3 py-2 text-sm text-white/50 font-bold uppercase hover:text-white/80 transition-all">Cancel</button>                                                       // line 445 — KEEP
                            </div>
                        </div>
                    )}
```

**Post-edit target — unlock handler body (lines 429-443 become):**
```jsx
                                <button data-passcode-unlock onClick={() => {
                                    if (passcodeAttempt === DEFAULT_PASSCODE) {
                                        setIsManagerAuthenticated(true);
                                        setShowPasscodeModal(false);
                                        setView('manager');
                                        markRequestsAsSeen();
                                    } else {
                                        setPasscodeAttempt("");
                                    }
                                }} className="w-full bg-white/80 dark:bg-white/20 text-slate-900 dark:text-white py-3.5 md:py-4 rounded-xl font-bold hover:bg-white dark:hover:bg-white/30 transition-all border border-white/50 dark:border-[#3a3a4a]">Unlock</button>
```

**Pattern rules for this edit (locked by Phase 1 decisions + CONTEXT.md):**
1. `data-passcode-unlock` attribute on the button: **preserve exactly** (locked decision D-04 from Phase 1; the Enter-key `onKeyDown` at line 426 clicks it via `document.querySelector('[data-passcode-unlock]')`).
2. Enter-key `onKeyDown` on the passcode input (line 426): **preserve byte-identical** — same `document.querySelector('[data-passcode-unlock]').click()` pattern.
3. Wrong-passcode branch (`else { setPasscodeAttempt(""); }`): **preserve byte-identical** — input clears, no toast (locked by ROADMAP success criterion #2).
4. The 4-line success branch (lines 430-433: `setIsManagerAuthenticated(true)` → `setShowPasscodeModal(false)` → `setView('manager')` → `markRequestsAsSeen()`) stays in the same order, staying a single synchronous callback.
5. Remove line 427 (New passcode input), lines 434-440 (Firestore write block), and the `handleNotify("Passcode updated")` call. `handleNotify` (line 160) itself is NOT removed — still used by submit/update/export/copy toasts.
6. Modal chrome — overlay, container classes, drag handle, "Admin Access" title, passcode input classes, Cancel button — all byte-identical (ROADMAP success criterion #5).

**The modal's sibling analog — `returnConditionModal` JSX (lines 450-472)** — same overlay/container/chrome structure (`fixed inset-0 z-50` → `bg-black/50 backdrop-blur-sm` overlay → `bottom-0 ... md:top-1/2 md:-translate-x-1/2 md:-translate-y-1/2 glass-card ... max-w-sm` → drag handle → title → inputs → action button → Cancel). Use it as the structural reference if the passcode modal's container must ever be re-verified — but this phase does not touch it.

---

### `.planning/PROJECT.md` (D-03 doc update)

**Analog:** the file's own passcode references. Exact current lines:

- **Line 30** (Validated requirements): `- ✓ Passcode-protected manager actions (changeable, stored in Firestore) — existing` → revise to reflect constant-based passcode (e.g., "stored as HTML constant").
- **Line 33** (Validated): `- ✓ Enter key submits passcode modal — existing` → unchanged (feature still true).
- **Line 54** (Context → Firebase): `` - **Firebase:** Project `kwm-logistics-camera`. ... App config (changeable passcode) at `artifacts/camera-borrow-wiramas/public/data/app_config` (doc id `app_config`, field `passcode`, fallback `DEFAULT_PASSCODE` = "1234"). `` → rewrite to remove the `app_config` passcode description; state passcode is the `DEFAULT_PASSCODE` constant in index.html Block 2 (edit to rotate). Optionally note `app_config` doc exists as dead data (D-02: leave untouched).
- **Line 65** (Constraints → Security): `- **Security**: Manager actions gated by passcode (Firestore-backed, changeable). No user accounts.` → change "Firestore-backed, changeable" to constant-based (e.g., "gated by passcode constant in HTML; rotate by editing DEFAULT_PASSCODE").
- **Line 75** (Key Decisions): `| Passcode in Firestore (changeable) | Managers can rotate it without editing code | ✓ Good |` → mark this decision superseded (e.g., change decision to "Passcode as HTML constant" or mark row as "— Superseded by Phase 2").

**Style to preserve:** tab-separated `* bullet` / `| table |` markdown matching the file's existing format; English; no emojis added beyond existing ✓/- markers.

---

### `.planning/research/FEATURES.md` (D-03 doc update)

**Analog:** the file's own passcode rows. Exact current lines:

- **Line 23** (Features table, "Manager authentication gate" row): `| **Manager authentication gate** | Passcode modal on Manager tab click, Enter key submits, passcode validated against Firestore `app_config` doc (fallback `DEFAULT_PASSCODE = "1234"`), optional new passcode field updates Firestore | MEDIUM | Passcode is loaded from Firestore on mount; must preserve the flow: modal → validate → set `isManagerAuthenticated` → switch to manager view |` → rewrite description/notes: validated against HTML constant `DEFAULT_PASSCODE = "1234"` (Block 2); no Firestore read; no rotation field; flow preserved: modal → validate against constant → set `isManagerAuthenticated` → switch to manager view.
- **Line 40** (Features table, "Enter key on passcode" row): description mentions `onKeyDown` + `document.querySelector('[data-passcode-unlock]').click()` → unchanged (behavior preserved); keep as-is unless the row ties to Firestore validation.
- **Line 165** (Validation table): `| Passcode gate | Click Manager tab → enter wrong passcode | Modal stays, input clears. Enter correct → switches to manager view |` → unchanged (expected behavior identical).
- **Line 166** (Validation table): `| Change passcode | Enter new passcode in modal → Unlock | Firestore `app_config` updated, new passcode works on next attempt |` → REMOVE this row (control is gone; feature no longer exists). Optionally replace with "Rotation" expectation: edit `DEFAULT_PASSCODE` constant in index.html.

---

### `.planning/research/ARCHITECTURE.md` (D-03 doc update)

**Analog:** the file's own passcode references. Exact current lines:

- **Line 44** (block inventory table, Config row): `| Firebase config & init | 45-57 | — (globals) | `firebaseConfig`, `appId`, `DEFAULT_PASSCODE`, `app/auth/db` globals |` → keep `DEFAULT_PASSCODE`; note it is now the sole passcode source (add "edit to rotate" note).
- **Line 45** (Firestore reads/writes row): includes `passcode fetch` → remove that phrase; line's remaining content (`onSnapshot` listener, `submitRequest`, `updateStatus`, `markAsReturned`) stays.
- **Line 68** (PasscodeModal component row): props/state list includes `newPasscode`, `setNewPasscode`, `currentPasscode` — the currentPasscode provenance is "From Firestore" → remove `newPasscode`/`setNewPasscode`/`currentPasscode` from the list; note validation against `DEFAULT_PASSCODE` constant, no rotation.
- **Line 109-110** (future dependency-graph calls): `fetchPasscode()` / `updatePasscode(newPasscode)` → remove or mark superseded (no passcode fetch/update service functions exist anymore).
- **Line 118** (`PasscodeModal, ReturnConditionModal, DetailModal` in future component tree): keep `PasscodeModal` (modal still exists, no rotation); remove the "(+ rotation)" implication (check accompanying text).
- **Line 211, 221** (future ESM layout): `import { PasscodeModal } from "./components/passcode-modal.js";` and `components/passcode-modal.js ← Admin gate + passcode rotation` → keep the component, drop "passcode rotation" phrasing.
- **Line 231-232** (future ESM exports): `export async function fetchPasscode()` / `export async function updatePasscode(new)` → remove both (no Firestore passcode service).
- **Line 269** (future App Shell effects): `useEffect: Firestore subscription, dark mode, passcode fetch` → drop "passcode fetch".
- **Line 284** (future service list): `fetchPasscode() / updatePasscode()` → remove.
- **Line 321-322** (state table): `isManagerAuthenticated`, `showPasscodeModal` rows → unchanged (still held in App Shell).
- **Line 327** (state table): `| `currentPasscode` | App Shell | PasscodeModal (reads) | From Firestore |` → REMOVE this row entirely (state no longer exists).
- **Line 378** (block dependency note): `FirebaseService reads from global config` with `DEFAULT_PASSCODE` shared global — keep `DEFAULT_PASSCODE`; adjust any "FirebaseService reads passcode" implication.
- **Line 415, 422** (future extraction plan mentions `fetchPasscode, updatePasscode`): remove those references.

**Caveat for planner:** ARCHITECTURE.md lines 149+ describe an OPTIONAL future "Phase 2: ES Modules + Import Maps" — a stale hypothetical that predates the current ROADMAP Phase 2 (Config-Driven Passcode). Update only the passcode-flow references listed above; do not rewrite the ES-module section wholesale.

---

## Shared Patterns

### Constant-as-config validation source
**Source:** `index.html` Block 2, line 58 (`const DEFAULT_PASSCODE = "1234";`)
**Apply to:** App Shell unlock handler (line 429 compares `passcodeAttempt === currentPasscode` → becomes `passcodeAttempt === DEFAULT_PASSCODE`). The constant is module-scope, available to all 7 blocks (shared global scope — Phase 1 architecture, no imports).

### Preserved hack: `data-passcode-unlock` DOM query (Phase 1 locked decision D-04)
**Source:** `index.html` lines 426 + 428
**Apply to:** The Enter-key handler and the Unlock button. Do NOT "clean up" the `document.querySelector('[data-passcode-unlock]').click()` pattern or remove the `data-passcode-unlock` attribute. Zero-cleanup rule from Phase 1 applies to this phase too.

### Preserved timing hack: `lucide.createIcons()` 100ms timeout
**Source:** `index.html` line 150 (`setTimeout(() => lucide.createIcons(), 100)` inside the onSnapshot effect)
**Apply to:** Untouched by this phase, but the requests `onSnapshot` effect (the effect immediately before the removed passcode-fetch effect) must remain byte-identical — removing the 153-158 effect must NOT disturb line 150.

### Wrong-passcode error behavior — inline silent clear, no toast
**Source:** `index.html` line 441-442 (`else { setPasscodeAttempt(""); }`)
**Apply to:** Preserved exactly. Locked by ROADMAP success criterion #2 ("rejected with the same error behavior").

### Verification pattern (established Phase 1)
**Source:** `.planning/phases/01-multi-block-decomposition/01-01-PLAN.md` lines 101-111 (node one-liner programmatic checks + real-browser UAT)
**Apply to:** Phase 2 verification — programmatic checks (`grep node -e` asserting zero `app_config` / `currentPasscode` / `newPasscode` matches, 7 text/babel blocks still present, modal structure) + real-browser UAT at `http://10.0.6.12:8080` (Babel standalone cannot run in preview env — Phase 1 finding). One commit per logical change (Phase 1 D-02): suggested commit split — (1) doc updates D-03, (2) code edits A-D.

---

## No Analog Found / Out of Scope

| File | Why |
|------|-----|
| `.planning/research/STACK.md` (lines 31, 35, 39) | Contains stale passcode references (`data.js — passcode fetch effect`, `constants.js — DEFAULT_PASSCODE`, `views/modals.jsx — passcode modal`). D-03 names only PROJECT.md, FEATURES.md, ARCHITECTURE.md — **do not edit** unless planner chooses to extend D-03. Flagged for awareness. |
| `.planning/research/SUMMARY.md` (lines 38, 73, 91, 114, 116, 144) | Stale passcode service references (`fetchPasscode()`, `updatePasscode(new)`). Same D-03 scope note — not named in D-03. |
| `.planning/research/PITFALLS.md` (lines 64-88, 224, 236, 256-272, 288, 298) | Pitfall #3 (passcode gate state coupling) and the risk register describe Firestore-backed passcode; some entries become obsolete after this phase. Not named in D-03. |
| `.planning/STATE.md` (lines 50, 63), `.planning/REQUIREMENTS.md` (PASS-01..03) | State/roadmap docs; REQUIREMENTS PASS-02 already requires removing Firestore logic — no edit needed (checkboxes flip at phase completion). |
| Firestore `app_config` doc itself | D-02: leave untouched — becomes harmless dead data. No programmatic deletion. |

## Metadata

**Analog search scope:** `index.html` (all passcode sections), `.planning/PROJECT.md`, `.planning/research/FEATURES.md`, `.planning/research/ARCHITECTURE.md`, `.planning/research/{STACK,SUMMARY,PITFALLS}.md`, `.planning/REQUIREMENTS.md`, `.planning/ROADMAP.md`, `.planning/STATE.md`, `.planning/phases/01-multi-block-decomposition/*` (Phase 1 CONTEXT/PLANs/SUMMARY for locked decisions & verification patterns)
**Files scanned:** 13 (1 source + 12 planning docs)
**Pattern extraction date:** 2026-09-14
**Research input:** none (RESEARCH.md skipped for this phase — per orchestrator note; pattern map built from CONTEXT.md + index.html + planning docs)