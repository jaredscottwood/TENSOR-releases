# TENSOR 2.0.0 — public release

**Theoretical Evaluation of NMR: Structures, Orchestration, and Rendering**

A native Rust application for Gaussian NMR job setup and analysis. One desktop application has two tabs: Gaussian job setup, and NMR analysis with an interactive molecular viewer and predicted spectra. The supplied Python program is retained, unchanged, in `reference/` for audit and regression comparisons. Scientific parsing/generation runs in Rust without a local Python runtime; optional SSH workflows can invoke your configured remote scripts.

## Start

Open the `tensor` application in the compiled distribution. The supplied logo appears by itself in a compact 230 × 230 logical-point splash for approximately 1.5 seconds, followed by a maximized first-run choice in the same window. The opening chooser credits development by Jared Wood and OpenAI GPT-5.6 Sol. The full-resolution asset uses mipmapped filtering so its downsampling follows the display's DPI scale without the jagged detail seen from single-level minification. Choose Job setup or Analysis & 3D view. The two numbered tab buttons switch workflows without clearing your session. Each tab has its own detailed Methodology window, and both Methodology and Predicted NMR spectra open maximized by default. The tabs, full-width run status, SSH/HPC, Configuration, and Methodology controls share one compact top row; the logo is reserved for the splash, window icon, and taskbar. The reclaimed height belongs to the tab content below it.

On Linux, the file must be executable; run `chmod +x tensor` if your archive extractor removed that permission. A desktop session with an OpenGL-capable driver is required for the window. The Windows build embeds the rounded logo in `tensor.exe` for Explorer and the taskbar, and normal GUI startup does not open a console window.

This archive contains source. Build on Linux for a Linux executable, or use the included PowerShell script on Windows for `tensor.exe`. Linux system-library requirements depend on the machine used to build it. See `VALIDATION.md` for the checks performed on each platform.

```sh
./tensor --demo
```

The built-in example is clearly labelled **Synthetic example**. It demonstrates three ethanol conformers, duplicate removal, population weighting, and shift labels. Its coordinates and NMR numbers are test data; no Gaussian calculation produced them. Setup from the embedded example works without separate SDF files. To analyze the example logs yourself, use `examples/logs/` from the source distribution.

The application prepares input files and processes finished logs. The new **SSH/HPC** button can upload files, run your configured cluster scripts, and retrieve and clean up completed outputs using host-specific methods. Upload methods appear first in the Method dropdown. Retrieval isolates server checksum records from SSH startup messages while retaining content and cleanup verification. See [SSH_HPC.md](SSH_HPC.md) for configuration, credentials and limitations. Gaussian is not bundled. The public configuration ships with commented generic SSH/workflow examples and no active hosts.

## Tab 1: Job setup

1. Add individual `compound_number.sdf` files, a collective SDF ensemble, or a folder. Drag-and-drop is supported. Native system dialogs provide multiple-file selection and folder selection. Folder scanning and SDF loading run in the background; Include subfolders is optional. V2000 and V3000 are accepted.
2. Choose no clustering, the chemical-shifts preset, the all-NMR preset, or custom linkage/threshold/selection settings. Input numbering defines the “lowest energy” selection rule; the setup stage does not calculate energies. With clustering enabled, the overlay controls align and cluster the current compound, then let you choose a cluster and inspect all of its conformers superimposed in the setup-tab 3D viewer. Successful Gaussian-input generation automatically installs the exact aligned overlay produced by that run; **Build overlay** or **Refresh** remains available for inspection before generation or after settings change. There is deliberately no individual-conformer selector in this view.
3. Choose the method/solvent, temperature, machine, and resource tier. Optimization, frequencies, and carbon/proton NMR are included. Add coupling constants, fluorine/nitrogen shifts, and sulfur/halogen carbon corrections as needed. An optional calculation is enabled only when both the selected structures and the selected method definition support it.
4. Review the Gaussian preview and generate. Leave Output blank to create `gjf_files/` beside each compound's conformer files, matching the analysis tab's beside-input behavior. Choose a directory only when one shared destination is wanted. A compound whose individual SDFs span folders requires an explicit shared destination. Files use Unix line endings; intermediate split/aligned/clustering SDFs can be retained.

