# Conformer search and selection methodology

## Workflow and authorship
The Conformational Search tab starts from one connected 3D structure with explicit hydrogens. Load a MOL or SDF V2000/V3000 record, or one MOL2 molecule, choose MMFF94s or UFF, and run Search conformers. The retained minima are ranked by force-field energy and aligned using heavy atoms and non-C–H protons. Optional hierarchical clustering selects representatives. Save individual SDF files or one grouped SDF, or choose Use in job setup to continue directly in the job setup tab. Uploading an existing conformer ensemble in the job setup tab is also supported.

Xsampler Lite is credited to Ryan Cohen and Anthropic’s Claude Opus. This application incorporates the supplied native Xsampler Lite force-field kernels, hybrid search, structure viewer and aligned ensemble renderer. The supplied Python host referred to an original xsampler_lite.py module that was unavailable in that source package. The native hybrid sampler is the implementation delivered with Xsampler Lite; exact reproduction of the missing original generator is not established.

The hierarchical clustering scheme is by Jared Wood. The alignment, chemical-equivalence analysis, duplicate analysis, molecular viewing, spectral prediction and data-processing tools are credited to Jared Wood and OpenAI GPT-5.6 Sol. These tools are integrated with the separately attributed force fields and configured Gaussian calculations. Scientific references and original component notices accompany the source.

## Input and chemical identity
The search accepts exactly one connected closed-shell structure, with 2–512 explicit atoms, valid bonds, 3D coordinates and explicit hydrogens. The input file limit is 16 MB. Formal charge, isotope information, atom ordering and user annotations are retained. MOL2 partial charges remain distinct from formal charge. Common file extensions are supported through content-based format detection.

Coordinate geometry supplies the tetrahedral and E/Z configuration used by the search. Connectivity and protonation remain fixed during sampling. Supply chemically consistent coordinates, bond orders and stereochemical annotations. A conformer search explores a molecule’s accessible shapes on the chosen force-field surface; it does not enumerate protonation states, tautomers or different molecular graphs.

Gaussian jobs use the methods, basis sets, element-specific options and resources chosen in TENSOR’s Job setup tab. Force-field search success does not establish that a selected Gaussian model chemistry is appropriate for the molecule.

## Force fields
MMFF94s is the default. It is a small-molecule force field with bond, angle, stretch–bend, out-of-plane, Fourier torsion, buffered 14–7 van der Waals and buffered Coulomb terms. The static MMFF94s variant supplies its own torsion and out-of-plane parameters. Atom typing, partial charges, parameter lookup and fallback rules follow the supplied implementation. Its parameter tables are embedded in the executable. Energies are in kcal mol^{-1}, coordinates in Å, and Cartesian gradients in kcal mol^{-1} Å^{-1}.

UFF offers another parameterization when appropriate, with bond, angle, Fourier torsion, inversion and Lennard–Jones terms. The supplied UFF implementation has no electrostatic term and uses lightweight hybridization/conjugation perception. Its supported elements and special torsion/inversion cases are limited to those implemented in the supplied source; the UFF name does not imply full periodic-table coverage here. An unsupported atom type produces a diagnostic. Force-field energies should be assessed for the intended chemistry.

The native kernels are carried over unchanged from Xsampler Lite. Their fixed-geometry regression fixtures cover energies, analytic gradients, types and charges against the supplied Python kernels. Validation scope is documented separately; agreement with a finite regression set does not establish universal chemical accuracy.

## Search strategy
The input is first relaxed with L-BFGS. A systematic torsion grid then explores six increments for flexible acyclic single bonds and two for restricted amide-like bonds, with a grid bounded to one third of the trial budget and at most 512 seeds. Terminal methyl rotations are excluded from conformational proposals; polar X–H rotations can contribute distinct geometries.

Further candidates use random single or multiple torsion changes, nonaromatic ring puckering/crankshaft moves and Cartesian perturbations. When short Cartesian MD is enabled, every twelfth post-grid trial can use a 256-step BAOAB Langevin trajectory at 900 K, with a 0.5 fs step and 5 ps^{-1} friction. Those hot, short trajectories generate proposals for minimization. They do not supply equilibrium populations.

Every proposal is locally minimized with the native L-BFGS optimizer: 12-vector history, safeguarded Armijo line search, maximum per-atom step 0.2 Å, and default maximum Cartesian gradient 0.001 kcal mol^{-1} Å^{-1}. The maximum iteration count is 1500. Geometry and stereochemical checks reject severe clashes, implausible bonds, unconverged candidates and changes to guarded C/Si tetrahedral or input E/Z geometry. Final gradients and energies use the physical force field after guard terms are removed. Nitrogen inversion is not frozen as a stereocenter.

