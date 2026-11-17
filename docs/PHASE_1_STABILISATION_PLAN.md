# SplitStitch — Phase 1: Stabilisation & Reliability Plan

**Status:** Planning only; not commissioned or implemented  
**Date:** 2026-11-17  
**Repository:** https://github.com/LukeHolmes/PDF-Tool  
**Planning branch:** plan/phase-1-privacy-foundation  
**Related document:** [Phase 1 Privacy-First Foundation Plan](./PHASE_1_PRIVACY_FOUNDATION_PLAN.md)  
**Estimated engineering effort:** 25–50 focused hours with AI assistance (indicative, not a delivery commitment)  
**Scope:** Stabilise the existing PDF and image utility. No new product features, broad visual redesign, merge, or deployment.

> This document records the stabilisation roadmap agreed in the November 17 planning discussion. The existing privacy-foundation plan is preserved separately. Before engineering begins, reconcile their sequencing, time estimates, and release criteria into one execution backlog. No work is authorised merely by saving this plan.

## 1. Objective

Transform SplitStitch from a functional prototype into a dependable, modular, tested, privacy-first application before undertaking Phase 2 product expansion.

The Phase 1 outcome must be:
- All supported merge, page edit, split, image stitch, image adjustment, and image/PDF export workflows remain functional.
- Reproducible correctness defects are fixed and protected by automated regression tests.
- The application can be built reproducibly and tested from a clean checkout.
- Large-document behaviour and resource use are measured, with targeted improvements where justified.
- Dependencies are pinned and bundled, and client-side privacy claims are tested.
- A review-ready branch and evidence-based release report are available, without automatic merge or deployment.

### Non-goals

Do **not** introduce OCR, text editing, a new annotation/signature editor, document comparison, cloud accounts, telemetry, billing, backends, workflow automation, or a major visual redesign in Phase 1. Do not claim validation or GxP compliance.

## 2. Work-package overview

| Package | Purpose | Priority | Indicative effort |
|---|---|---|---|
| WP1 | Baseline audit and development safeguards | P0 | 2–4 h |
| WP2 | Automated testing infrastructure | P0 | 5–8 h |
| WP3 | Correctness and reliability fixes | P0 | 8–14 h |
| WP4 | Memory and performance improvements | P1 | 4–10 h |
| WP5 | Dependency and privacy hardening | P1 | 3–7 h |
| WP6 | Regression verification and release report | P0 | 3–7 h |
| **Total** | | | **25–50 h** |

Estimates are planning ranges, not measured delivery times. If defects require deeper changes, re-estimate before extending scope.

## 3. Detailed work packages

### WP1 — Establish protected baseline (2–4 h, P0)

**Tasks**
- [ ] Record the exact main commit SHA, application URL, deployment configuration, supported browsers, and dependency versions.
- [ ] Inspect existing branches and pull requests; do not silently include unmerged work.
- [ ] Establish an isolated stabilisation implementation branch *when engineering is authorised*. The present planning branch is documentation-only.
- [ ] Inventory PDF merge, page editing, splitting, image editing/stitching, image-to-PDF and export flows.
- [ ] Capture reproducible user journeys, screenshots, sample outputs, and baseline behaviour.
- [ ] Maintain a defect register: severity, exact reproduction steps, expected/actual result, evidence, status, and proposed fix.
- [ ] Benchmark initial memory, rendering and export performance on synthetic documents.
- [ ] Separate confirmed runtime defects from static-review hypotheses.

**Deliverables:** baseline audit, feature-parity matrix, defect register, performance baseline.

**Acceptance:** Every supported workflow is inventoried. Each proposed defect has evidence or is explicitly marked unverified. Original app state is recoverable.

### WP2 — Introduce automated tests (5–8 h, P0)

**Tasks**
- [ ] Establish the minimal reproducible package/build environment; use Vite + TypeScript only where migration risk is acceptable.
- [ ] Use incremental extraction, not an uncontrolled rewrite of the monolithic index.html.
- [ ] Add Vitest for independently testable processing utilities.
- [ ] Add Playwright for end-to-end browser tests.
- [ ] Create deterministic, non-sensitive PDF/image fixture generators.
- [ ] Introduce assertions for real file output: page count, order, rotation, selected page content, dimensions and valid downloads.
- [ ] Add tests for corrupt inputs, duplicate names, empty selections, file limits, and undo/redo invariants.
- [ ] Configure GitHub Actions for clean installation, build, unit tests and browser smoke tests.
- [ ] Define test commands and developer setup in the README.

