<!-- GSD:project-start source:PROJECT.md -->

## Project

**KWM Logistics Equipment Log**

A mobile-first web app for KWM Logistics staff to request equipment (cameras, vehicles, other gear) and for managers to approve, reject, and track handover. Staff submit a borrow request with their details and the equipment they need; managers review it, approve or reject, record who approved, and mark when the item was passed over. Built as a single-file React app with a Firebase backend, styled with a glassmorphic dark-mode UI.

**Core Value:** Accountable equipment handover — track who borrowed what, when, and whether it was returned, with an approval trail (approver name, passed status) so nothing gets lost or unaccounted.

### Constraints

- **Tech stack**: Single-file HTML with CDN React 18 + Tailwind + Babel + Lucide + Firebase compat v11.6.1 — no build step, no npm, no bundler. Keep it that way.
- **Firebase API style**: Namespaced compat API (`firebase.auth()`, `db.collection()`, `.add()`, `.doc().update()`) — not the modular v9+ API.
- **Mobile-first**: Staff use phones over LAN HTTP (non-secure context). Clipboard, layout, and touch interactions must work there.
- **Security**: Manager actions gated by passcode (Firestore-backed, changeable). No user accounts.
- **Compatibility**: Must keep working as a single file that can be opened/served anywhere without a build step.

<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->

