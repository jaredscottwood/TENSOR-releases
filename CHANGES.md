# TENSOR changes

## 2.0.0 — public release

- Format the NOE equivalence factor with subscript indices and a fully raised fractional exponent in Methodology.
- Remove active built-in SSH hosts/workflows and retain only commented generic examples. Public docs and screenshots use generic site settings; existing user configurations are preserved.
- Fix valid long transfer filenames failing because staging added the full basename to a prefix/suffix.
- Reject slash-only root aliases as a remote working base, and complete malformed-section diagnostics.
- Retain validated scientific algorithms/constants, the framed SHA-256 verification, one-pass SFTP fast path, cleanup guards, optional HPC integration, spectra, atom-selection controls and developer credits.
- Update application/resources/build scripts and exact-name GitHub release workflow to 2.0.0; rerun release and independent scientific checks.

## 1.2.1 — private/testing

- Show upload/submission workflows before retrieval workflows for each SSH host, including user-defined methods with custom names.
- Capture SHA-256 output into a framed manifest so SSH startup messages, blank lines and surrounding stderr are not mistaken for file checksums. Use a fixed C locale and bypass shell functions for the utility invocation.
- Require one complete frame, a successful command exit, and exactly one valid hash for every expected filename. Malformed, missing or duplicate records still block cleanup; no checksum failure silently falls back to an unverified download.
- Retain the single-pass SFTP transfer path, local/remote checksum comparisons, no-overwrite commits and pre-cleanup inventory/content checks. Provide a bounded offending-record excerpt for malformed checksum diagnostics.
- Update source, configuration, Windows resources and exact-archive GitHub workflow to 1.2.1. Retain credits and user configuration.

## 1.2.0 — private/testing

Follow-up fixes from the Windows test:
- Reset idle SSH/HPC windows on close; ask before stopping active work or exiting during it, and interrupt confirmed transfers without undoing submitted jobs. Collapse inputs/commands/history, display ordinary Windows paths and compact single-line command wrappers.
- Replace three repeated full-file network comparisons with server/local SHA-256 checks when available, use 1 MiB buffers, and show transfer progress/speed. Retain byte-comparison fallback, no-overwrite commits and cleanup guards; remove the required cleanup checkbox.
- Update private remote bases to `/2026` and restore commented generic SSH/method examples. Existing edited configs remain intact.
- Remove the independent C–H estimate checkbox; supply missing one-bond estimates automatically with calculated values always preferred. Aggregate satellite notices by spectrum and collapse secondary model details.
- Remove redundant header descriptions and give status the remaining space between the tabs and SSH/HPC controls.

- Added toggleable natural-abundance ¹³C satellites in proton spectra, labeled missing one-bond C–H estimates, and coupling-only H/C partners for incomplete shift inventories. Calculated J values retain priority.
- Spectrum settings update live, preserving nucleus selection, zoom/pan and intensity; cancelled or stale calculations cannot overwrite current settings. Removed the fully enriched GUI checkbox and expanded first-order/decoupling explanations.
- Replaced manual receipt handling with host-specific ordered upload/run/download/cleanup methods in the existing remote base. Default upload picker/download folder is beside the executable. Xsampler launches in the background.
- Added the requested private host settings and scripts, optional plaintext config passwords with `<prompt>` support, modern key-exchange preferences, and Windows bundled OpenSSL.
- Cleanup follows successful byte-verified retrieval and stops on conflicts or changed inventories. Broad cleanup scripts additionally require their entire guarded scope to have been retrieved. Legacy receipt APIs remain in the source for compatibility/tests; the new interface does not create or require receipts.
- Centered SSH rows, widened the top run-status area, removed duplicate automatic status tooltips, and displayed frequency units as cm⁻¹.
- Updated source/resources/config/build identity to 1.2.0 and retained existing developer credits.

## 1.1.0 — private/testing (not a public release)