**Deliverables:** test suite, reusable fixtures, CI workflow, reproducible build instructions.

**Acceptance:** Critical baseline workflows run automatically locally and in GitHub Actions. Proposed regressions can block a merge.

### WP3 — Correct confirmed defects (8–14 h, P0)

Investigate first and repair only defects that are reproduced or otherwise technically established.

1. **Bookmark splitting:** the current path calls getOutlines() on a pdf-lib document, which is not part of the documented pdf-lib PDFDocument API. Evaluate PDF.js getOutline() and destination resolution. Cover top-level bookmarks, page boundaries, duplicate destinations, named destinations, and unbookmarked documents.
2. **Initial undo state:** verify whether opening a PDF incorrectly enables Undo before any user operation. Make history semantics consistent.
3. **Repeated thumbnail listeners:** stop dragover/drop handlers being registered repeatedly during re-render. Verify a document can be edited and reloaded without listener accumulation.
4. **Merge ordering:** remove dependence on parsing displayed filenames to identify underlying file objects. Use stable IDs/references and cover duplicate filenames.
5. **Blank-page insertion:** truly insert a blank page; do not draw the literal '(Blank Page)' label unless the user explicitly requested it.
6. **Image-to-PDF scaling:** validate page dimensions, aspect ratios, pagination and clipping for tall, wide, portrait and landscape images. Repair the conversion math only after comparison against expected layouts.
7. **Error recovery:** test partially failed PDF operations, save failures, undo/redo rollback, and invalid input handling.
8. **Object URL lifecycle:** review creation, use and revocation to avoid retained resources.

**Deliverables:** focused commits for each verified defect and associated regression tests.

**Acceptance:** Every fixed defect has a failing-before/passing-after test. All established critical defects are resolved or explicitly documented as blockers.

### WP4 — Improve performance and resource management (4–10 h, P1)

**Tasks**
- [ ] Benchmark document loads with 10, 50, 100 and 250 pages.
- [ ] Measure first preview duration, thumbnail rendering duration, memory peak, scrolling responsiveness, edit operation duration and export time.
- [ ] Introduce bounded thumbnail concurrency and lazy/virtual rendering only where results demonstrate a benefit.
- [ ] Avoid unnecessary whole-document parsing and re-rendering after simple operations.
- [ ] Cancel stale render jobs and clean up PDF.js instances, canvases, ImageBitmaps and object URLs.
- [ ] Investigate the memory impact of full PDF byte snapshots used for undo.
- [ ] Preserve responsive progress display and add cancellation safely where feasible.
- [ ] Record before/after benchmark numbers, test device and methodology.

**Deliverables:** targeted performance changes and benchmark report.

**Acceptance:** No material performance regression; identified bottlenecks show evidence-backed improvement, and repeated operations do not exhibit uncontrolled resource growth.

### WP5 — Harden security and privacy (3–7 h, P1)

**Tasks**
- [ ] Replace runtime CDN imports with pinned and locally bundled application dependencies.
- [ ] Inventory dependency versions, known vulnerabilities and relevant licences.
- [ ] Review file validation, sanitisation, dynamically inserted HTML, and browser download handling.
- [ ] Avoid document bytes, text and sensitive metadata reaching external endpoints or logs.
- [ ] Test outbound network activity during load, edit, stitch and export; distinguish downloading app assets from transferring user documents.
- [ ] Introduce suitable Content Security Policy controls compatible with GitHub Pages.
- [ ] Verify the supported workflows function after the app is installed/cached and the network is disconnected.
- [ ] Document the limits of privacy/offline guarantees, including browser extensions and cache eviction.

**Deliverables:** dependency inventory, privacy/data-flow assessment, offline and outbound-request test results.

**Acceptance:** No unintended user-document egress in monitored tests; supported workflows function offline under documented conditions; privacy claims match the evidence.

## 4. Regression test matrix

| Area | Minimum test scenarios |
|---|---|
| PDF merge | Order, duplicates with same filename, different sizes, malformed source |
| PDF split | One page per PDF, every N pages, explicit ranges, top-level bookmarks, no bookmarks |
| PDF page edit | Insert, remove, rotate, extract, reorder, save, reload and inspect |
| Undo/redo | Initial state, multiple actions, undo/redo boundaries, new edit after undo |
| Images | Crop, rotation, brightness, vertical/horizontal stitch, spacing, watermark |
| Image-to-PDF | A4, Letter, portrait, landscape, very tall/wide image, pagination and clipping |
| Failure handling | Corrupt/encrypted/unsupported files, oversized images, empty selections |
| Resources | Repeated opening/closing, cancellation, large-document operations |
| Privacy | Offline execution, outbound network monitoring, no document-content transmission |

