# TENSOR 2.0.0 — public release validation

Validated on 2026-10-05 UTC on Linux x86-64 with Rust 1.95.0 and the pinned Cargo.lock dependencies. This release starts from the validated 1.2.1 source. Scientific algorithms and numerical constants are retained; the release changes public defaults, mathematical text formatting, transfer staging names and directory validation. See AUDIT.md for the review findings.

## Checks performed

- Release test suite: **114 regular tests passed** (50 library, 19 audit, 23 regression, 22 spectrum). Four SSH/SFTP integration tests are opt-in and run separately.
- Isolated loopback SSH/SFTP fixture: **all four integration tests passed**, using Paramiko 5.0.0 with throwaway local credentials. LOG and SDF retrieval accept surrounding startup stdout/stderr. A 36-file SDF case exercises multiple 32-file checksum batches with spaces, apostrophes, Unicode and a 240-byte basename. Upload also exercises a long quoted basename. Short staging names preserve these valid source filenames.
- Malformed, missing and duplicate checksum records, truncated/duplicate frames, and a nonzero exit with a plausible complete frame stop before commit or cleanup. A malformed pre-cleanup recheck retains both the local copy and the remote output. Content mutation, corrupt hashes, missing-utility fallback, cancellation, no-overwrite, inventory and broad-script guards pass.
- An instrumented **8 MiB LOG transferred exactly 8,388,608 SFTP payload bytes**, including cleanup verification: one payload pass. The local fixture completed that operation in 0.479 s. This is a loopback observation, not a real-HPC speed prediction.
- A real POSIX shell/sha256sum test checks command framing, quoting, leading-dash filenames and propagation of a missing-file failure without a success frame.
- Public first-run configuration has no active SSH profiles or workflows and validates without errors. Configuration migration retains custom host/workflow sections and preserves the previous text in its backup. Slash-only remote root spellings are rejected. Method ordering and profile filtering are tested using a generic fixture separate from the built-in config.
- Rust formatting and strict Clippy for all targets with warnings denied passed.
- All **seven built-in self-check groups passed**.
- Independent Python reference comparison: **2,065 exported numeric values across 44 files matched** for two synthetic ensembles.
- Independent NumPy full-space spin-Hamiltonian comparison passed for H/C/F/N: 288/146/176/142 transitions respectively. Maximum frequency error was 1.25 × 10⁻¹⁰ Hz; maximum relative intensity error was 4.99 × 10⁻¹¹.
- The actual egui interface was rendered headlessly into the **25 included screenshots**. The NOE formula's raised exponent and lowered indices, the unconfigured public SSH window and the generic laptop SSH example were visually checked. Demonstration host/account values come from a test-only fixture.

## Scope

No real company host, scheduler, credentials or cleanup script was contacted. No native Windows executable was built or run in this Linux workspace. The supplied GitHub workflow requires exactly `TENSOR-2.0.0-source.zip` at the repository root, verifies its root and Cargo version, and builds/tests on Windows with Rust 1.95.0 and Perl for bundled OpenSSL. A native Windows build and real-site SSH/HPC use remain environment-specific checks.

These checks cover synthetic/reference datasets and targeted regressions; they do not establish correctness for every possible Gaussian output or molecule. Download throughput still depends on network latency, server disk/hash time, SSH pipelining, local disk and endpoint scanning. An arbitrary site cleanup script and a concurrent write after the final verification remain outside the client's guarantees; removing that race requires a server-side locking/snapshot contract. Commands are not retried automatically after uncertain execution.

This validation is not a Microsoft Defender review or a guarantee about antivirus classification. The public package contains the generic bundled config, with no user configuration or backups.
