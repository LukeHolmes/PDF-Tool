---
name: local-privacy-regression
description: Check SplitStitch PDF/image workflows for accidental remote data exposure and output regressions using synthetic browser fixtures.
---

# SplitStitch privacy and regression

1. Inspect the current source and scripts to identify supported operations and actual test commands; do not rely on stale README behaviour.
2. Build only synthetic files containing non-sensitive text/images and exercise merge, split, rotate, reorder, insert, extract, undo/redo and export where supported.
3. Observe browser network requests during import, preview, processing and export. Third-party library CDN loading is distinct from transmitting user documents; assert that no file bytes, thumbnails or extracted text leave the browser.
4. Compare output page count, order, orientation and readable content against expected fixtures. Check password-protected/corrupt/large-file error handling as supported.
5. Check cleanup of object URLs and generated previews, repeat operations, keyboard support and mobile/desktop interactions.
6. Capture script output, minimal screenshots, network evidence, browser limitations and any missing automated coverage. Recommend narrow regressions before architecture changes.
7. Work on a branch and request approval before any change to privacy boundary, cloud processing, merges or deployment.