- Aligned custom clustering controls in fixed-height, vertically centered rows.
- Omitted the unnecessary analysis TSV ownership manifest; complete result directories still commit atomically. Setup manifests remain for stale-output cleanup.
- Centered labels on atom projections without changing depth testing, occlusion or antialiasing; added element symbols to the existing label/export path.
- Matched Reset view height to the neighboring label menu and spectra button.
- Deferred child-window maximization until after native creation, once per opening. Manual restore remains user-controlled.
- Expanded chemical-equivalence methodology: graph refinement, whole-molecule bijections, parity, rigid/flexible mapping, search limits and NMR-context assumptions. Scientific algorithms/constants were not changed.
- Added explicit host-verified SSH/SFTP profiles, configurable workflows and batch scripts, JSON receipts, status adapters and no-overwrite retrieval. Passwords/passphrases are session-only, not saved configuration.
- Added the SSH/HPC top-row button and integration documentation.
- Clarified selection: point at an atom, then middle-click or press right shift. Left shift remains non-selecting.
- Updated Cargo/resources/config/build packaging to 1.1.0. Retained developer credits. Future public 2.0.0 is not created here.

## 1.0.0-public

- Established a separate public-testing version line from the validated 1.5.4 development build. Cargo, CLI, Windows resources, documentation, and distribution folders identify this package as `TENSOR-1.0.0-public`.
- Replaced ring-relative aromatic-dash spacing with a fixed absolute centerline separation and 0.058-model-unit surface clearance for ordinary five-, six-, and seven-membered rings. Distorted geometry retains an inward safety cap.
- When one bond belongs to two fully delocalized chordless local faces, the renderer now draws a dash guide into both geometrically distinct faces. Partial fused systems still choose only the best-supported local ring, and chorded composite outer perimeters remain excluded.
- Added shared cycle enumeration with explicit length/result budgets and reused it for fused-ring rendering and aromatic H–H fallback perception. Six-membered C/N coupling candidates can no longer be hidden merely because another fused path is shorter.
- Added regressions for indole-like 5/6, naphthalene/quinoline-like 6/6, and azulene-like 5/7 fused graphs; equal cross-ring spacing; dual-sided shared bonds; the supplied redacted fused aza topology; aromatic MDL and Kekulé coupling encodings; and fused C/N six-membered coupling perception.
- Made the first public run use the `TENSOR` roaming/configuration folder and clean bundled defaults. It neither discovers nor imports `nmr_workbench` data, and public distribution builders do not carry private/test-build configurations forward.
- Added Right Shift as a laptop-friendly alternative to middle-click atom selection. Selection occurs at the atom nearest the pointer only while the analysis viewer is hovered; Left Shift is deliberately unaffected.
- Narrowed the analysis settings pane's responsive default and allowed a slightly smaller minimum so the analysis structure viewer receives more horizontal space on laptop displays. The divider remains resizable.
- Added the visible development credit “Developed by Jared Wood and OpenAI GPT-5.6 Sol” to the opening workflow chooser.

The entries below record the internal development lineage from which the public test release was prepared; they are not later public version numbers.

## 1.5.4

- Corrected delocalized-bond placement in asymmetric fused and bridged systems. Composite outer perimeters containing a shared-ring chord are no longer selected over local rings, and every dash guide now uses a centered, constant ring-relative perpendicular offset rather than moving its endpoints independently toward a centroid.
- Added a redacted fused 6/5 aza-ring regression fixture. It verifies local five- and six-membered cycle selection, centered/equal dash spacing around each ring, and the expected adjacent aromatic-proton fallback coupling.
- Extended the optional aromatic H–H fallback to carbon-bound protons on qualifying six-membered C/N rings, including fused aza-aromatic systems whose nitrogen bonds are represented as single. Calculated pairs still take precedence; N-bound hydrogens remain excluded; the benzene-derived values and their extrapolation limits are now explicit.
- When two proton-bearing sites occur in more than one qualifying aromatic ring, choose the shortest ring-path separation deterministically instead of the first set ordering.
- Added separate `sulfur_calculation_reference` and `halogen_calculation_reference` entries to every built-in method. The sulfur definition cites Pierens (J. Comput. Chem. 2014, DOI 10.1002/jcc.23638); the halogen definition cites Giesen and Zumbulyadis (Phys. Chem. Chem. Phys. 2002, DOI 10.1039/B206245C). These are distinct from the C/H scaling provenance.
- Increased the setup cluster viewer's minimum logical height from 220 to 260 points while retaining the surrounding high-DPI scroll region.
- Made configuration augmentation retain a release-specific `.pre-VERSION.bak` file and removed stale 1.5.2 attribution from comments added by later releases.
- Audited all source targets and direct dependencies. Strict Clippy reports no deprecated API use or warnings; all direct dependencies remain active, and duplicated event-loop/Wayland crates are transitive to the pinned GUI/dialog stack.

