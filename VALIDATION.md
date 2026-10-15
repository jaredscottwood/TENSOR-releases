# TENSOR 2.2.0 — public UI and run-log validation

This release starts from validated public TENSOR 2.1.0. It changes header/footer text, the shared bottom Run log panel, search activity logging and the conformational-viewer toolbar. Scientific kernels/defaults, SSH transfer behavior, artwork and pinned dependency versions are retained. The prior scientific and loopback-SSH validation is preserved in VALIDATION-2.1.0.md.

## Current checks

Validated on 2026-10-15 UTC on Linux x86-64 with Rust 1.95.0 and the unchanged resolved dependency versions.

- The optimized release build and release regression suite pass. There are **157 regular tests**: 63 library, 19 audit, 7 conformer-pipeline, 23 regression, 22 spectrum, 4 update-specific and 19 inherited Xsampler checks. The four opt-in loopback SSH tests are separate; they are not rerun for this UI-only update.
- New interface checks exercise all three tabs at laptop/desktop widths with a pending worker: the former header status and bottom-right sentence are absent, while the spinner and cancellation control remain visible and within the header.
- Actual pointer-click tests open/close the shared docked Run log, copy its active history and clear only that history. The search log stays inside the main window. Search and Gaussian histories remain separate. Existing search/transfer tests additionally verify retained activity beyond the former 500-entry cap and across a new search and handoff.
- Viewer-toolbar geometry is checked with a clustered result, in individual and ensemble modes, at 340/640/960 logical-point widths and 1×/1.5×/2× display scales. Mode buttons, label dropdown and Fit have matching heights and vertical centers within each row, with no overlap or horizontal overflow when wrapping.
- All seven built-in scientific/interface self-check groups pass. Formatting and strict Clippy for all targets with warnings denied pass.
- All 32 actual egui demonstration renders are regenerated, including new open search-log and clustered-toolbar desktop/laptop examples. The new examples and existing Job setup/Analysis views are visually reviewed.
- A source comparison confirms changes only to conformer/UI logging, methodology text and interface verification, apart from release metadata/documentation. All 15 pre-existing artwork/font images, including vendored viewer assets, are byte-identical to 2.1.0. The Cargo dependency versions are unchanged.
- Public config still has no active SSH/workflow headers. ZIP integrity, source root/version, Linux build-script permissions and equality of the standalone/embedded Windows workflow are verified before saving.

## Platform and scientific scope

No native Windows executable is built or run in this Linux workspace. The included workflow builds/tests Windows x64 from exactly TENSOR-2.2.0-source.zip at the repository root, verifies version 2.2.0 and packages the executable and runtime files. Native Windows rendering remains a workstation check; the geometry tests use the actual egui controls rather than mockups.

No company host, scheduler or credentials are contacted. The 2.1.0 independent scientific comparisons, force-field oracle checks, one-pass LOG-transfer observation and platform limitations remain in the preserved previous validation record. They are prior checks, not new claims of Windows or real-HPC testing in 2.2.0.