“Process every loaded compound with these settings” starts **unchecked**: generation processes only the selected compound. Tick it to apply the visible setup to every loaded compound. Settings are remembered while switching compounds during the session. All additional calculations and all Keep options start unchecked. Element-specific options are enabled only when every selected structure contains the required element and the current method defines the necessary route and scaling constants.

The application starts maximized. Setup and analysis settings panes choose responsive initial widths and remain manually resizable. The analysis pane deliberately starts narrower than the setup pane so the structure viewer retains more horizontal room on laptop displays. At small logical window sizes—including high operating-system UI scaling—the entire left setup card scrolls vertically, while its cluster viewer retains a 260-point minimum height. Wheel input over that viewer zooms the molecule without moving the surrounding card.

Conformers are aligned to the lowest-numbered structure using heavy atoms and hydrogens bound to N/O/S/P, using all atoms if that selection contains fewer than three atoms (structures with fewer than three atoms keep their supplied coordinates). Alignment uses proper rotations. Identical atom numbering, elements, charges, isotopes, and connectivity are required within an ensemble. SDF parsing and independent alignments use available CPU workers while preserving deterministic input order; pairwise fingerprint construction is also parallelized.

The two clustering presets are:

| Preset | Linkage | Relative threshold τ | Selection | Dihedral weight |
|---|---|---:|---|---:|
| Chemical shifts | complete | 0.40 | tiered | 0.50 |
| All NMR, including J | complete | 0.15 | lowest_energy | 0.50 |

Custom linkage supports complete, average, single, weighted and Ward. The all option selects with average linkage and reports Ward, complete, average and single for comparison. A medoid minimizes the sum of distances to the other cluster members. Tiered selection retains the lowest-numbered conformer, also retains the medoid if different when mean pairwise spread is at least 0.12, and adds the remaining conformer farthest from the medoid when spread is at least 0.18. The Methodology window explains the normalized distance/torsion fingerprints.

## Configuration

`tensor_config.txt` uses `key = value` entries within named sections. The Configuration button opens an editor with validation and reload. A first launch with no existing TENSOR configuration creates the bundled public defaults; older `nmr_workbench` data is not imported. Existing TENSOR configurations retain their edits and are augmented with missing shipped fields without replacing their existing values; before a later release adds fields, TENSOR preserves the prior text once as `tensor_config.txt.pre-VERSION.bak`. Custom method sections are not populated with unrelated defaults.

Lookup order is an explicit `--config` path, an existing `tensor_config.txt` in the working directory, an existing file beside the executable, then the TENSOR per-user folder (`%APPDATA%\TENSOR` on Windows; `$XDG_CONFIG_HOME/TENSOR` or `~/.config/TENSOR` on Linux). Defaults are created only when the chosen file does not exist. Public distribution builders deliberately replace their packaged configuration with the clean bundled defaults; keep intentional custom methods in a separate file and select it with `--config` while rebuilding.

Add `[method:NAME]` sections for calculation methods, `[basis:NAME]` sections for multiline explicit basis data, and `[machine:NAME]` sections for resource profiles. Each shipped method keeps separate functional, basis, keyword, and complete route fields for optimization, carbon, proton, fluorine, nitrogen, and coupling calculations, plus sulfur- and halogen-correction optimization/carbon passes. Route lines use the matching `{*_functional}` and `{*_basis}` placeholders, so WP04 remains transparently represented as `blyp` plus its configured IOP keywords rather than as an opaque functional name.

