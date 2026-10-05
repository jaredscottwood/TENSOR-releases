# TENSOR 2.0.0 — optional SSH/HPC methods

The public configuration contains only commented generic examples. No hosts, accounts or workflows are active by default. No connection is made on startup or when Configuration is opened. Host keys are verified before authentication using an OpenSSH known_hosts file or an independently verified SHA256 fingerprint.

## Configure your site

Open Configuration and uncomment/edit the SSH profile and workflow examples. Replace every placeholder with your site's values; the remote base must be a dedicated directory that already exists. The SSH username need not match your local Windows username. Example domains are placeholders, not a service supplied with TENSOR.

```ini
[ssh:my_hpc]
host = hpc.example.org
port = 22
user = your_username
auth = password
password = <prompt>
known_hosts = C:\Users\your_username\.ssh\known_hosts
remote_base = /home/your_username/TENSOR
timeout_seconds = 60

[workflow:submit_gaussian]
label = Gaussian — upload and submit
profiles = my_hpc
step1 = upload
step1_extensions = gjf
step2 = run
step2_command = "$HOME/bin/asubmit.sh"
step2_background = false

[workflow:retrieve_logs]
label = Gaussian — retrieve and clean up
profiles = my_hpc
step1 = download
step1_pattern = *.log
step2 = cleanup
step2_pattern = *.log
step2_guard_pattern = *.log
```

The submission command is an example: replace it with your scheduler/site script. The retrieval example deletes only the verified LOG filenames through SFTP. Omit the cleanup step entirely to keep the remote copies. SDF methods can instead use `step1_extensions = sdf` for upload and `*_confs.sdf` for retrieval. Setting the last run step's `stepN_background = true` launches it with nohup, detached stdin and a uniquely named output log in the same base. A returned PID acknowledges launch, not successful completion; use your site's scheduler/status notification to decide when outputs are ready.

Methods are filtered by their `profiles` assignment and checked again by the backend. TENSOR creates no remote subdirectory. Pickers and download destinations initially use the executable's directory. Upload/submission methods appear before retrieval methods. Selected inputs and commands are collapsed to keep the window compact; expand them to review. Test connection verifies the host key and login without transferring files or running a command.

Closing an idle window clears its state. Existing user configuration is retained across app updates, including custom SSH profiles. To start from public defaults, use the supplied config in a new distribution folder or replace a saved config deliberately after retaining any settings you need.

## Passwords and host verification

```
password = <prompt>
```

This uses the masked session field. Alternatively, a literal `password = ...` is used automatically with `auth = password`. Literals are plaintext in the editable configuration and any configuration backups; they are not encrypted. A config password hides the session-password row until changed to `<prompt>`. Leading/trailing configuration whitespace is trimmed by the INI parser. Key passphrases remain session-only. Session values are cleared after an operation, host change, or closing the window; worker copies use zeroizing buffers. Passwords are not added to commands or receipts, and Profile debug output is redacted.

The configured known_hosts paths must exist on the machine running the executable. Unknown or changed keys are rejected. Verify new host keys with the HPC administrator and update known_hosts outside TENSOR. A `fingerprint = SHA256:...` pin is also supported; if both a file and pin are supplied, both must match. Password correctness cannot resolve a failure that occurs before authentication.

Authentication supports direct password, private-key file, and SSH agent modes. Interactive MFA, jump hosts and OpenSSH config aliases are not implemented. No administrator elevation is requested by this feature.

## Numbered-step configuration

Methods stay in `[workflow:NAME]` sections. For example:

```
[workflow:retrieve_logs]
label = Retrieve and clean up logs
profiles = my_hpc
step1 = download
step1_pattern = *.log
step2 = cleanup
step2_pattern = *.log
step2_guard_pattern = *.log
step2_command = "$HOME/bin/log_cleanup.sh"
```

Steps must be consecutive, from step1 to a maximum of step16. Supported types:

| Type | Fields | Behavior |
|---|---|---|
| upload | stepN_extensions = sdf or gjf | Upload the selected inputs directly into remote_base, without overwriting existing remote filenames |
| run | stepN_command, optional stepN_background = true/false | Run the trusted site command from remote_base; a background run must be the last step |
| download | stepN_pattern | Flat, case-insensitive filename matching with * and ?; no recursive paths |
| cleanup | stepN_pattern, optional stepN_guard_pattern, optional stepN_command | Require a preceding download of the same pattern and verified local copies; without a command, selectively remove those filenames by SFTP |

`profiles` is a comma-separated list of permitted SSH profile names. An empty assignment permits any configured profile; the shipped methods have explicit assignments. Multiple foreground run steps can be chained. An asynchronous long calculation belongs in a separate method from its later retrieval.

Commands are intentional trusted user/site shell input. TENSOR wraps them in a shell-quoted `cd` into the selected base and supports one-pass `{files}`, `{remote_dir}` and `{receipt_id}` substitutions; `{receipt_id}` is an operation identifier in the method runner. Substitution strings cannot introduce further placeholders. File/remote-path arguments are quoted individually. Scripts must already exist and be executable on the HPC, or the command can explicitly use `bash "$HOME/bin/script.sh"`.

Legacy submit_command definitions without numbered steps can be converted for simple direct-base operation. Definitions requiring legacy batch scripts or nested output directories report that conversion is required. Old receipt APIs remain in the source for their original format and tests, but the new GUI does not create, open or depend on receipts. Use the commented profile/workflow examples in the shipped config as templates for your own site in Configuration; existing edits are preserved automatically.

## Retrieval and cleanup guarantees and limits

The Method dropdown lists upload/submission methods before retrieval methods, based on their steps rather than their names.

Matching output files must be regular files with portable filenames, with no traversal, symlinks or case-colliding names. Individual transfers are limited to 2 GiB and a directory to 10,000 entries. Downloads stage locally with a 1 MiB read buffer and a streaming local SHA-256, then commit without overwriting an existing target. With `sha256sum` available, batched server checksums are compared to the streaming hash; the complete new payload crosses SFTP once. Stable size/mtime and inventory checks still apply. Without the utility, staged files are compared byte-for-byte with 1 MiB buffers instead. An existing identical copy is verified and reused; a different existing copy stops the method. Earlier verified downloads remain available after any later failure.

Immediately before cleanup, the output inventory must match the downloaded inventory, and all remote SHA-256 values and local copies are rechecked (or compared byte-for-byte again on fallback hosts). The inventory is checked again after hashing immediately before deletion. Any partial transfer, conflicting file, changed output, extra arrival, symlink or missing verified copy blocks cleanup. Empty retrieval skips cleanup entirely. No command is automatically retried after a disconnect, timeout or ambiguous result; inspect the remote working base/scheduler before retrying.

If a site cleanup script removes all SDF files but retrieval selects only *_confs.sdf, set its guard to **all *.sdf**. An unretrieved input or other SDF then blocks the script and local downloads are retained. Omit stepN_command to use selective SFTP cleanup of only the verified output filenames.

A configured shell script is arbitrary site code: TENSOR cannot prove what it deletes or prevent a producer from changing the directory in the instant after the final check. Do not run a broad cleanup script while another job writes into the base. Site scripts are supplied and maintained by the user or administrator. Cleanup failures preserve local copies and may leave a partly cleaned remote directory; inspect it before retrying. Background-launch log files are retained for diagnosis and are not Gaussian LOG matches.

## Modern SSH compatibility

Windows builds now statically bundle OpenSSL instead of libssh2's narrower default Windows crypto backend. This adds modern Curve25519 key exchange and Ed25519 host/key handling to the Windows client. The method runner prefers supported modern algorithms and omits SHA-1 KEX and group-exchange negotiation from its defaults. Host-key checking remains mandatory.