Tests must examine the actual exported bytes and document contents wherever possible, rather than relying only on UI success messages.

## 5. Performance benchmark methodology

Use synthetic fixtures sized **10, 50, 100 and 250 pages** with a documented mix of content and dimensions.

- **10 pages:** check fast interactive loading and basic edits.
- **50 pages:** typical PDF workflow, responsive scrolling and export.
- **100 pages:** validate bounded memory and progressive rendering.
- **250 pages:** stress test for freezing, leaks, and cancellation/recovery.

For each run, log document hash and characteristics, browser/device, build/commit SHA, operation, duration, memory where observable, output validation, and errors. Use measured baselines to set release thresholds; do not invent target timings.

## 6. Release gates

- [ ] **Gate 1 — Baseline protected:** source/behaviour documented and suspected defects triaged.
- [ ] **Gate 2 — Testing operational:** deterministic fixtures and automated critical-flow tests run locally and in CI.
- [ ] **Gate 3 — Critical correctness fixed:** all verified critical issues resolved, with regression tests.
- [ ] **Gate 4 — Performance/privacy evidence:** benchmark, resource cleanup, offline and network checks documented.
- [ ] **Gate 5 — Release candidate reviewed:** regression suite green, limitations logged, candidate branch ready for human approval.

**Completion standard:** Supported operations are reliable, regressions are detectable, and remaining limitations are documented. Passing gates does not authorise deployment.

## 7. Ownership and agent delegation

**GrokBot — coordinator**
- Maintain work-package status, evidence links, exact commits and test outcomes.
- Delegate narrowly scoped tasks to Cursor.
- Prefer scripts and CI artefacts over repeated repository-wide AI analysis.
- Report blockages and requests for decisions, not unsupported completion claims.

**Cursor — engineering**
- Implement isolated changes and focused tests.
- Run fixture generation and regression checks.
- Produce reviewable diffs and short evidence summaries.
- Avoid unapproved scope expansion, merges or deployments.

**Human owner — review/approval**
- Approve architecture and scope changes.
- Review critical defect evidence and relevant UX decisions.
- Authorise merges, publication and release separately.

### Suggested commissioning instruction

> Audit the current LukeHolmes/PDF-Tool main branch and inspect unmerged work. Preserve a verifiable baseline, then implement the six work packages above incrementally. Establish a lightweight automated test suite and reproducible build. Verify and repair bookmark splitting, undo behaviour, duplicate listeners, merge ordering, blank-page insertion, image-to-PDF scaling and error recovery. Benchmark 10/50/100/250-page fixtures; improve resource handling based on evidence. Bundle and pin dependencies, verify no unintended document egress, and test offline operation. Record exact commits, before/after observations, test results, residual risk and release recommendations. Keep the work separate from main. Do not merge or deploy without approval. First deliverable: baseline audit and evidence-backed defect register, before substantial refactoring.

## 8. Execution order and reporting

1. Complete WP1 and obtain approval for the implementation order.
2. Establish the WP2 testing foundation before risky extraction/refactoring.
3. Fix verified WP3 correctness defects in small commits.
4. Measure and address WP4 performance bottlenecks.
5. Complete WP5 dependency/privacy checks.
6. Execute WP6 final verification and report.

### WP6 — Final verification and release report (3–7 h, P0)

- [ ] Run the complete automated regression suite.
- [ ] Check Chromium, Firefox and WebKit/Safari where feasible; document any excluded platforms.
- [ ] Inspect desktop/mobile layout behaviour and keyboard/accessibility basics.
- [ ] Re-run benchmarks against the original baseline.
- [ ] Produce a concise decision report: fixes, tests, outputs, measurements, known defects, residual risks, recommended next steps.
- [ ] Confirm no unrelated files or features slipped into the change set.
- [ ] Present release candidate for approval; do not merge or deploy.

**Final deliverable:** A tested stabilisation candidate, complete test evidence, and an explicit go/no-go recommendation.

---

**Repository change represented by this document:** Planning record only. No feature work, test execution, branch merge or deployment is implied.