Each method also contains complete slope/intercept pairs for carbon, proton, fluorine, nitrogen, sulfur-carbon, and carbons bearing one, two, or three Cl, Br, or I atoms. Partial pairs are rejected. The factors selected during setup are written into each NMR job title; completed-log analysis reads those exact title values rather than consulting the current config. `scaling_provenance` records the C/H calibration source or unpublished status beside the editable values. The separate `sulfur_calculation_reference` entry cites Pierens (J. Comput. Chem. 2014, DOI 10.1002/jcc.23638), while `halogen_calculation_reference` cites Giesen and Zumbulyadis (Phys. Chem. Chem. Phys. 2002, DOI 10.1039/B206245C). These calculation references do not silently change the editable special routes or correction constants.

For iodine, `iodine_basis` names an editable `[basis:NAME]` triple-quoted block in this same file. TENSOR substitutes `gen` into the active standard or correction route, applies that calculation's configured basis to non-iodine atoms, and appends the named iodine block. Optimization includes frequency, connectivity, temperature, and NBO keywords; the preview uses the same generator as saved files.

The **Default** machine provides checkpoint, scratch-file, core, memory and tier settings. Add named profiles for other machines or clusters.

## Tab 2: Analysis and 3D view

1. Add completed `compound_number.log` files or a folder. Matching `compound-csulfur_number.log` and `compound-chalogen_number.log` files beside loaded standard logs are picked up automatically. File lists use natural numeric ordering, so conformers 1–10 appear as 1, 2, …, 10 rather than 1, 10, 2.
2. Choose deduplication and the desired exports. Output switches can be shared or set for individual compounds. A blank result destination writes beside the logs; otherwise each compound gets a folder under the chosen destination.
3. Run analysis. The interface remains responsive and displays progress; Cancel is available before the export commit.
4. Select a compound and retained conformer. Drag to rotate, right-drag to pan, scroll to zoom, and use Reset view or double-click to restore the view. The Labels menu displays weighted shift values, one-based atom numbers, element symbols, or no labels. On narrow logical viewports the controls use two rows instead of extending beneath the analysis pane.

In Shift values mode, gold labels show weighted, chemically equivalent shifts, including available sulfur/halogen carbon replacements. They are the ensemble prediction and remain the same when viewing another conformer. Atom numbers match the one-based numbering used in Gaussian inputs and analysis exports. The initial displayed conformer has the lowest available Gibbs energy.

Point at an atom, then middle-click or press **right shift** to select it. Left shift has no selection behavior. Repeating either selection action on the same atom removes it; **Clear selection** clears the set. Picking chooses the closest atom in projected screen space and uses camera depth to break an exact overlap. Up to four atoms are marked with translucent cyan rings, numbered badges, and dashed connectors. Two atoms report distance in Å. Three connected atoms report their angle, and four connected atoms report the signed dihedral angle. A disconnected sequence is identified instead of producing an angle. Changing compound or conformer clears the selection. Picking and measurements are intentionally limited to the analysis viewer; the setup cluster overlay remains an inspection-only view.

The viewer uses a fixed 3D model: cylindrical rods capped by hemispheres of the same diameter, multiple bonds, and widely spaced aromatic dashes. Bonded atoms have no extra spheres; isolated atoms retain a sphere. Bond offsets are defined in molecular coordinates and do not change as the camera rotates. In fused or bridged systems, composite outer-perimeter cycles containing a shared-ring chord are excluded before local chordless rings are ranked by delocalized-bond support. Every ordinary five-, six-, or seven-membered face uses the same absolute bond-to-dash surface clearance; ring size determines direction but no longer scales the gap. If a delocalized bond is genuinely shared by two fully delocalized local faces, one dash guide is placed into each geometrically distinct face. Distorted rings cap an offset before it can travel too far through their local interior. Per-pixel depth resolves crossing surfaces, and nearby molecular surfaces can occlude rear labels according to each labelled atom's camera depth. Every capsule records its endpoint atoms: bonds touching the labelled atom yield to that atom's label, while any unrelated surface nearer than the label still occludes it. Camera motion, zoom, and panning retain the full display-density antialiased rendering; the completed frame is cached while idle. Gold labels use a slightly larger, bold outlined font.

Use **Open results** to reopen a prior result folder or its parent. Retain conformer SDFs and the PyMOL script if you want to reopen the same viewer state after restarting. Current-run viewing and copied shifts work independently of export switches.