## 1.5.3

- Replaced the analysis viewer's shift-label checkbox with a mutually exclusive Labels menu for Boltzmann-weighted shift values, one-based atom numbers, or no labels. The active label type is retained across conformers and is also used by molecular figure export.
- Added analysis-only atom picking and geometric measurements. Middle-click selects the nearest projected atom center, exact screen-position ties prefer the frontmost atom, a second click removes an atom, and selection is capped at four atoms. Two selections report distance; connected three- and four-atom sequences report angle and signed dihedral.
- Added translucent cyan selection rings, ordered badges, dashed selection connectors, a compact measurement readout, and an explicit Clear selection action. Loading a different conformer or compound clears the selection; the setup cluster viewer remains non-selectable.
- Made the Methodology and Predicted NMR spectra auxiliary windows request maximized state when opened, matching the working application window.
- Made the analysis viewer toolbar switch to two rows at narrow logical widths so the wider label selector cannot extend beneath the resizable analysis pane.
- Added regressions for label inventories and export metadata, screen-space/depth-tie picking, selection toggling and limits, connected-sequence validation, distance/angle/dihedral calculation, middle-click delivery through the real analysis UI, auxiliary-window maximize requests, and narrow toolbar layout.

## 1.5.2

- Made each shipped method definition self-contained: standard and sulfur/halogen correction jobs now expose separate functional, basis, keyword, route, and scaling entries in `tensor_config.txt`. Route templates use functional and basis placeholders, including the explicit `blyp` plus IOP representation used for WP04 calculations.
- Moved the complete iodine MIDIX data into an editable multiline `[basis:iodine_midix]` configuration section. Iodine-containing standard and correction jobs resolve their configured non-iodine basis and named iodine block through the same generator; no special route or basis data remain hard-coded in Rust.
- Added conservative config augmentation for older retained built-in methods. Missing 1.5.2 fields are added without replacing existing values, and the original file is retained once as `tensor_config.txt.pre-1.5.2.bak`. Analysis continues to read the exact C/H/F/N/sulfur/atom-specific halogen factors serialized into completed Gaussian titles.
- Updated both distribution builders to carry an existing 1.5.1 configuration into a new 1.5.2 distribution before the in-app augmentation runs.
- Disabled optional setup calculations when the selected method lacks their required route or scaling constants, and made all requested C/H/F/N scalings mandatory at generation time. Missing halogen atom-specific factors now produce a precise error rather than an incomplete title.
- Vertically centered setup/analysis form labels, dropdowns, buttons, and blank-destination hint text. The analysis export label now reads “Coupling constants (if applicable).”
- Made the working window maximize by default after the frameless splash (and immediately for direct-workspace launches). Side-pane defaults respond to the available logical width.
- Made the complete left setup card vertically scrollable for high UI scaling and small logical viewports, retained a 220-point minimum cluster-viewer height, and consumed viewer-local wheel zoom before the surrounding card can scroll.
- Added regressions for editable standard/special routes and iodine blocks, non-destructive legacy-config augmentation, maximized startup, exact overlay-row alignment, and a 760 × 520 logical high-DPI layout with the Run log open.

## 1.5.1

