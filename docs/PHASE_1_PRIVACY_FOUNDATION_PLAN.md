# SplitStitch — Phase 1: Privacy-First Foundation & Reliability

**Status:** Planning only — approved for a dedicated branch; implementation not started  
**Date:** 2026-11-17  
**Repository:** https://github.com/LukeHolmes/PDF-Tool  
**Branch:** plan/phase-1-privacy-foundation  
**Base:** main at 15791170328197adfb0017b88faba4e4a039e5a7  
**Owner / execution:** GrokBot coordinates; Cursor or a coding agent implements; human approval for deployment/merge.  
**Scope:** Improve the existing application without adding Phase 2 editor features or deploying changes.

## 1. Objective and product contract

Make SplitStitch a maintainable, tested, offline-capable, genuinely browser-only PDF/image utility. Preserve all working core flows (merge, edit pages, split, image stitch/edit/export) while correcting verifiable defects and removing third-party runtime dependencies.

**Non-negotiable requirements**

1. **No document upload or external processing.** PDF/image bytes and extracted content stay in the browser; no network transfer of user content, filenames, thumbnails, text, or metadata.
2. **Free and self-contained.** No mandatory paid service, account, API key, telemetry service, or PDF backend. Use locally bundled OSS libraries with an auditable licence inventory.
3. **Offline-capable after initial installation/visit.** Essential UI, fonts, scripts, workers, and PDF/image operations work while disconnected. Browser storage eviction and private-browsing restrictions must be documented.
4. **Regression-safe.** Current supported workflows keep working; every corrected defect has an automated regression test.
5. **Static deployment.** Keep compatibility with the existing GitHub Pages project site and its /PDF-Tool/ subpath, without requiring a runtime server.
6. **Incremental, reviewable changes.** Small commits/PRs, no unexplained rewrite, and no merge or deployment without explicit approval.

**Out of scope for Phase 1:** new text/annotation/signature editor, OCR, PDF compression/encryption engine, advanced redaction, DOCX conversion, login, cloud storage, billing, analytics, paid APIs, backend, major visual redesign. Encrypted files should fail safely and clearly until a proper solution is explicitly approved.

## 2. Verified baseline (static inspection, not a runtime audit)