### Predicted spectra

After analysis, select a compound and press **NMR spectra…** above the 3D view. A separate window shows every supported ¹H, ¹³C, ¹⁹F and ¹⁵N nucleus with a shift prediction. It uses the same combined, two-decimal shifts as the viewer and PyMOL labels.

- Calculated atom-resolved J values take precedence. Optional missing H–H estimates cover conformer-weighted vicinal H–C(sp³)–C(sp³)–H paths and carbon-bound protons on qualifying six-membered aromatic C/N rings, including fused aza-aromatic rings. The aromatic values are benzene-derived screening estimates rather than heteroaromatic-specific calculations. Other unknown couplings are omitted explicitly.
- Fast methyl/CF₃ rotation is optional. General chemical equivalence does not erase magnetic inequivalence.
- Connected systems of up to ten spins use a numerical spin Hamiltonian; larger systems use a labelled first-order approximation. Carbon and nitrogen default to proton decoupling, and every spectrum has separate partner-decoupling controls.
- Spectrum controls update live after a short debounce, keeping the selected nucleus, horizontal zoom/pan and vertical scale. **Show ¹³C satellites in ¹H** adds representative natural-abundance sidebands. Missing one-bond C–H values are estimated automatically when needed; calculated values, including zeros, take precedence. Each proton plot shows one satellite summary, with more explanation in its collapsed model details and atom-specific sources in the coupling inventory.
- When all accepted standard logs agree on one supported SCRF solvent, optional conventional residual-solvent markers are shown as comparison guides. They are not simulated impurity peaks or a recalibration.
- Carbon peak response is qualitatively weighted by protonation, so protonated carbons are visually stronger than quaternary carbons; the areas are not quantitative experimental response factors.
- Export transition frequencies and relative areas as CSV, or broadened curves as vector SVG. Exported metadata records the settings and limitations. Screen traces use cached, peak-preserving pixel envelopes; export retains the full scientific sample array. Closing the window releases its compound-derived spectrum data.

These are fast-exchange solution-spectrum predictions. Unknown couplings, approximate J values, first-order fallback, isotope conventions and omitted experimental effects limit the result. See [SPECTRA.md](SPECTRA.md) for the full model, numerical conventions, sources and limits. To reopen calculated couplings later, also retain the weighted coupling export; reopened exports cannot reconstruct magnetic inequivalence that was already averaged away.

### Scientific behavior

| Operation | Behavior in TENSOR 2.0.0 |
|---|---|
| Shielding conversion | δ = (shielding − intercept) / slope. Every C/H/F/N calculation uses explicit factors from its own title. Missing factors are errors; analysis never looks up a method key or supplies default factors. |
| Thermal populations | Gibbs energy differences in kcal/mol; 627.509474 kcal mol⁻¹ Hartree⁻¹ and R = 0.0019872036 kcal mol⁻¹ K⁻¹. Temperature comes from the logs unless explicitly overridden in the CLI. |
| Duplicate minima | ΔG ≤ 0.20 kcal/mol; graph-legal atom permutations and proper rotations; RMSD ≤ 0.10 Å; maximum atom displacement ≤ 0.15 Å. Comparable frequency lists must agree within RMSD 2.0 and maximum 3.0 cm⁻¹. |
| Missing frequency data | Geometry/energy deduplication may proceed, with an explicit report that the frequency comparison was unavailable. |
| Equivalence | Graph refinement, stereochemical parity, rigid-bond, ring, and conservative treatment of difficult centers. Chemical equivalence and duplicate-conformer permutations remain separate decisions. |
| Couplings | Read the total blocked J matrix, perform equivalence averaging, then population weighting. |
| Distances | Methyl/CF₃ closest-separation circle model, including circle–circle treatment. Per-conformer distances are rounded to 0.01 Å before weighting. |
| NOE quantities | Relative 1/r⁶ values. Per-conformer NOEs are rounded to three decimal places in scientific notation before weighting. Equivalence multiplicities (N<sub>i</sub> N<sub>j</sub>)<sup>−1/6</sup> and corresponding apparent distances are reported separately. |
| Special carbons | Sulfur and halogen job ensembles have separate populations. Sulfur requires explicit carbon title factors; halogen requires per-atom title factors or an explicitly supplied uniform carbon pair. A carbon bound to a heavy halogen is excluded from the sulfur correction. No analysis table or average-method fallback is used. |
| Bond orders | Initial Parameters supplies connectivity and the first NBO Wiberg matrix supplies continuous indices. Thresholds create discrete depictions; an ambiguous high-index terminal C–O is shown as carbonyl, while two comparably strong terminal C–O bonds can remain delocalized. Raw WBI values remain in reports and missing entries retain the supplied order. |

