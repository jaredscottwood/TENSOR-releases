# TENSOR 2.0.0 — predicted spectra

Open **Analysis & 3D view**, select a result compound and press **NMR spectra…**. The window plots all supported nuclei with shift predictions, or a selected nucleus. It is a one-dimensional, fast-exchange solution-spectrum model. It does not simulate a pulse sequence or fit experimental spectra.

## Data and averaging

The included nuclei are ¹H, ¹³C, ¹⁹F and ¹⁵N. Input shifts are the combined ensemble predictions, including available carbon corrections, rounded to the same two decimal places as the viewer/PyMOL labels. Every included atom contributes one spin. Hydrogen, fluorine and nitrogen use unit integrated model area. Carbon uses qualitative protonation weights: quaternary C 0.45, CH 1.00, CH₂ 1.15, and CH₃ or higher 1.30. These are visual response factors, not quantitative experimental sensitivity or relaxation corrections. Atoms without a shift have no observed peak. Bonded H/C atoms can still supply coupling-only partners, including in proton-only or carbon-only shift data. Their placeholder shift is never observed; unknown same-nucleus shifts and associated homonuclear mixing are not invented. Explicit unsupported isotopes are omitted; there is no spin-1 deuterium or quadrupolar ¹⁴N simulation.

For a current analysis, the simulator obtains J values from the raw, atom-resolved Gaussian matrices retained with the accepted logs. It weights each pair over the positively populated standard conformers, using the analysis populations after exclusions and deduplication. A pair must exist in every contributing conformer to be accepted as a calculated average. Zero-population conformers cannot suppress that average. Carbon correction ensembles affect the combined carbon shifts; they do not supply a separate J ensemble.

A reopened result can supply its saved weighted coupling file. That file has already undergone chemical-equivalence averaging. Magnetic inequivalence lost in that export cannot be reconstructed; reanalyzing the original logs retains the raw assignments for simulation. Keeping the conformer SDFs, PyMOL labels, weighted coupling file and population summary permits reopening. Current-run spectra do not depend on export switches.

The fast-exchange assumption means parameters are averaged before simulation. This is not an incoherent sum of independently simulated conformer spectra, nor does it include exchange broadening.

## Optional missing proton estimates

A calculated pair always takes precedence over its estimate. For an otherwise missing pair, an explicit H–C(sp³)–C(sp³)–H path can supply

\[
{}^3J_{HH}(\theta)=7.76\cos^2\theta-1.10\cos\theta+1.40\quad\mathrm{Hz}.
\]

Both carbons must have four explicit neighbors and single bonds. Degenerate dihedrals are excluded. TENSOR evaluates J separately in each conformer and then population-weights the resulting couplings; it does not average the angles first.

The same switch supplies a conservative fallback for carbon-bound hydrogens on a six-membered aromatic C/N ring when their pair is absent from the calculated inventory. This includes fused rings and aza-aromatic rings whose bonds to a ring nitrogen are represented as single rather than delocalized. A candidate must be a six-membered chordless C/N cycle with two cycle neighbors at each site and at least three internal bonds having order above 1.3. Ring-path separations use 7.54 Hz for adjacent, 1.37 Hz for two-edge, and 0.69 Hz for opposite pairs. These are benzene-derived screening values, not heteroaromatic- or substituent-specific predictions. Calculated pairs always win. This H–H option does not estimate N-bound, geminal or exchangeable-proton values. The separate one-bond C–H option is described below; all other missing heteronuclear J values remain unknown. These unmodeled couplings are unknown, rather than experimentally established zeros.