The allowed preference list is:

```
curve25519-sha256,curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha256,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512
```

An optional profile `kex_algorithms = ...` can choose a comma-separated subset, for example `ecdh-sha2-nistp256,diffie-hellman-group14-sha256`, when an administrator recommends a specific supported policy. Test connection reports the negotiated KEX and host-key algorithms. A handshake failure reports the enabled client list and identifies the pre-authentication stage. It does not automatically retry a run/cleanup or weaken verification.

Compatibility depends on overlap between the client's supported algorithms and those permitted by the server. This client does not implement hybrid post-quantum algorithms such as `sntrup761x25519-sha512`; a server restricted exclusively to such algorithms requires a different approved client/backend. A failed negotiation message is not a complete inventory of server capabilities. Ask the site administrator which supported algorithms the server permits.

Sources: [ssh2 features](https://docs.rs/crate/ssh2/0.9.6/features), [libssh2 source/backend](https://docs.rs/crate/libssh2-sys/0.3.3/source/), and [OpenSSH 9.9 release notes](https://www.openssh.org/txt/release-9.9). No real HPC host was contacted during validation. Successful loopback tests validate client paths and guards, not a guarantee of compatibility with your host or site security policy.

## Window lifecycle and transfer performance

Closing an idle SSH/HPC window resets selections, inputs, destination, messages and step history. Closing it or exiting TENSOR during active work asks whether to keep working or stop and close. Confirmation interrupts the TCP connection and checks cancellation before starting each later step; completed local files remain, and the current local `.part` is discarded. Remote uploads interrupted after connection loss may leave a uniquely named `.tensor-upload-...part` file that can be removed later. DNS resolution and a TCP connect already in progress may finish or time out before cancellation returns. Closing a connection cannot undo an already submitted scheduler job, reverse cleanup already performed, or guarantee that an executing remote script stops. Inspect the HPC before retrying any uncertain command.

The older retrieval read each new file four times across SFTP: download, staged-copy comparison, committed-copy comparison, and pre-cleanup comparison. The checksum path removes the three repeated network payload reads, keeping two server-side hash passes (before retrieval and before cleanup) and a local pre-cleanup read. A fallback host needs three SFTP passes for a new file (download plus two comparisons). Progress distinguishes preparing checksums, downloading, verifying existing copies, and rechecking before cleanup. Transfer speed reflects payload reads and elapsed retrieval time; file reuse is reported separately.

This improves a known client bottleneck, but does not promise WinSCP-equivalent throughput. VPN/network latency, HPC disk speed/load, server policy, libssh2 pipelining and local disk/endpoint scanning can still limit performance. Hashing itself reads the server disks, and large inventories require metadata operations. A checksum/metadata check is a snapshot: a producer can change files after the final check, and arbitrary cleanup scripts can delete outside their stated guard. Only a server-side locking/snapshot contract shared with those producers can eliminate that race.

`SSH_HPC.md` is optional human-readable documentation included beside README.md; it is not loaded by the application. Generic commented examples are in the bundled tensor_config.txt. Existing user configuration is retained, so adapt the examples to your own host and working directory in an already edited file.

## Checksum response handling

Checksum commands capture utility stdout before printing a per-request framed manifest. Surrounding SSH startup messages are excluded from parsing; the utility runs with `LC_ALL=C`, and `command` bypasses shell functions. Every frame must be complete and unique; every expected filename must have exactly one valid checksum, and a nonzero utility exit still aborts. A malformed record reports a short escaped excerpt. Initial verification and the pre-cleanup recheck both use this path for SDF and LOG retrieval.

The fast path reads each new file once over SFTP. Only hosts lacking `sha256sum` use the byte-comparison fallback; malformed hashes or failed commands do not silently select it. If a checksum error persists, retain its full message and remote outputs for investigation. No remote script is retried automatically.
