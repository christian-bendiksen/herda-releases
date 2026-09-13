# Native system update qualification: dev-20260913.7

Current installation media starts with native boot evidence. The first complete
update uses the same single-trial-reboot flow as subsequent updates. The portable
updater and two-pass migration for older development images have been removed.

## Exact artifacts

- Release: `dev-20260913.7`, publisher sequence `7`, updater API `2`.
- Signed manifest SHA-256: `e2170c72d20d9992abbf44ab23dd55044b1115e26b469a0a04790c5993de57e1`.
- USB image: `herda-installer-native-r3-x86_64.img`, 2,409,324,544 bytes.
- USB image SHA-256: `657c8c4517666073c63c77cd97ad0a193e0e3af5037ccde9ffb127911827b779`.
- Image and release Herda source: `a7c07dacd50635efa694ec9b83621ea09a55d0a7`.
- Publisher fingerprint: `60ad738c0df1ee6d782811203ab084d08f0f0917`.
- Package snapshot: `2026-09-12`; kernel `7.2.4-arch1-2`.

| Component | Source commit |
| --- | --- |
| gnist | `e3e29f4082a391bd70167cfeca6e70fe9d3b2d4e` |
| herda | `a7c07dacd50635efa694ec9b83621ea09a55d0a7` |
| herda-launcher | `e22bcd55da358e946937193832a7b15d961e31b5` |
| malm | `75c39a8fee61d48ad0e9b16cd7c60474db72c0ac` |
| mango | `641c36fcbbe508c46f27b435d5a68d4e9b32f59f` |
| smia | `5038290eaa658adf3de3652ecc9c23a11f2fcfea` |

Builds used clean pinned checkouts. Uncommitted development work was excluded.
The five signed artifact roles are native payload, substrate, kernel, desktop
runtime and Smia. No portable bootstrap executable or publisher private key is
included in the release. The runtime ABI check passed for 192 ELF files and 1,215
paths; GPT and EROFS checks passed for the USB image.

## Actual boot checks

The run used a fresh disposable 64 GiB virtual disk, KVM, UEFI and Secure Boot
disabled on September 13, 2026. The live installer ran its normal discovery,
review and disk-identity checks before installing. All later commands used the
installed `/usr/bin/pkg`; no diagnostic replacement, bootstrap service override
or manually installed signing dependency was used in this final run.

| Check | Observed result |
| --- | --- |
| Fresh installation | Passed. The initial 526-package system booted healthy with native boot evidence. OpenSSL and all required PE signing tools were available. |
| Snapshot update | Passed. The complete 541-package successor booted healthy with the exact release recorded as installed and the canonical projection matching authority. |
| Interrupted trial | Passed. The VM was stopped after authenticated userspace handoff, before health confirmation. The next boot recovered and confirmed the original 526-package revision. |
| Cancellation and retry | Passed. The failed trial was cancelled through `pkg update --cancel`, then the same signed release was imported, staged and booted successfully. |
| Full rollback | Passed. A native trial restored the original view and 526-package set, confirmed healthy, and cleared installed-release metadata. |
| Rolling update | Passed. After choosing the rolling source, the 541-package successor booted healthy and retained `https://geo.mirror.pkgbuild.com` as its source policy. |
| Configuration preservation | Passed. Nine tracked account, declaration, provenance, home and Malm-local files remained unchanged. Only the intentionally changed rolling source declaration received a new expected digest. |
| Boot parameters | Passed. Repeated transitions retained exactly one copy of each inherited console option. |
| Retained image readback | Passed twice. All four retained images were read completely after requesting cache eviction; hashes matched between passes. See the open kernel issue below. |

All 17 matrix steps completed successfully. The publication report binds all six
required qualification checks to this exact signed manifest. It also records the
additional readback and the open kernel issue. Local transcripts, state snapshots,
per-step exit records and image verification accompany the report.

Targeted validation passed: 90 native state tests, 191 installation tests, 127
package CLI unit tests, 31 CLI integration tests run serially, 29 setup tests and
six release-composition tests. Publisher tests verify historical signed sequence
checks without accepting retired artifact formats for client installation.

## Public delivery

GitHub publication completed at `2026-09-13T20:21:09Z`. The publisher verified
all 27 assets (6,046,075,281 bytes) and confirmed that the
release is immutable. The healthy installed VM's own `pkg releases check` then
verified the public feed and returned the exact manifest above. Native state
remained unchanged and no trial was staged by this read-only check.

## Known issue and limits

The VM emitted Btrfs/fs-verity `FILE CORRUPTED` kernel messages with zero-page
hashes while reading retained substrate images. The previous release's serial
logs contain similar messages. Two complete cache-evicted readbacks returned
identical hashes for every image, all boot/update/rollback checks passed, and
systemd reported no failed units. The cause remains unconfirmed; the messages
are not treated as harmless or as proof that every read path is sound.

The follow-up is to reproduce the warning with a minimal sealed file on this
kernel, compare demand reads and read-ahead, and verify a kernel correction before
changing the storage or mount policy. [The kernel's fs-verity documentation](https://docs.kernel.org/filesystems/fsverity.html)
describes its page-cache and read-ahead verification boundaries; the suspected
read-path interaction is an investigation direction, not a confirmed diagnosis.

This is a development release with a passed update workflow, not general storage,
Secure Boot or physical-hardware qualification. Older pre-release installations
must use current installation media. Desktop configuration retains its separate
Malm history.