- Kept aromatic 1.5-order dashes coherent across fused and bridged rings. When a shared bond can close more than one ring, the renderer prefers the bounded path with the strongest delocalized-bond support and uses deterministic length/atom-order tie-breaks.
- Restored full display-density antialiasing during rotation, panning, and zooming; the viewer no longer substitutes a reduced-resolution interaction frame.
- Gaussian-input generation now returns and installs the exact aligned cluster overlay it calculated. The manual Build overlay/Refresh control remains available, but a successful generation does not repeat alignment and clustering.
- Expanded the default setup and analysis settings panes, aligned each viewer-control group in fixed-height cells, removed the duplicate working-header logo, and made the expanded Run log span beneath both panes with its scrollbar at the far right.
- Renamed the active configuration and generated run logs to `tensor_config.txt` and `tensor-log.txt`. Legacy configuration files are migrated without rewriting their custom contents. Window and chooser text no longer include a stale release number; version identity remains in executable metadata and `--version`.
- Made every method's C/H/F/N/S and one-, two-, and three-Cl/Br/I scaling pairs editable. Added per-method provenance notes for DELTA50, Menegatti et al., the unpublished method-4 adaptation, and Pierens.
- Exported spectrum SVGs now encode isotope and formula super/subscripts as semantic text spans, avoiding missing-glyph boxes in SVG viewers that do not preserve the application's font fallback.
- Added focused regressions for fused-ring dash placement, exact setup-overlay handoff, complete scaling inventories/provenance, semantic SVG scripts, expanded layouts, full-width logging, and full-resolution interaction rendering.

## 1.5.0

- Added an on-demand setup-tab cluster viewer. It uses the production alignment and clustering path, lets the user choose a compound and cluster, and superimposes every conformer in that cluster without exposing an individual-conformer selector. Disabling clustering unloads the overlay.
- Made a blank setup destination mean “beside the conformer files,” matching blank analysis output behavior. Compounds may resolve to separate source folders; one compound spread across folders requires an explicit shared destination.
- Added conservative topology-aware bond depiction after Wiberg classification. A terminal carbonyl candidate in the ambiguous WBI band is displayed as C=O, and two comparable terminal C–O bonds can be retained as delocalized. Raw Wiberg indices remain unchanged in reports.
- Improved laptop responsiveness: SDF parsing, conformer alignment, and pairwise fingerprint work use available CPU workers; long file lists virtualize their rows; 3D interaction uses a bounded temporary raster before producing the settled high-DPI antialiased frame.
- Spectrum plots now cache peak-preserving pixel envelopes while retaining full calculation data for CSV/SVG export. Closing the spectrum window releases its compound-derived input, simulation, and plot caches, and opening it clones only the analysis fields it needs.
- Filled the spectrum work card to the viewport instead of leaving an unused dark region below it, and removed the output-row layout behavior that created a large blank block in setup.
- Enabled mipmapped logo filtering for the compact frameless splash so the full-resolution supplied image downsamples cleanly at different monitor DPI scales.
- Corrected chemical typography throughout the interface and current documentation, including CF₃, H–C(sp³)–C(sp³)–H, and isotope superscripts.
- Added regressions for topology overrides, cluster-scene composition, beside-source setup output, ambiguous multi-folder input, and peak-preserving display downsampling.

## 1.2.3

- Replaced nonfunctional blue DOI hyperlinks in the predicted-spectrum window with compact, neutral source text and retained the complete DOI references in `SPECTRA.md`.
- Let the active-tab description use all header space left after the status and action buttons, so the full analysis description is visible whenever the window has room. Narrow windows still truncate safely and expose the complete text on hover.

## 1.2.2