Retained minima are sorted by energy, filtered by the energy window and grouped by symmetry-aware RMSD. The lowest-energy representative of each RMSD group is kept. Search parents include the initial minimum and up to 64 retained low-energy minima. A finite search cannot prove exhaustive coverage, particularly for difficult fused rings, macrocycles or very flexible molecules.

## Search controls and defaults
Thorough is the default budget. For R eligible acyclic rotors and C flexible rings, it requests 1024 + 256R + 256C candidate trials, capped at 16384. Standard requests 256 + 80R + 96C, capped at 4096. A candidate trial includes proposal generation, local minimization and acceptance checks; the number retained will normally be smaller. Thorough offers more opportunities to find minima, with correspondingly more computation.

Trials defaults to 0, which selects the budget formula. A positive count overrides the formula, up to 100000. RMSD defaults to 0.35 Å: lowering it can retain more geometrically similar minima; raising it can merge nearby shapes. The symmetry-aware comparison uses proper rotations and graph-consistent correspondence, with polar hydrogen matching and carbon-bound hydrogen permutations excluded. Symmetry enumeration is bounded and reports truncation.

The energy window defaults to 10 kcal mol^{-1} above the lowest retained force-field energy. Narrowing it removes higher-energy minima from the search archive. The random seed defaults to 2026. Trial-specific streams and fixed batches of 32 proposals preserve repeatability independently of worker admission on the tested platform; the same seed is not a guarantee of identical floating-point results on every architecture.

Maximum iterations defaults to 1500 per local optimization. Converged candidates stop earlier. Conformer limit defaults to 2000; reaching the limit truncates the energy-ranked archive and is reported. Short Cartesian MD starts enabled. Changing these controls affects the next search; the displayed result retains the actual settings used to generate it.

Resource allocation uses Xsampler Lite’s Balanced policy with automatic worker selection. External-power admission uses up to 45% of logical CPUs, capped at four workers and constrained by available memory; battery operation uses one worker. The native governor can reduce admission under CPU or memory pressure. Search and Gaussian setup/analysis run heavy tasks sequentially so their worker pools do not compete. Stop search retains already accepted minima when available; stopped sampling is identified in the log.

## Proper alignment and 3D inspection
After search, every retained conformer is fitted to conformer 1, the lowest-energy member, using every heavy atom and every hydrogen isotope not bonded to carbon. A proper-rotation Kabsch-equivalent Horn quaternion fit preserves handedness and atom correspondence. All atoms receive the same rigid transformation, including excluded C–H protons. Structures with a degenerate fitting mask can have an undetermined rotation about an axis; reflections and atom permutations are not introduced.

The aligned work copy is used consistently for individual viewing, the full ensemble object, hierarchical clustering, saved files and transfer to Gaussian job setup. Rigid rotations/translations preserve intramolecular distances, handedness and force-field energies. Alignment does not create another conformer or minimize the structure again. TENSOR subsequently prepares Gaussian optimization jobs using the chosen calculation method; Gaussian runs externally.

The header shows a rotating loading indicator while work is active. Open + Run log at the bottom to inspect search, clustering, export and transfer activity in the same resizable panel used by the other tabs. Copy log copies the complete history; Clear clears the active tab’s log. Search activity is kept separately from Gaussian setup/analysis activity. The three main cards use the remaining window height, with setup on the left, molecular viewing in the middle and conformers on the right.

The individual viewer retains the supplied renderer’s element colors, labels, rotation, zoom, panning and geometric measurements. Select any conformer in the right-hand list, including members omitted by clustering. Middle-click or right Shift selects up to four atoms for distances, connected bond angles and dihedrals. Input returns to the starting geometry.

Aligned ensemble displays every retained conformer as one cached static wireframe object. Cluster overlay uses the same renderer for every member of the chosen hierarchical cluster. Drag rotates the whole object, the wheel zooms, and right-drag pans. Geometry is compiled once when the result is prepared; camera movement does not repeat fitting or clustering. Overlap shows spatial spread under the selected fit. It is not a quantitative density, population or completeness estimate.

## Hierarchical fingerprints
Optional hierarchical clustering reduces the number of conformers passed to the expensive AIMNet2 stage. It complements the search’s RMSD grouping by organizing shapes using a normalized distance/dihedral fingerprint. None, the default, retains all search minima. After choosing a preset or custom settings, Run clustering computes the tree and representatives. Changing a clustering setting invalidates the previous selection until it is recalculated.

Distance fingerprints contain all pairwise distances among atoms other than carbon-bound hydrogens. Dihedral fingerprints contain sine and cosine values for paths across heavy-atom bonds, choosing the first available heavy-atom neighbor at either end. Each block uses an RMS difference between conformers and is normalized by the largest block difference across this ensemble. The combined dissimilarity is √[(1−α)D² + αT²], where α = w/(1+w) and w is the dihedral weight. Without varying torsion information, only the distance block contributes.

