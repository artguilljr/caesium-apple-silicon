# Apple Silicon port changelog

This fork adapts Caesium Image Compressor 2.8.5 for native Apple Silicon macOS. It is an independent port, not an upstream release.

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