- Shift labels retain camera-depth occlusion but now render over every bond capsule attached to their own atom. Unrelated nearer molecular surfaces continue to occlude them in the live viewer and PNG/SVG/PDF exports.
- Reduced the startup splash from 460 × 460 to 230 × 230 logical pixels while sampling the same full-resolution supplied image.
- Combined the logo, workflow tabs, active-tab description, status, Configuration, and Methodology controls into one fixed 44-pixel top row. The removed row height is returned to the setup and analysis content below.
- Gaussian and SDF file lists now use numeric-aware path ordering, so conformers 1–10 display in numerical order.
- Spectrum region selection keeps its press origin across input frames, restoring left-drag zoom. Plot-local wheel input changes intensity without also scrolling the surrounding spectrum list, and content dragging no longer competes with plot gestures.
- Replaced a remaining historical Python-reference sentence in the generated analysis report with the exact rounding rules, and corrected the documentation to match the implemented three-decimal scientific-notation rounding of per-conformer NOE values.
- Added focused regressions for local-bond label priority, natural filename sorting, one-row content allocation, spectrum wheel consumption, and multi-frame region zoom.

## 1.2.1

- Reworked the interface into a dim charcoal shell with rounded white work cards, clearer contrast, and persistent far-right scrollbars for the SDF and Gaussian-log lists.
- Shift labels now participate in camera-depth occlusion. Nearby atoms and bonds mask rear labels consistently in the live viewer and PNG/SVG/PDF exports.
- Aromatic 1.5-order dashes are directed by detected ring paths, keeping them inside five- and six-membered rings.
- Added missing aromatic proton-coupling estimates for benzene-like six-carbon rings: 7.54 Hz ortho, 1.37 Hz meta, and 0.69 Hz para. Calculated Gaussian J values retain precedence.
- Spectrum plots now support left-drag region expansion, wheel-controlled vertical intensity, right-drag panning, and a one-click full-spectrum reset. Full-view curves are cached and zoom sampling is bounded.
- Added optional conventional residual-solvent markers when all accepted standard logs agree on one supported SCRF solvent keyword.
- Added qualitative ¹³C response weighting by protonation so protonated carbon peaks appear stronger than quaternary carbon peaks.
- Applied one subtle rounded-corner treatment to every in-app logo instance, while retaining a stronger mask for the executable/taskbar icon.
- Reworded first-run and Methodology text so each feature explains its current behavior without referring to an unseen prior workflow.

## 1.2.0

- Splash displays the full supplied image with no white frame, padding, rounded mask or translucent corner fillers.
- Bundled scientific-symbol fallback fonts cover superscript minus and scientific notation in proportional and monospace interface text.
- Bonded atoms no longer add separate spheres. Each rod keeps its existing cylindrical body and matching hemispherical ends; multiple bonds, aromatic dashes, depth ordering and camera behavior are retained. Isolated atoms remain visible as spheres.
- Removed the Workspaces button. Tab buttons retain the session and switch workflows.
- The main window carries the application title and version. Inside the window, the header briefly describes the active tab.
- Methodology and user documentation explain the current application without development history or references to an unseen workflow. Tiered selection now gives its exact representative rules and spread thresholds. Machine descriptions are self-contained.
- Added an interactive predicted-spectrum window for ¹H, ¹³C, ¹⁹F and ¹⁵N, with calculated J precedence, optional population-weighted vicinal proton estimates, optional methyl/CF₃ averaging, field/linewidth/cutoff controls, partner-specific decoupling, isotope-enrichment modes, and CSV/SVG exports.
- Connected systems of up to ten spins use a numerical Hamiltonian; larger systems use an explicitly identified first-order approximation. Missing inputs and scientific limits are reported in the window and exports. See SPECTRA.md.
- Reopened results load a weighted J export when available. Reanalyzing original logs provides atom-resolved coupling data for magnetic-inequivalence treatment.

- Build scripts respect CARGO_TARGET_DIR when collecting the executable; spectrum export generation runs in a worker thread.
- Disabled cross-crate ThinLTO after reproducible release-link failures; standard optimization level 3 and codegen-units = 1 remain enabled. Normal release builds use the same configuration as validation.

The normal Windows GUI launch retains the Windows subsystem setting, including builds made with `cargo build --release`. No companion console is requested. Build and validation details are in VALIDATION.md.