- Repository has two tracked files: README.md and index.html; no package manifest or checked-in test/CI infrastructure.
- index.html is approximately 2,367 lines of HTML, CSS, and vanilla JavaScript.
- Uses pdf-lib 1.17.1, PDF.js 3.11.174 and its worker, jsPDF 2.5.1, Cropper.js 1.6.1, Tailwind Play CDN, and Google Fonts via external network resources.
- Code contains handlers for PDF merging, splitting, page extraction/reorder/rotation/deletion/insertion, undo/redo, thumbnails and zoom; image stitching, crop, brightness and image/PDF export are also implemented.
- No explicit fetch/XMLHttpRequest/sendBeacon/WebSocket call was identified in the app source. This **does not prove** the absence of network activity from third-party scripts or browser facilities.
- The bookmark split path calls \`currentPdfSplitDoc.getOutlines()\`, although pdf-lib's published PDFDocument API does not provide that method. PDF.js provides \`getOutline()\`; confirm and fix this defect with fixtures.
- Several PDF loads pass \`ignoreEncryption: true\`; pdf-lib explicitly warns this does not decrypt encrypted PDFs.
- Merge drag/drop ordering maps selected file entries back by filename, creating an ambiguity when different files share the same name.
- The Add Blank Page handler draws the visible text "(Blank Page)" onto the inserted page.
- Single-page delete/rotate paths should be audited for consistent undo/redo history capture.
- Code creates object URLs for downloads; lifecycle and cleanup should be reviewed to prevent avoidable memory retention.
- README still refers to "Integrated.html" although the deployed entry file is "index.html".

References:
- Repository source: https://github.com/LukeHolmes/PDF-Tool/blob/main/index.html
- pdf-lib encryption limitation: https://github.com/Hopding/pdf-lib
- PDF.js \`getOutline()\` API: https://mozilla.github.io/pdf.js/api/draft/module-pdfjsLib-PDFDocumentProxy.html
- Tailwind Play CDN limitation: https://tailwindcss.com/docs/installation/play-cdn

## 3. Delivery sequence and work packages

Complete work packages in order; a later task should not destabilise earlier tests. Estimates are focused engineering effort, not a calendar commitment; they exclude unforeseen PDF compatibility defects.

### P1-00 — Freeze baseline, inventory and risks (0.5–1 day)

**Tasks**
- [ ] Record current branch head, deployment URL, runtime versions, dependencies, and observable behaviour.
- [ ] Record existing UI flows and output filenames in a short feature-parity matrix (merge, edit, split, stitch, convert).
- [ ] Capture baseline screenshots/video and 6–10 representative synthetic PDF/image workflows with expected outputs.
- [ ] Build a dependency, source, licence and outbound-request inventory; identify runtime requests to CDNs, fonts and worker scripts.
- [ ] Reproduce and log suspected bugs before fixing them; distinguish confirmed failures from static-review hypotheses.
- [ ] Create a risk/decision log (security, compatibility, performance, legal/licensing, offline support).

**Deliverable:** docs/phase-1/BASELINE.md and docs/phase-1/RISKS.md.

**Gate:** Every existing user workflow is enumerated and every proposed defect has a reproduction or is marked "requires verification".

### P1-01 — Minimal, reproducible build system (1–2 days)

**Tasks**
- [ ] Keep vanilla JS and static hosting. Add package.json, a pinned package-lock.json and documented Node LTS requirement.
- [ ] Choose a lightweight static bundler (Vite is preferred); configure relative URLs and GitHub Pages /PDF-Tool/ base path.
- [ ] Move current inline JS/CSS into source assets incrementally; use imports rather than global CDN scripts.
- [ ] Compile Tailwind at build time, replacing the Play CDN without changing the existing UI styling; retain compatibility while refactoring.
- [ ] Bundle pdf-lib, PDF.js, the matching worker, jsPDF, Cropper.js, styles and fonts locally. Pin compatible versions; log intentional upgrades separately.
- [ ] Add reproducible commands: npm ci, npm run dev, npm run build, npm run preview, npm run test, npm run test:e2e, npm run lint (or equivalent). Avoid a framework migration.
- [ ] Add .gitignore, README setup instructions, build-output policy, dependency licence inventory and GitHub Pages deployment notes.
- [ ] Verify clean checkout builds offline **after dependencies are installed**; distinguish developer build prerequisites from end-user offline use.

**Deliverable:** deterministic static build, locally served assets, no production runtime CDN calls.

**Gate:** A clean environment can build and preview the site; output works under both / and /PDF-Tool/ without broken assets or workers; baseline user flows retain parity.

### P1-02 — Automated regression safety net (1.5–3 days)

**Tasks**
- [ ] Add Vitest or similar free test runner for pure logic and Playwright for browser E2E.
- [ ] Create only non-sensitive, synthetic fixtures: 1/2/20/100-page PDFs, mixed sizes/orientations, bookmarks, nested bookmarks, duplicate names, images, malformed PDF, encrypted PDF, scanned/image-only page, special characters in filenames.
- [ ] Build deterministic fixture generator and document expected characteristics, e.g. page count, rotations, first/last page text and file sizes.
- [ ] Cover importing, merging in specified order, splitting each/X/ranges/bookmarks, page selection, reordering, rotation, extraction, insertion, image stitching and image-to-PDF export.
- [ ] Assert PDF structure and page order, not only "download button appeared"; open generated outputs again using an independent parser/rendering check when practical.
- [ ] Add undo/redo invariants and error-recovery cases; include empty selection and no-file states.
- [ ] Add baseline visual snapshots with stable viewport/font setup, tolerance for rasterisation differences and accessibility smoke checks.
- [ ] Add GitHub Actions for install, lint, unit tests, static build, and headless Chromium E2E; add Firefox/WebKit coverage where stable/feasible.
- [ ] Ensure tests never use private documents or upload test fixture contents to external telemetry.

**Deliverable:** tests/, e2e/, fixture generation, and repeatable CI logs.

**Gate:** Existing functionality is protected by automated tests; all critical workflows pass in CI on a clean checkout.

### P1-03 — Modularise without redesign (2–4 days)

**Tasks**
- [ ] Extract app bootstrap, navigation, status/progress, confirmation dialogs, download helpers, file validation and shared UI components.
- [ ] Extract PDF operations into clear modules: merge, split, page edit, renderer, selection/order, history, bookmark resolver.
- [ ] Extract image workflows into modules: import/state, transform/crop/brightness, stitch/preview, export.
- [ ] Introduce well-defined interfaces for current document bytes, rendering instances, output blobs and cleanup.
- [ ] Keep document data in memory by default; do not add persistence of loaded files to localStorage, IndexedDB, caches, telemetry, logs, or service workers.
- [ ] Ensure each extracted workflow continues to use the same UI and the same semantics unless a verified defect is being corrected.
- [ ] Introduce basic linting/formatting (ESLint/Prettier or equivalents) and remove unused state/dead handlers.
- [ ] Avoid broad asynchronous rewrites, state-library migrations and visual changes in this work package.

**Suggested layout** (refine based on actual refactor): 

    index.html
    src/main.js
    src/pdf/{merge,split,edit,bookmarks,render,history}.js
    src/image/{state,edit,stitch,export}.js
    src/ui/{tabs,feedback,dialogs}.js
    src/utils/{files,downloads,validation}.js
    src/styles.css
    tests/   e2e/   public/   docs/phase-1/

**Deliverable:** smaller, independently testable modules with no functional regression.

**Gate:** No circular dependencies or duplicate state owners; UI parity and PDF E2E tests pass after each extraction.

### P1-04 — Enforce privacy and offline operation (1–2.5 days)

**Tasks**
- [ ] Remove production runtime dependencies on CDNs, hosted fonts, remote scripts/styles, analytics and error-reporting services.
- [ ] Publish an explicit data-flow/privacy model: user file -> in-memory parser/editor -> local Blob/download; no document transfer.
- [ ] Add a sensible production CSP and other feasible static-site controls; prefer external modules over inline scripts, and document limitations of CSP via HTML meta on GitHub Pages.
- [ ] Add automated request monitoring that fails if the app requests any third-party resource during normal load/edit/export; verify no file content is sent anywhere. Distinguish first-party app asset downloads from user-document uploads.
- [ ] Add optional **static-asset-only** service worker precaching for offline second visits (including PDF.js worker, CSS, fonts and JS). Never cache user PDFs/images or generated outputs. Ensure versioned updates and reliable stale-cache cleanup.
- [ ] Test offline browser refresh after first visit, all baseline tools offline, and first-visit failure messaging when the app isn't installed/cached.
- [ ] Audit DOM injection, unsafe innerHTML, links, filename handling, object-URL revocation, raw console output, and accidental persistence of sensitive filenames/content.
- [ ] Correct privacy copy: claim only what can be demonstrated; note site requests may be needed to load the application initially, and browser cache eviction/private mode can impact offline use.
- [ ] Document that browser extensions, compromised devices and third-party systems are outside the application's direct control.

**Deliverable:** docs/phase-1/PRIVACY_THREAT_MODEL.md, offline support and verified no-document-egress tests.

**Gate:** No external runtime requests, no user document egress in monitored tests, user files excluded from service-worker cache, and supported workflows pass offline after installation.

### P1-05 — Repair verified reliability issues (1–3 days)

**Required investigations / fixes**
- [ ] **Bookmark split:** replace unsupported pdf-lib \`getOutlines()\` path with PDF.js \`getOutline()\` plus destination resolution; validate named destinations, top-level bookmarks, duplicates, bookmarks that start on page 1, malformed destinations and correct section boundaries. Clearly define whether content before the first bookmark is included.
- [ ] **Encrypted files:** remove use of \`ignoreEncryption: true\` as a decryption workaround; detect unsupported encrypted inputs, display clear non-destructive error messages and avoid half-processed outputs. Do not implement decryption in Phase 1.
- [ ] **Duplicate filenames:** store stable per-selection identifiers and derive merge order from those IDs, not filename text; verify reordering same-named PDFs merges correct content.
- [ ] **Blank page:** insert a genuinely blank page, without "(Blank Page)" text.
- [ ] **Undo/redo:** verify and, if needed, fix history capture for single-page delete/rotate, batch operations and failed operations; avoid snapshots for no-op actions.
- [ ] **Memory/download cleanup:** revoke obsolete Blob URLs, release replaced PDF.js document instances and limit thumbnail/canvas lifetimes; preserve successful downloads.
- [ ] **Failure recovery:** invalid/malformed inputs, unsupported PDFs, oversized inputs and interrupted operations should not leave disabled controls, stale previews or corrupted state.
- [ ] **README drift:** update entry-file references and usage instructions to match actual deployment.

**Deliverable:** issue-by-issue evidence and regression tests for every confirmed defect.

**Gate:** Each issue has a failing pre-fix reproduction and passing post-fix regression or a documented finding explaining why no fix was necessary.

### P1-06 — Cross-browser QA, accessibility and performance (1.5–3 days)

**Tasks**
- [ ] Validate latest supported Chrome, Firefox and Edge; Safari/WebKit smoke checks if accessible.
- [ ] Test desktop and narrow mobile viewports; keyboard focus, dialogs, labels, file drop areas, selection controls and screen-reader semantics.
- [ ] Validate 1/20/100-page synthetic PDFs and a moderately large PDF using a fixed machine; measure time to first preview, merge/split/export elapsed time, peak memory and JS errors.
- [ ] Avoid unbounded rendering of every page at once; consider lazy thumbnails and cancellation if measured bottlenecks justify changes.
- [ ] Validate exported PDFs reopen, retain expected page order, rotations, readable content and unaffected metadata/forms/annotations where applicable. Log known limitations separately.
- [ ] Perform a network audit both online and offline using browser instrumentation; verify no third-party requests from normal application operation.
- [ ] Perform dependency/licence review and vulnerability scan; record mitigations and unresolved findings.
- [ ] Create an accessible manual regression checklist for behaviours not fully automated.

**Deliverable:** docs/phase-1/QA_REPORT.md with metrics, evidence, defects and pass/fail status.

**Gate:** No known severity-1/severity-2 regressions; all critical flows pass; performance regressions are quantified, explained or fixed.

### P1-07 — Release readiness and handoff (0.5–1 day)

**Tasks**
- [ ] Update README with local install, build, offline behaviour, privacy model, limitations and browser compatibility.
- [ ] Add CHANGELOG entry and a concise architecture diagram.
- [ ] Produce release checklist and QA evidence links; provide a concise executive summary of what changed and what remains.
- [ ] Open a draft PR from the implementation branch to main **only if explicitly approved**; do not automatically merge, change GitHub Pages, publish or deploy.
- [ ] List deferred Phase 2 features and lessons learned with priority estimates.

**Deliverable:** release-ready branch plus explicit go/no-go decision summary.

**Gate:** Owner reviews tests, privacy proof, risks and deployment plan before any merge/deployment.

## 4. Acceptance criteria / definition of done

The branch is **Phase 1 complete** only if all are true:

- [ ] All previously supported PDF/image workflows pass their regression checks, aside from explicitly approved defect corrections.
- [ ] All runtime JS, CSS, font files, libraries and PDF workers are served locally from the application origin.
- [ ] Browser tests demonstrate **zero document data uploads** and no third-party network requests during normal use.
- [ ] After one successful connected visit, a previously cached installation loads and performs all core actions while offline (under stated browser-storage conditions).
- [ ] A fresh checkout can reproduce its build using documented Node/npm versions and locked dependencies.
- [ ] Bookmark splitting, encrypted-file rejection, duplicate-filename merge order, genuine blank-page creation, and undo/redo have documented regression outcomes.
- [ ] Generated PDFs are checked for semantic correctness (page order/count/content, not just file existence).
- [ ] CI executes successfully with unit tests, browser tests and production build; no critical failures.
- [ ] Accessibility and cross-browser smoke tests are recorded; outstanding non-critical limitations are disclosed.
- [ ] The privacy statement is accurate, evidence-backed and avoids absolute guarantees outside the app's control.
- [ ] The owner has reviewed the change summary; main is unchanged until separately approved.

## 5. Test scenario matrix (minimum functional set)

| ID | Scenario | Expected result |
| --- | --- | --- |
| T01 | Merge three PDFs with distinct page markers | Exactly ordered pages and correct count |
| T02 | Merge two files with identical filename and different content | Preserves explicitly chosen order and content |
| T03 | Split each page and every N pages | Correct number of files, page boundaries and content |
| T04 | Split overlapping/custom ranges | Correct independent results and clear validation |
| T05 | Split top-level bookmarks with first bookmark on p1 | Correct partitions, no empty/off-by-one outputs |
| T06 | Split top-level bookmarks starting after p1 | Defined treatment of pre-bookmark pages; no lost content |
| T07 | Edit mixed rotations and page sizes | Correct page order and orientation after export |
| T08 | Delete single page, undo, redo | Exact expected page states |
| T09 | Insert blank page | Page contains no "(Blank Page)" content |
| T10 | Insert pages from another PDF | Correct insertion index and preserved content |
| T11 | Image crop/brightness/stitch and PNG/JPEG output | Correct orientation, dimensions and output type |
| T12 | Image -> PDF output | Reopens with expected pages and dimensions |
| T13 | Encrypted PDF | Explicit unsupported-file warning, no incorrect output |
| T14 | Corrupt or malformed PDF | Recoverable error, UI remains usable |
| T15 | Offline first visited app | Interface and essential operations run without a network |
| T16 | Privacy check on load, edit, export | Zero document upload; no third-party network request |
| T17 | Firefox/Chromium, wide/narrow viewports | Core workflows usable and no layout blocker |
| T18 | Undo, redo and repeated file reload | No stale state or unexpected output; downloads remain valid |

Tests must use only generated, non-sensitive fixtures. Set exact fixture counts, checksums and acceptance assertions in test code.

## 6. Suggested commit / PR checkpoints

1. **Checkpoint A — Baseline & CI:** inventory, reproducible fixtures, safety-net tests (no user-facing changes).
2. **Checkpoint B — Build & assets:** package/lockfile, compiled styles and locally bundled libraries (visual parity).
3. **Checkpoint C — Extract modules:** incremental refactor into PDF/image/UI/utilities modules.
4. **Checkpoint D — Privacy & offline:** CSP, static precache, request auditing and documentation.
5. **Checkpoint E — Defect fixes:** bookmarks, encrypted handling, merge order, blank-page and undo/redo as needed.
6. **Checkpoint F — Release assessment:** cross-browser, performance, accessibility, verification report.

Prefer small independent commits and green CI at every checkpoint. If CI fails, halt subsequent work and isolate regression before continuing.

## 7. Risks and decisions

| Risk | Mitigation |
| --- | --- |
| PDF rendering version mismatch | Keep PDF.js and worker versions identical; verify all assets in production build |
| Refactor breaks UX or file export | Baseline screenshots, scenario fixtures, incremental modules and E2E tests |
| Offline caching accidentally stores private PDFs | Precache explicit immutable static-asset allowlist only; test Cache Storage contents |
| CDN removal changes typography/layout | Bundle local font, compare screenshots and visual snapshots |
| Browser memory limit with big PDFs | Measure memory, release stale canvases/documents/Blob URLs, render lazily if required |
| Bookmark semantics differ among PDFs | Test named/explicit destinations and corrupted structures; disclose unsupported cases |
| CSP blocks PDF.js workers/Blob downloads | Verify exact worker-src and blob: requirements under GitHub Pages static constraints |
| Dependency/licence violations | Maintain licence inventory and document notices; audit before publication |
| Overengineered build complexity | Keep vanilla JS and static output; no framework/backend migration |

**Decision rule:** Privacy, correctness, preservation of existing workflows, and reproducibility take precedence over speed or additional features.

## 8. Effort, ownership and execution protocol

**Working estimate:** roughly 8–15 focused engineering days, with meaningful uncertainty from PDF compatibility and browser QA. This is **not** a delivery promise. GrokBot/Cursor can automate much of implementation and test execution; the owner should make product trade-offs, inspect QA results and approve releases.

**Agent operating instructions**
1. Read this plan plus README and index.html; report branch/head before changes.
2. Work only on an implementation branch derived from plan/phase-1-privacy-foundation, or continue the planning branch with owner permission; never write to main.
3. For each work package, produce an evidence-based task breakdown, run tests and commit a small, reviewable change.
4. Keep a running work log with confirmed defects, tests, risks, commit SHA, next actions and unresolved questions.
5. Do not introduce external APIs, paid dependencies, file uploads, telemetry, accounts or backend services.
6. Never store customer PDFs in the repository or CI artifacts; use synthetic fixtures only.
7. Request owner approval for design deviations, third-party licence changes, merging and deployment.
8. Stop if a potential sensitive-file upload or major output-integrity regression is discovered.

**Next authorised action after this planning commit:** Start P1-00 on this branch or a new implementation branch following the owner's instruction; no Phase 1 implementation has yet been authorised by the planning document alone.