Manual populations are optional: place `compound_boltzmann.txt` beside the logs, with lines such as `Conformer 1: 65.0%`. Use the corresponding `compound-csulfur_boltzmann.txt` / `compound-chalogen_boltzmann.txt` names for special ensembles. Positive surviving populations are normalized after exclusions and deduplication. Negative, repeated, nonfinite-total, or all-zero populations are rejected. Zero-population conformers retain their unweighted data but do not add weighted output keys or invalidate the populated ensemble’s scaling/coupling inventory.

Titles use fields such as `aspirin_1_carbon s:-1.0065 i:196.0386`. Proton, fluorine, nitrogen, and sulfur-carbon titles each carry their own pair. Halogen titles may use one-based atom-specific fields such as `hal{1:s:-1.02,i:177.1|2:s:-1.07,i:207.3}`. The setup configuration still supplies constants when writing new titles; completed-log analysis reads only the constants written in those titles.

Software tests do not establish the accuracy of a particular DFT method, experimental assignment, or NOE distance model.

### Exports

Files and folders are produced for weighted/unweighted shifts, couplings, distances, NOE intensities, equivalence multipliers, conformer SDFs, frequencies, Wiberg indices, and PyMOL labels. Gibbs energies and population summaries are always written, along with an analysis report. Standard and special carbon results remain separate. An additional `compound_combined_weighted_shifts.txt` records the values used for viewing when carbon corrections are present.

**Export image** saves the current orientation, framing, and aspect ratio as PNG, SVG, or PDF, with independent controls for transparency and the current shift-value, atom-number or element-symbol labels. Selection rings, badges, connectors, and measurements are interactive aids and are not exported. The default is 2400 pixels wide at 300 DPI. Every format uses the same high-resolution depth-tested molecular and label raster, so rear-label occlusion is consistent on screen and in exports. PNG includes DPI metadata; SVG/PDF preserve physical dimensions. The shaded molecule is raster content in those two containers; it is not an editable vector bond drawing. PDF transparency uses an image soft mask.

The run log reports every parsed file, title factors, acceptance/exclusions, structure and equivalence work, duplicate comparisons, populations, individual conformer processing, carbon corrections, and saved output types. A `tensor-log.txt` is also written to the destination. Its scrollbar spans the full application width beneath both work panes. Parsing and per-conformer shift/distance/NOE calculations use a bounded thread pool; results are collected in deterministic order.

### Practical limits

