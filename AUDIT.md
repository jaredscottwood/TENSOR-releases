# TENSOR 2.2.0 — public UI and run-log review

This release starts from the validated public TENSOR 2.1.0 source. Changes are confined to interface layout, activity logging, release identity and documentation. Gaussian parsing/generation, conformer-search kernels and defaults, clustering mathematics, NMR/NOE calculations, spectra and SSH transfer algorithms are retained. Pinned dependency versions and existing artwork/font assets are unchanged.

## Changes reviewed

- The shared header no longer draws the status message or its hover text on any tab. The rotating busy indicator and main-worker cancellation control remain available. Internal status values are retained for operation/error handling, without a replacement header label.
- The bottom-right “Local analysis • optional SSH/HPC job submission” sentence is removed.
- All three tabs use one shared, resizable bottom Run log renderer, with the same + Run log / − Run log toggle, Copy log and Clear controls. Conformational Search selects its own history; Job setup and Analysis retain their shared Gaussian history. Copy and Clear act only on the selected history. Search no longer opens its separate log viewport.
- Search activity records loading, input-validation problems, operation starts, progress, cancellation requests, completion/failure, every saved path and Gaussian-setup transfer. A new operation appends to the existing history instead of clearing it, and the old 500-entry truncation is removed. Starting an operation opens the log, matching the other tabs. Long histories use virtualized monospaced rows and horizontal/vertical scrolling, so offscreen entries are retained without being painted every frame.
- Clustering progress is forwarded while the worker runs rather than being delivered only after completion. The forwarding channel is closed when the operation returns or unwinds, and the scoped forwarding thread finishes before the result is published. Existing cancellation and failure handling remain active.
- The search viewer’s Individual conformer, Aligned ensemble, Cluster overlay, label selector and Fit controls share a common row height. The dropdown is allocated as one complete control when wrapping; all mode controls retain their existing selection/enabled behavior. Narrow layouts continue onto the next row.
- Source/config/resources/build scripts and the exact-archive Windows workflow identify 2.2.0. The public config still contains only commented generic SSH/workflow examples. The previous 2.1.0 validation record is retained separately.

## Review scope

Focused interface regressions exercise hidden header/footer text, busy/cancel controls, actual log toggle/copy/clear clicks, separation of histories, retained activity across search/transfer and viewer-row alignment at multiple widths and display scales. Actual egui renders are used for visual review. See VALIDATION.md for results and platform limits.