## Technology Stack

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| React (keep, PIN from `@18`) | 18.3.1 UMD | UI framework | React 19 (Dec 2024) **removed UMD builds entirely** — `unpkg.com/react@19/umd/...` does not exist. The app's existing `@18` pin resolves to 18.3.1, the final 18.x, whose UMD builds remain served (HEAD-verified live). `@18` is effectively frozen, but pinning the exact 18.3.1 URL removes the residual CDN-drift risk. Upgrading to React 19 would force a switch to ESM loading — out of scope for a behavior-preserving refactor. |
| ReactDOM (keep, PIN) | 18.3.1 UMD | React rendering | Same reasoning as React. Verify `react-dom@18.3.1/umd/react-dom.production.min.js` — HEAD 200 text/javascript confirmed. |
| @babel/standalone (keep, PIN) | 8.0.5 | In-browser JSX transform + **the decomposition engine** | The only mature client-side JSX compiler. Current at 8.0.5 (Babel 8, June 2026). Official docs + source confirm: auto-compiles `text/babel` / `text/jsx` script tags, loads external `src=` scripts via XHR (**MIME-agnostic** — `.jsx` served as `application/octet-stream` by Python still compiles), executes in document order, and defaults to the `react` preset with `runtime: 'classic'` in browsers (changelog #18086) — exactly the app's current `React.createElement` usage. Babel 8's ESM-only change applies to Node packages, not the browser bundle. |
| Multi-file classic scripts (the mechanism) | n/a | File decomposition | Split the one inline `text/babel` block into several external `text/babel` scripts. Classic scripts share the global lexical scope, so `constants.js` → `utils.js` → `views/*.jsx` → `App.jsx` reference each other naturally. No import/export, no bundler, no behavior change. Deployment stays "serve the folder". |
| Tailwind CSS Play CDN (keep) | 3.4.17 | Styling | `cdn.tailwindcss.com` currently redirects to `/3.4.17` (verified live) — the app's exact current setup, including `tailwind.config = { darkMode: 'class' }`. The v4 browser runtime (`@tailwindcss/browser`) **cannot read JS configs** (`loadModule` throws; `darkMode: 'class'` must become `@custom-variant dark` in CSS) — migrating Tailwind would change styling behavior, which this milestone must not do. Leave Tailwind untouched. |
| Lucide (keep, PIN from `@latest`) | 1.44.0 | Icons | Official docs still document the UMD global (`lucide.icons`, `lucide.createIcons()`) and now explicitly recommend anchoring the version instead of `@latest`. Pin `unpkg.com/lucide@1.44.0/dist/umd/lucide.min.js` (HEAD-verified 200). The `@lucide/icons` tree-shakable package is for bundler users — irrelevant here. |
| Firebase compat (keep) | 11.6.1 namespaced | Backend (Firestore + anonymous auth) | The namespaced compat API is still shipped (`gstatic.com/firebasejs/12.19.0/firebase-app-compat.js` HEAD-verified 200 — compat survives into v12). Rewriting `db.collection().add()` → modular `collection(db, ...)/addDoc()` is the **single highest behavior-change risk** in this codebase and buys nothing for a decomposition milestone. Keep 11.6.1 exactly as-is. |
| Static server (keep) | Python 3.12 `python -m http.server` | Deployment | Unchanged. LAN: `http://10.0.6.12:8080`. Serves `.js` as `text/javascript` (verified) and transforms `.jsx` only via Babel's XHR path (MIME-independent). |

### Recommended File Layout

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| None new required | — | — | The whole point of this milestone is that the existing CDN set is sufficient. Do not add npm packages. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Editor (VS Code) | Edit `.jsx` files with real syntax highlighting | `.jsx` extension gives JSX highlighting/IntelliSense for free; Python's MIME handling of `.jsx` is irrelevant because Babel loads via XHR |
| Browser DevTools | Verify each split; Network tab shows XHR loads of each `.jsx` | After each file extraction, reload and confirm the app renders identically |
| (optional) `python -m http.server 8080` | Local verification server | Keep using it; do not double-click `index.html` (XHR under `file://` fails CORS) |

## Installation

# Verify after each extraction step (Windows PowerShell)

# open http://localhost:8080 and exercise: submit form, manager passcode, approve, filter, CSV export,

# copy details, dark-mode toggle, return flow

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| External `text/babel` scripts (Babel standalone) | Native ES modules + import maps (esm.sh / jsDelivr +esm) | When a build-free *global-scope* split is insufficient and real module encapsulation is required — but not this milestone: (1) Python 3.12 serves `.jsx` as `application/octet-stream` (verified locally), so `<script type="module" src="app.jsx">` fails; (2) `esm.sh/run` and `esm.sh/tsx` only compile **inline** blocks — verified the 1.7 KB `esm.sh/run` loader contains no `src` handling. Import maps are Baseline (Chrome 89+, Firefox 108+, Safari 16.4+), but they solve npm imports, not local JSX files. |
| No-build classic split | Vite 8 build step | Only when the app actually needs npm dependencies, TypeScript, HMR, or asset pipelines. Vite latest is 8.3.0 (`@vitejs/plugin-react` 6.1.1); requires Node 20.19+/22.12+; every deployment then runs a build and ships `dist/` — directly contradicting the serve-a-folder model and the project's stated constraint. The app is ~500 lines; the fragility being fixed is structure, not tooling. |
| Babel standalone (keep) | `esm.sh/run` / `esm.sh/tsx` replacing it | If the JSX transform were the pain point (it is not — it already works), `/tsx` on **localhost** compiles client-side with `@esm.sh/tsx`; on LAN/non-localhost it POSTs source to esm.sh's build API (60 builds/min limit, third-party internet dependency for code, localStorage caching). Replacing a working local dependency with a remote compile service is a step backward for this deployment. |
| JSX as-is (keep) | Rewrite JSX to `React.createElement` / `htm` | If JSX were dropped, local `.js` files could be native ES modules without a transform. But rewriting ~400 lines of JSX is a massive diff — the exact behavior-change risk this milestone exists to avoid. Rejected. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| React 19 UMD URLs (`unpkg.com/react@19/umd/...`) | React 19 removed UMD builds (official upgrade guide, PR #28735). The files simply do not exist. | Keep React 18.3.1 UMD; if React 19 is ever wanted, load it as ESM from esm.sh (`react@19.3.0`) with an import map — a separate migration. |
| `import` / `export` statements inside the split `.jsx`/`.js` files | Babel finishes the text/babel transform and injects the code as a *classic* script; real ESM syntax throws at parse time. | Cross-file references via shared global scope (topological script order in the HTML). |
| `.jsx` files loaded as `<script type="module">` | Python's `mimetypes` returns no type for `.jsx` → served `application/octet-stream` → browsers refuse to execute a module with a non-JS MIME type (verified locally: `guess_type('x.jsx') = (None, None)`). | `<script type="text/babel" src="js/views/form.jsx">` — Babel XHR-loads the text, transforms, then injects a classic script. MIME never matters. |
| Tailwind v4 `@tailwindcss/browser` (jsDelivr `@4`, latest 4.3.3) | Browser build cannot read `tailwind.config` JS (`loadModule` throws — `darkMode: 'class'` unsupported; requires `@custom-variant dark (&:where(.dark, .dark *));` in a `text/tailwindcss` style block). Migrating silently restyles/light-modes the app. | Keep `https://cdn.tailwindcss.com` (currently serving 3.4.17, verified); treat v4 as a separate styling milestone with its own UAT. |
| `@latest` / range CDN URLs (`lucide@latest`, `react@18`, bare `@babel/standalone/babel.min.js`) | Unpinned ranges drift when a browser/CDN cache is busted — the app's render output can change under you. Lucide's own docs now say: anchor the version. | Exact pins listed above. |
| Opening `index.html` directly (`file://`) | Babel standalone loads `src=` scripts via XHR; `file://` origin blocks XHR (CORS). App silently fails to boot. | Always serve over HTTP (`python -m http.server`) — the existing deployment. |
| Moving `lucide.createIcons()` out of the data-listener `setTimeout` | The `Icon` component renders `<i data-lucide>`; icon SVGs are materialized only when `lucide.createIcons()` re-runs after React commits. "Cleaning this up" breaks icon rendering. | Preserve the existing `setTimeout(() => lucide.createIcons(), 100)` after snapshot updates, exactly as-is. |

## Stack Patterns by Variant

- Use external `text/babel` scripts in dependency order; keep all 19 `useState` hooks and every handler body byte-identical; move code out of `index.html` *without editing it* except removing it from the inline block.
- Because execution is in document order and the global scope is shared, extract bottom-up: constants → utils (pure helpers) → firebase/data (side effects) → views (presentational) → App → main. Every extraction is a pure move, verifiable by reload + the UAT list.
- The project then outgrows the no-build model on its own merits. Migrate to Vite 8 + `@vitejs/plugin-react` 6.x (Node 20.19+), move `js/*.jsx` into `src/`, convert classic-scope splits into real `import/export` modules. Keep Firebase compat (works with Vite; tree-shaking is the only lost bonus) or defer the modular rewrite.
- Move to import maps + esm.sh ESM React 19 + `esm.sh/run` for inline JSX compilation, and rename local files `.js`/`.mjs` (no JSX in local files) or accept the MIME limitation with a tiny custom static server that maps `.jsx` → `text/javascript`. All three constraints are solvable; none are worth solving in this milestone.

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| react@18.3.1 UMD | @babel/standalone@8.0.5 | Babel 8 browser build defaults to the `react` preset with `runtime: 'classic'` (babel changelog #18086) → emits `React.createElement` → matches the app's `const { useState } = React` usage. |
| @babel/standalone@8.0.5 | text/babel `src=` external scripts | Official docs: supported; loaded via XHR; executed in document order (`async` attribute respected; non-async wait their turn). |
| @babel/standalone@8.0.5 | Node tooling | Babel 8 is ESM-only *for Node imports* (requires Node 22/24/26) — does not affect the browser bundle or this app. |
| firebase compat 11.6.1 | gstatic CDN | Head-verified 200. Compat packages continue to publish in v12 (12.19.0 files also 200); no pressure to move. |
| cdn.tailwindcss.com | `tailwind.config = { darkMode: 'class' }` | Redirects to `/3.4.17` (verified); v3 Play CDN reads the inline `tailwind.config` object — current behavior preserved. |
| Import maps (if ever used) | Browsers | Baseline Widely Available: Chrome/Edge 89+, Firefox 108+, Safari 16.4+ (all since March 2023). Inline-only — external `src=` import maps are not in any spec. Multiple import maps: Chromium 133+/WebKit only; keep to one map. |

## Sources

- React 19 Upgrade Guide (react.dev/blog/2024/04/25/react-19-upgrade-guide#umd-builds-removed) + facebook/react PR #28735 + release v19.0.0 — React 19 removed UMD; ESM CDN recommended. Confidence: HIGH (official).
- npm registry `latest` dist-tags queried live 2026-09-11: react 19.3.0, @babel/standalone 8.0.5, firebase 12.19.0, lucide 1.44.0, vite 8.3.0, @vitejs/plugin-react 6.1.1, @tailwindcss/browser 4.3.3. Confidence: HIGH (primary source).
- unpkg + gstatic HEAD checks (all 200, text/javascript) for the pinned URLs listed above; `cdn.tailwindcss.com` → `/3.4.17` redirect observed. Confidence: HIGH (verified live).
- @babel/standalone official docs + `packages/babel-standalone/src/transformScriptTags.ts` (source) — src= XHR loading, in-order execution, runtime:'classic', react+env default presets, production warning. Confidence: HIGH (official).
- Babel 8.0.0 release blog (babeljs.io/blog/2026/06/16/8.0.0) — ESM-only for Node, ES5 default dropped. Confidence: HIGH (official).
- Tailwind CSS docs — Play CDN page (`@tailwindcss/browser@4` snippet), dark-mode page (`@custom-variant dark`), PR #15558 (browser build is IIFE, dev-only); github issue #7081 (v3 CDN needs inline config, not tailwind.config.js). Confidence: HIGH (official + maintainers).
- firebase.google.com/docs/web/alt-setup + /learn-more — modular ESM on gstatic CDN; compat API still listed; compat "does not get new features". Confidence: HIGH (official).
- esm.sh README (v135 release notes: `esm.sh/run` inline-only, 60 builds/min, localStorage+edge caching; `esm.sh/tsx` localhost client-side compile); `esm.sh/run` loader source fetched and confirmed no `src` handling (1,738 bytes). Confidence: HIGH (primary source, verified).
- jsDelivr blog (2026-08-08) — `/+esm` bundling pipeline rebuilt on Rollup 4. Confidence: HIGH (official).
- ESModules.com / MDN / caniuse import-maps — Baseline support matrix, inline-only requirement, single-vs-multiple map engine differences. Confidence: HIGH.
- Lucide getting-started docs — UMD CDN pattern; explicit "anchor to a specific version" guidance. Confidence: HIGH (official).
- Vite 7 announcement (vite.dev, June 2025 — Rolldown preview, Node 20.19+); Vite 8.3.0 current via npm registry. Confidence: HIGH.
- Local verification: Python 3.12.10 `mimetypes` returns no type for `.jsx`/`.tsx` (text/javascript for `.js`); full read of `index.html` (532 lines, single text/babel block, 19 useState hooks). Confidence: HIGH.

<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->

## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->

## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->

## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->

## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:

- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->

## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
