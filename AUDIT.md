# TENSOR 2.0.0 — release review

The public release is based on the validated 1.2.1 source. Gaussian generation/parsing, populations, equivalence, clustering, NOE calculations and the spin-Hamiltonian solver retain their existing algorithms and numerical constants. Developer credits remain Jared Wood and OpenAI GPT-5.6 Sol. Current validation is in VALIDATION.md; the previous public audit remains in AUDIT-1.0.0-public.md.

## Changes and verified fixes

- The Methodology NOE multiplicity factor uses small, lowered i/j indices and a small, raised −1/6 exponent in one wrapping text layout. The slash is raised with the rest of the exponent. Text exports retain their ASCII formula for compatibility; numerical results are unchanged.
- Built-in SSH profiles and workflows are removed. The config contains only commented generic examples, so fresh installs have empty host/method selections. The user is directed to Configuration to define a site. Existing custom configurations remain intact.
- Documentation, tests and rendered examples use generic placeholder hosts/accounts. Example profiles used in UI tests/renders are isolated in a fixture, rather than becoming startup defaults. Current source, Cargo lock entry, Windows resources, distribution folders and GitHub exact-archive workflow identify version 2.0.0.
- Local and remote transfer staging names use the operation ID and file index instead of repeating the original filename. A valid long basename can no longer exceed the filesystem component limit solely because staging added a prefix and suffix. Exclusive creation, cleanup ownership and no-overwrite commits are retained.
- The dedicated-remote-directory check rejects slash-only root spellings (`/`, `//`, `///`). Such paths must never qualify as an application working base.
- Malformed configuration-section diagnostics now include the supported SSH/workflow section types.

## Reviewed behavior

Configuration migration adds missing retained method/basis fields without overwriting user settings or injecting hosts. Worker completion/disconnection, cancellation, stale spectrum results and window state are checked by regressions. Source and result-folder protections, finite/scaling validation, no-overwrite file transfer, inventory/content verification and cancellation guards remain active. Temporary artifacts are excluded from release archives. Runtime distributions contain the bundled public config, never a discovered user config or backup.

The former receipt-based SSH functions are retained for backwards configuration/API compatibility and tested separately; they are not required by the current GUI. Removing them would be a separate compatibility change. Search-budget limits, exact-spin limits and scientific approximations remain documented rather than being silently changed during code cleanup.

A code review and passing tests do not prove the absence of every bug. Native Windows rendering, actual HPC/server policies, real Gaussian data and concurrent remote writers beyond the final check remain environment-specific limits. See VALIDATION.md for checks actually performed.