- A final normally terminated Gaussian job is required. Significant imaginary frequencies below −12 cm⁻¹, convergence failures, and error terminations are reported. Truncated final coordinate blocks are rejected rather than substituted with an earlier geometry.
- A populated conformer with missing C/H data or a different NMR atom inventory cannot silently contribute zeros. Incomplete coupling inventories are reported and the weighted coupling file is omitted. Special calculation atom identities/connectivity must match the standard ensemble.
- Analysis processes the explicitly loaded logs and their companions. Different compounds can come from different folders; one compound cannot combine ambiguous same-named ensembles from different directories.
- Regeneration replaces existing jobs for the selected compound. Stale conformer/correction jobs and deselected intermediate outputs are removed. Other compounds in a shared job directory are preserved. Loaded SDF inputs must be outside the generated intermediate folders that this run would replace or clean.
- Analysis prepares a complete new compound folder before replacing the previous folder and removing its stale contents. Previous edits inside that generated result folder are replaced. Cancellation or failure before the replacement leaves the previous analysis folder in place. A folder containing any selected input log, including one that failed to parse, cannot be replaced. Windows may require a locked output to be closed in another application first.
- Continued V3000 atom rows are handled consistently when reading and rewriting coordinates, retaining charge, isotope, connectivity, and SDF data fields. V3000 output includes the required atom-mapping field and SDF delimiter. PyMOL scripts refer to the correct conformer SDF subfolder.
- When generating Gaussian connectivity, MDL aromatic bond type 4 is converted to fractional bond order 1.5. It remains type 4 in SDFs and the viewer. Ambiguous SDF query bond types are rejected.
- Very large clustering matrices are rejected before an excessive allocation. Search budgets bound hard symmetry cases; unresolved equivalence is kept separate and reported. These are conservative limits, not claims of exhaustive graph-isomorphism coverage for every chemical system.
- Scientific comparisons use bundled synthetic fixtures. Validate a representative real Gaussian ensemble from your workflow before relying on the new executable for research production. Gaussian itself is not available in the build environment.

## Optional SSH/HPC methods

The public configuration contains only commented examples. Open Configuration, copy/uncomment the example `[ssh:...]` and `[workflow:...]` blocks, and replace the host, username, independently verified host-key settings, existing remote folder and submission script with your site's values. Upload methods appear before retrieval methods. No network connection occurs until you explicitly start an SSH/HPC action.

Uploads use no-overwrite staging in the existing `remote_base`. Retrieval verifies matching outputs before cleanup. Servers with `sha256sum` use server/local SHA-256 comparison so each new file crosses SFTP once; hosts without it use byte comparison. The window shows file/byte progress and speed. Closing it resets its state; active operations ask before stopping the connection. Already submitted jobs may continue. File pickers and download destinations default to the executable's folder.

Passwords default to `<prompt>`; optional literal config passwords are plaintext, including in backups. The examples use selective SFTP cleanup of verified downloaded filenames. A trusted site cleanup script is optional and requires an appropriate inventory guard for its full deletion scope. See [SSH_HPC.md](SSH_HPC.md) for configuration and failure handling; it is optional documentation, not a runtime dependency.

An existing user configuration is retained, including custom SSH/workflow entries. Removing private defaults from the public package does not erase a user's saved configuration. A first launch with no existing file creates the generic public defaults.

## Build from source