The three aromatic values come from [Read, *A precise determination of the proton coupling parameters in benzene*, Journal of Molecular Spectroscopy (1967)](https://doi.org/10.1016/0022-2852(67)90188-9). Extending them to qualifying C/N rings is an explicitly approximate fallback, not a claim that benzene and every aza-aromatic system have identical couplings.

This is a simplified Karplus curve, without substituent/electronegativity corrections. Its use in published work is documented by [Crich and Vinod, Organic Letters (2007)](https://doi.org/10.1021/ol070427b). The broader relationship and substituent effects are discussed by [Haasnoot, de Leeuw and Altona, Tetrahedron (1980)](https://doi.org/10.1016/0040-4020(80)80155-4). These sources support the form and limitations of the model; they do not establish the accuracy of estimates for a particular molecule.

**Fast methyl / CF₃ rotation** optionally averages couplings of the three equivalent terminal sites. An external partner must have all three pair values, and those values must all be calculated or all estimated; incomplete or mixed inventories remain unaveraged. Only existing pair entries are changed. General chemical equivalence of shifts is not treated as magnetic equivalence of all couplings.

## Spin Hamiltonian

Connected components containing at most ten coupled spins are diagonalized numerically. In frequency units, the implemented Hamiltonian is

\[
\mathcal H/h=\sum_i\nu_i I_{zi}
+\sum_{\text{same nucleus}}J_{ij}\left(I_{zi}I_{zj}+\frac{I_{+i}I_{-j}+I_{-i}I_{+j}}2\right)
+\sum_{\text{different nuclei}}J_{ij}I_{zi}I_{zj}.
\]

Here ν = δ × observation frequency in MHz. Homonuclear isotropic coupling includes the flip-flop terms, allowing second-order patterns and roofing. Heteronuclear coupling uses the high-field secular approximation. Isotropic solution spectra and the high-temperature transverse-detection approximation are assumed. The implementation is independent Rust code, not a wrapper around a separate simulation package. [Kuprov's liquid-state simulation tutorial](https://spindynamics.org/documents/sd_m2_lecture_09.pdf) provides background on Hamiltonian-based NMR simulation and more complete treatments.

The solver separates total-magnetization sectors and uses a real symmetric Jacobi eigensolver. Transitions use the squared matrix elements of the summed lowering operator for the observed species. The observation operator includes each spin's square-root area weight, so the integrated carbon response follows the qualitative protonation factors above. This does not represent different nuclei's experimental sensitivities, relaxation, NOE enhancement, or isotope abundances.

Components larger than ten spins use a labelled first-order approximation. The **Force first-order approximation** control applies that method to every component. Each directly coupled spin-½ partner gives ±J/2 splitting; equivalent partners naturally produce binomial multiplets. Mutual splitting is omitted only for spins with the same nucleus, same shift, and equal couplings to every other site. The window warns when a first-order system contains a magnetically inequivalent homonuclear pair with Δν/|J| < 10. This ratio is a screening indicator, not a guarantee of first-order accuracy above that value.

Pairs below the selected absolute-J cutoff are omitted before finding connected components. Raising it can reduce solver size but also discards real splittings. The default is 0.1 Hz. First-order expansion is bounded to prevent uncontrolled transition growth; an excessive expansion reports an error and preserves the previously displayed spectrum.

**Force first-order approximation** applies independent ±J/2 splitting to every component instead of diagonalizing its spin Hamiltonian. It uses equal binomial multiplet weights and discards roofing, second-order shifts/intensity redistribution, and magnetic-inequivalence effects. This assumes weak coupling: chemical-shift separation Δν (in Hz) must be much larger than |J|. A Δν/|J| ≥ 10 rule is only a screen, not proof of accuracy. With the option off, components of up to 10 spins use the exact solver and larger components still fall back. Raising the field increases Δν without changing J; decoupling can shrink a connected component.

## Isotopes, field and decoupling

| Setting | Meaning |
|---|---|
| ¹H frequency | Sets field-dependent ppm-to-Hz conversion; default 600 MHz |
| Frequency ratios | ¹³C 0.25145020, ¹⁹F 0.94094011, ¹⁵N 0.10136767 relative to ¹H; positive frequency magnitudes |
| Natural-abundance mode | H spectra can optionally include ¹³C partners with abundance weighting; F spectra exclude rare C/N partners. C/N observation conditions on its observed site carrying that isotope. |
| Show ¹³C satellites in ¹H | Optional isotope-weighted proton sidebands; default off |
| Missing one-bond C–H J | Estimated automatically when needed; calculated values, including zeros, retain priority |
| Decouple matrix | Each row is an observed spectrum; each checked partner column is removed from that spin system |
| Defaults | Proton decoupling for carbon and nitrogen; fluorine couplings retained when available |

The GUI uses natural-abundance behavior. The unused fully enriched GUI checkbox has been removed; an idealized fully labeled model remains available internally for the independent solver test fixture.

### Carbon satellite approximation

The representative ¹³C amount fraction is **p = 0.0107**. CIAAW reports natural terrestrial carbon isotope variation (¹³C fraction 0.0096–0.0116), so this is a display assumption rather than a measurement of the user's sample. With one coupled carbon site, a proton contributes 98.93% main area and a satellite at each ±J(C,H)/2 with 0.535% area. Existing H–H multiplet structure and strong-coupling behavior are recomputed in the isotope-specific spin systems; satellites are not blindly copies of every main-system line.

Each connected system mixes the all-¹²C state and individual ¹³C states. Up to 30 carbon partners, simultaneous pairs of ¹³C sites are also included. Above 30 partners only single-site states are retained; more than 128 carbon partners reports a bounded-model error. A state with k labeled sites has weight pᵏ(1−p)ᵐ⁻ᵏ, where m is the carbon-partner count. Retained weights are normalized to conserve model area. One summary per proton spectrum reports the number of connected systems and calculated/estimated C–H pair counts. Collapsed model details report the carbon-partner range and minimum retained abundance across all those systems; the inventory supplies actual atom numbers and individual sources. Solver counts include calculations for isotope states, rather than a count of distinct molecular fragments. The retained abundance matters because omission is usually small for small molecules but becomes material in large connected systems. This approximation excludes ¹⁵N satellites, ¹³C–¹³C satellites in carbon observation, isotope-induced chemical-shift changes, enriched-sample modeling and isotope-dependent changes of J.

Calculated C–H J values always win, including a calculated zero. Missing directly bonded C–H values automatically use **125 Hz for single-bond carbon, 160 Hz for double/aromatic carbon, 250 Hz for triple-bond carbon**. MDL aromatic bond type 4 is treated as aromatic, not as a triple bond. These are coarse bond-class screening values, not calibrated predictions; heteroatom substituents, strain and aldehydes can differ substantially. Missing long-range C–H J values are not estimated. Each estimated pair is labeled “One-bond C–H estimate” in the inventory. The satellite checkbox controls whether those carbon partners contribute to proton spectra; coupled carbon spectra can use these values when proton decoupling is off.

The ranges are consistent with the [Hebrew University NMR laboratory's heteronuclear coupling explanation](https://chem.ch.huji.ac.il/nmr/whatisnmr/hetcoup.htm) and [Oregon State's NMR course notes](https://sites.science.oregonstate.edu/~gablek/CH362/NMR/bare_2DNMR.htm). Abundance assumptions are grounded in [CIAAW's carbon isotope table](https://www.ciaaw.org/carbon.htm). These sources provide screening ranges, not molecule-specific accuracy.

Carbon decoupling in the **¹H row / ¹³C column** removes the satellites. Proton decoupling in the **¹³C row / ¹H column** removes C–H splitting of the carbon spectrum. Carbon observation is proton-decoupled by default. If no usable J values remain (missing, zero, below cutoff), or carbon satellites were already off, a corresponding control can correctly have no visible effect. The assumptions show the usable-pair count for enabled decoupling.

The reference frequency ratios follow [IUPAC NMR nomenclature and reference conventions (2001)](https://doi.org/10.1351/pac200173111795); the ¹⁵N ratio uses the nitromethane reference convention. Input chemical shifts are used as supplied and are not re-referenced by this conversion.

**J isotope convention matters.** Gaussian's total J matrix is isotope-dependent, whereas its K matrix is isotope-independent, as described in the [Gaussian NMR documentation](https://gaussian.com/nmr/). TENSOR uses the reported J matrix without inferring or converting its isotope convention. Nitrogen splitting must therefore be based on J values appropriate for ¹⁵N; a calculation using ¹⁴N J values cannot be interpreted as a ¹⁵N prediction unchanged. Verify the job's isotope settings. The interface identifies this limitation when nitrogen shifts are present.

## Residual-solvent markers

TENSOR parses `solvent=...` from the accepted standard Gaussian logs. A conventional residual-solvent marker is available only when every accepted standard conformer contains exactly one solvent keyword and all agree. Supported normalized keywords are chloroform, DMSO, acetonitrile, water, methanol, benzene, acetone, tetrahydrofuran, and dichloromethane. Nucleus-specific markers come from common room-temperature reference values; some solvents have more than one ¹³C marker.

The marker is an optional comparison guide. It is not added to the molecular spin Hamiltonian, does not couple to the compound, and does not recalibrate predicted shifts. Solvent peak positions vary with temperature, concentration, water content, and referencing. Missing, multiple, conflicting, or unsupported SCRF solvent keywords disable the marker and add an explanatory note.

## Line shapes, interaction and exports

The linewidth is Lorentzian FWHM in Hz for all nuclei. Plots use bin-integrated Lorentzians so lines much narrower than a screen pixel retain their area. The finite drawing range omits tails farther than 500 half-widths; the discrete transition CSV is unaffected by plot sampling. Transition merging tolerances are 10⁻⁷ Hz for the exact solver output and 10⁻⁶ Hz during first-order expansion. Extremely weak exact transitions below 10⁻¹⁰ relative area are omitted.

The horizontal axis descends in ppm from left to right. Left-drag a horizontal region to expand it, turn the wheel over a plot to scale its vertical intensity from 0.05× to 50×, and right-drag to pan. The selection origin is retained until button release, even when the pointer moves across frames or leaves the plot, and plot-local wheel input is consumed before the enclosing spectrum list can scroll. Double-click restores that plot and its intensity; **Zoom out to full spectrum** restores every horizontal range. Zoomed curves are regenerated with a bounded 4096-sample display, while the full-view curve is precomputed once per simulation. Screen painting caches a min/max envelope per horizontal pixel bucket so narrow lines survive downsampling without rebuilding the entire high-resolution trace on every frame. Scientific CSV/SVG export uses the complete calculation rather than that display envelope. Closing the spectrum window drops its compound-derived input, simulation, and plot caches. Hovering identifies the nearest **unsplit** atom shift and any nearby residual-solvent marker; it is not a full transition assignment or multiplet-labeling algorithm.

Controls recompute live in a worker thread after a 150 ms debounce. Superseded calculations are cancelled and stale results are ignored. Updates retain the selected nucleus, horizontal range/pan and vertical intensity; cached curve data alone is rebuilt. There is no Update spectra button. Cancel or a failed update retains the previous spectra and their applied parameters. **Transition CSV** and **Spectrum SVG** export the displayed calculation, even when new controls have not yet been applied. CSV contains discrete frequency, ppm and relative-area records with comment metadata. SVG contains vector traces and embeds model/settings metadata in its description. Isotope prefixes and molecular-formula digits/signs are written as semantic SVG `<tspan>` super/subscripts rather than relying on precomposed Unicode glyphs, so standards-compliant viewers retain the intended notation even without TENSOR's bundled font fallback.

## Accuracy and validation

The tests include analytical AB roofing, first-order binomial intensities, heteronuclear decoupling, magnetic inequivalence, population weighting, natural-abundance behavior, Karplus limits, cancellation and Lorentzian area/FWHM. An independent NumPy calculation constructs a complete seven-spin Hilbert-space Hamiltonian with Kronecker products and compares every nucleus against the Rust sector solver. See `VALIDATION.md` for the actual results.

Numerical agreement establishes implementation consistency for those cases. It does not validate the input DFT shifts, calculated or estimated couplings, conformer populations, solvent model, or a real molecule's experimental spectrum. The model excludes relaxation matrices, anisotropic interactions, chemical-exchange line shapes, pulse sequences, quadrupolar nuclei and isotope effects beyond the documented carbon-satellite approximation.
