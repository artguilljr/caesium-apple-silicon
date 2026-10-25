# Apple Silicon port changelog

This fork adapts Caesium Image Compressor 2.8.5 for native Apple Silicon macOS. It is an independent port, not an upstream release.

## 2026-10-25 — Preview responsiveness fix (2.8.5-mac.3)

### Evidence and changes
- A process sample from the previous diagnostic build showed the main thread blocked in `imageList_selectionChanged -> previewImage -> QFutureInterfaceBase::waitForFinished`. Compression workers were still in file reads/writes during that sample.
- Remove the synchronous wait from preview replacement on the GUI thread.
- Defer new previews while compression is active; the normal completion path refreshes the selected preview afterward.
- Coalesce pending preview replacements to the latest request and retain a persistent model index so a removed row cannot be reused accidentally.
- Preserve the previous completion diagnostics and compression algorithms.

### Validation
- ARM64 macOS build, existing importer tests, and packaged launch/signature checks passed.
- Watchdog-backed tests verified that selection during compression and rapid preview replacement return promptly with unfinished preview futures.
- The synthetic 80,026-entry completion tests still passed.
- Build: https://github.com/artguilljr/caesium-apple-silicon/actions/runs/40218765606

### Limits
This addresses the GUI blocking wait observed in the process sample. It does not establish why file I/O was slow, reproduce the separate post-completion exit, or validate the user's actual image collection. Confirmation on the affected workload is still needed.

## 2026-10-25 — Completion diagnostic build (2.8.5-mac.2)

### Changes
- Replace the blocking completion summary with a parented, nonblocking dialog.
- Use an in-app completion message on macOS instead of the native system-tray notification while investigating unexpected exits.
- Replace per-row completion refreshes with one model-range update for large batches.
- Defer preview refresh and optional usage reporting until after the completion handler returns.
- Guard percentage calculations when the original total size is zero.
- Add local completion-stage log messages to identify where an unexpected exit occurs.

### Validation
- Built on an ARM64 macOS 15 GitHub-hosted runner.
- Existing importer tests passed.
- A synthetic 80,026-entry completion test verified summary visibility, parent ownership, main-window survival after dismissal, repeat completion, the skip-dialog preference, and a single range refresh.
- Packaged app launch, ARM64 checks, and local code-signature verification passed.
- Build: https://github.com/artguilljr/caesium-apple-silicon/actions/runs/40215608598

### Remaining uncertainty
The exact cause of the reported post-compression exit has not been reproduced or confirmed. This is a diagnostic mitigation, not a verified fix for every unexpected exit. The synthetic completion test did not recompress 80,026 real images. Confirmation on an affected workload is still needed.

## 2026-10-19 — Initial Apple Silicon CPU build
- Add a native ARM64 macOS build and standalone ZIP/DMG packaging.
- Correct the macOS compression-library output declaration from `.so` to `.dylib` during the build.
- Set the deployment target to macOS 12 and correct framework runtime-linker flags.
- Bundle Qt, Sparkle, and libcaesium; apply a local ad-hoc signature.
- Preserve the existing interface and compression engine, including CPU multithreading.
- Existing importer tests and a packaged launch check passed.
- Build: https://github.com/artguilljr/caesium-apple-silicon/actions/runs/39353272299

## Build and distribution notes
- Port changes currently reside in `.github/workflows/build-qt.yml`. The workflow applies the source/build patches before compiling; the corresponding C++ files on the branch do not yet contain those patches directly. A plain local checkout build does not automatically apply them.
- No Metal/GPU compression has been implemented. No comparative performance gains have been established.
- Installers are ad-hoc signed, not Developer ID signed or Apple-notarized.
- Finished installers are Actions artifacts, not published GitHub Releases. Artifact retention limits apply.