Install Rust 1.95 or later from [rust-lang.org](https://rust-lang.org/tools/install/). The validation build uses Rust 1.95.0 and pinned `Cargo.lock` dependencies. Windows builds need the MSVC Rust toolchain and Visual Studio C++ build tools **with a Windows SDK** (`rc.exe` embeds the icon/version). Windows SSH now statically bundles OpenSSL to support modern Curve25519/Ed25519 algorithms. Building that dependency needs Perl (Strawberry Perl or the Perl included with Git for Windows); the provided build script finds the usual paths. NASM is optional and the script uses OpenSSL's no-assembly mode. Users of the resulting executable do not need Perl or an OpenSSL DLL. Linux builds need a C compiler/linker, pkg-config, OpenSSL development headers/libraries, and the usual X11/Wayland/OpenGL runtime libraries. Native folder dialogs use the desktop's XDG portal.

```sh
cargo build --locked --release
cargo test --locked --release
./target/release/tensor --self-test
./target/release/tensor --demo
```

From PowerShell, open the extracted source folder containing `Cargo.toml`, then run:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\build-windows.ps1
Start-Process .\dist\TENSOR-2.0.0-windows\tensor.exe
```

The first command builds, tests and assembles the application. PowerShell can be closed after the second command; the GUI runs independently. Later, double-click `tensor.exe` in that distribution folder. You can instead use `cargo build --release` and launch `target\release\tensor.exe`; console suppression is part of the source, not a special launcher.

Linux: run `./build-linux.sh`. Each script builds for the host platform, tests, runs self-checks, and collects a distribution under `dist/`; one build does not create both Linux and Windows executables. Initial builds download Cargo dependencies. Normal analysis is local; the optional SSH/HPC workflow connects only when explicitly requested. Windows GUI builds suppress the companion console, including ordinary `cargo build --release`.

For GitHub, place **TENSOR-2.0.0-source.zip** at the repository root and the supplied
**build-TENSOR-2.0.0.yml** in `.github/workflows/`. Run its manual Actions workflow.
It uses only that exact archive name, checks version 2.0.0, builds/tests on Windows,
and uploads `TENSOR-2.0.0.exe` plus the full Windows distribution ZIP as a public-release
artifact. It does not publish a GitHub release. Download the full ZIP to retain
documentation/configuration/licenses alongside the executable.

Useful headless commands:

```sh
# Prepare an ensemble with the chemical-shifts clustering preset.
tensor --setup examples/sdf/ethanol_1.sdf examples/sdf/ethanol_2.sdf examples/sdf/ethanol_3.sdf --out jobs --cluster shifts --couplings

# Process the two synthetic regression ensembles, with separate result folders.
tensor --analyze examples/logs --out results

# Export only chemical shifts and reusable viewer files.
tensor --analyze my_logs --out results --outputs chemical_shifts,conformer_sdf,pymol_script

# Reopen exported structures and labels.
tensor --open-results results

# Draw the actual egui interface without a display server.
tensor --render-demo screenshots

# Exercise transparent 300-DPI PNG, SVG and PDF figure exports.
tensor --export-demo figure-examples
```

`tensor --help` lists all command-line options. Failed headless operations return a nonzero exit status; successful compounds may have been exported even if another compound failed, and the report identifies them.

## Regression checks

`--self-test` is built into the executable. `cargo test --release` uses the same core checks plus integration regressions. `tools/compare_reference.py` compares exported numerical records against the supplied Python’s golden outputs:

```sh
python3 tools/compare_reference.py target/release/tensor
```

This comparison needs only Python’s standard library; Python is not needed by TENSOR. To regenerate the fixtures/golden files, run `python3 tools/make_reference_fixtures.py`; that optional developer operation needs NumPy, SciPy, and the original script’s imports. It operates only in `tests/fixtures/python_results/`, where the original Python removes its own previous generated folders.

The actual test outcomes and packaging platform details are recorded in `VALIDATION.md`. `screenshots/` contains renders of the real interface, not design mockups.

## Source map

| Module | Responsibility |
|---|---|
| `config.rs`, `jobs.rs` | Routes, machine profiles, basis blocks, job generation, output replacement and progress |
| `molecule.rs`, `math.rs` | SDF handling, geometry alignment, numerical helpers |
| `cluster.rs` | Fingerprints, hierarchical linkage and conformer selection |
| `gaussian.rs` | Gaussian log parsing and completeness checks |
| `symmetry.rs` | Chemical equivalence and duplicate conformer comparison |
| `analysis.rs` | Scaling, populations, couplings, distances, NOEs and exports |
| `render3d.rs`, `viewer.rs`, `figure.rs` | Fixed molecular solids, depth rendering, camera controls and figure export |
| `spectrum.rs`, `spectrum_ui.rs` | Population-weighted spin parameters, spectra, controls and CSV/SVG export |
| `hpc.rs`, `hpc_methods.rs`, `hpc_ui.rs` | Verified SSH/SFTP, ordered methods and their interface |
| `ui.rs`, `methodology.rs`, `branding.rs` | Native dialogs, two tabs, startup, contextual methodology and supplied logo |
| `verification.rs`, `tests/` | Embedded example, regression fixtures, interaction checks and QA renders |

Developed by Jared Wood and OpenAI GPT-5.6 Sol. Third-party dependency notices are supplied separately; the original Python reference retains its original attribution.

Changes are listed in `CHANGES.md`. `AUDIT.md` and `VALIDATION.md` describe the checks and practical limits of this release.