The relative cutoff τ is a fraction of the largest merge height of that ensemble’s linkage tree. A larger τ generally gives fewer clusters. It is not an Å threshold and cannot be compared directly with the search’s RMSD setting. Because normalization and the cut are ensemble-relative, adding other shapes can change the partition. Clustering selects geometric coverage; it does not establish that two conformers have identical shifts or interchangeable solution populations.

## Linkage philosophy
Complete linkage uses the farthest cross-cluster pair. It favors groups whose members are all mutually similar and limits chaining, making it the basis of both presets. Single linkage uses the nearest cross-cluster pair; it can connect a series of locally similar shapes into a broad group. Average linkage uses the size-weighted mean of all cross-cluster distances, balancing overall similarity against extreme pairs.

Weighted linkage gives equal weight to the two merged clusters when updating distances, independent of how many original conformers they contain. Ward uses its variance-based distance update to favor compact groups on the combined fingerprint representation. These methods express different ideas of geometric similarity and can yield different representatives from the same ensemble.

Compare methods reports Ward, complete, average and single cluster counts at the chosen cutoff and uses average linkage for the actual representative selection. It does not vote across methods or form a consensus partition. Pairwise fingerprint work uses a bounded worker pool and a memory-admission estimate with headroom.

## Presets and representatives
Standard: complete linkage, τ = 0.40, tiered selection, dihedral weight 0.50. This is the broader reduction preset, retaining extra representatives for clusters with substantial internal spread.

Conservative: complete linkage, τ = 0.15, lowest-energy selection, dihedral weight 0.50. The lower relative cut tends to form smaller, tighter clusters. One lowest-energy representative is retained per cluster.

Lowest energy selects the lowest-numbered member. Here numbering follows the Xsampler force-field energy ranking, so that rule selects the actual lowest force-field-energy member of each cluster. Medoid selects the member with the smallest sum of fingerprint distances to other members, which represents the group’s central geometry rather than its energy minimum.

Tiered selection always retains the lowest-energy member. If mean pairwise cluster spread is at least 0.12 in normalized fingerprint units, it also retains the medoid when different. At spread at least 0.18, it adds the remaining member farthest from the medoid. Thus it preserves one to three representatives according to internal geometric diversity. Deterministic source-order tie-breaks are retained.

The list marks representatives with a filled circle. All cluster members remain inspectable individually or in a cluster overlay. Omitted members do not receive an inferred degeneracy weight: the job setup tab weights distinct retained AIMNet2 minima using their own electronic energies. Reducing the ensemble can therefore change its population balance as well as its geometric coverage. Inspect the selections and compare with an unclustered run when that sensitivity matters.

## Files and transfer
Save conformers writes either compound_1.sdf, compound_2.sdf, … or one compound_confs.sdf. Original energy-ranked conformer numbers are preserved in individual filenames and record titles, including gaps caused by selection. Choose SDF V2000 or V3000. Existing files are protected; all files are staged before publishing. An applied clustering result also saves compound_clustering.txt with settings, memberships and representatives.

Records retain energies in kcal mol^{-1}, relative energies, seed, actual search settings, source proposal, gradient and full-precision aligned coordinates. Displayed SDF columns can be rounded; TENSOR restores the full-precision coordinate property when reloading searched conformers and uses those coordinates for Gaussian job setup. User atom ordering, charges, isotopes and compatible annotations are preserved. A new search replaces inherited TENSOR, Workbench, AIMNet2, Xsampler and clustering result properties, so newly generated geometries receive their own sampling seeds and source numbers.

Use in job setup transfers the selected conformer identities and full-precision coordinates directly to the second tab. The original structure file remains their source, so a blank Gaussian output destination writes beside that real file. Save conformers provides persistent SDF copies of the search result. Switching tabs retains the search result and viewing state. Job setup can also load conformers produced elsewhere; Analysis & 3D view processes the completed Gaussian logs.

## References and scope
MMFF94: T. A. Halgren, Journal of Computational Chemistry 17, 490–641 (1996), MMFF94 series. MMFF94s: Journal of Computational Chemistry 20, 720–729 (1999). Official reference data: https://server.ccl.net/cca/data/MMFF94s/.

UFF: A. K. Rappé and coauthors, Journal of the American Chemical Society 114, 10024–10035 (1992). BAOAB Langevin integration: B. Leimkuhler and C. Matthews, Applied Mathematics Research eXpress 2013, 34–56. Those methodological references are distinct from the supplied kernels’ implementation-specific scope.

Gaussian calculation models and NMR analysis assumptions are documented in the other tabs’ Methodology sections. Xsampler Lite source attribution, component identities, parameter-table origins, regression fixtures and third-party notices are retained with the source. Finite-budget sampling, force-field ranking and optional geometric clustering are practical approximations whose suitability depends on the molecule and intended use.
