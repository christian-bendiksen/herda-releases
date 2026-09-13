# Installed-system qualification: dev-20260913.5

The first signed development release updates an existing native Herda
installation without reinstalling it. The tested artifact is available from
[Herda releases](https://github.com/christian-bendiksen/herda-releases/releases/tag/dev-20260913.5).
See [system update instructions](README.md) before the first migration.

## Exact candidate

- Version: `dev-20260913.5`; publisher sequence: `5`.
- Manifest SHA-256: `b2b7e474c81d3f86611904901e8d7e10efbe72dfa2fcc0fc65033adf541838f0`.
- Bootstrap SHA-256: `b5b4116de629720b513b20049710d2e7e56a07dc56821d2a23df86719d8622e2`.
- Publisher fingerprint: `60ad738c0df1ee6d782811203ab084d08f0f0917`.

| Component | Source commit |
| --- | --- |
| Herda | `9f173f6a57879e5379efd1529a9c90a5d475ac73` |
| Malm | `75c39a8fee61d48ad0e9b16cd7c60474db72c0ac` |
| Gnist | `e3e29f4082a391bd70167cfeca6e70fe9d3b2d4e` |
| Launcher | `e22bcd55da358e946937193832a7b15d961e31b5` |
| Smia | `5038290eaa658adf3de3652ecc9c23a11f2fcfea` |
| Mango | `641c36fcbbe508c46f27b435d5a68d4e9b32f59f` |
| SceneFX | `37ccd723bef49e6891156ffafce8f549f01446cc` |

The build used clean pinned checkouts. Uncommitted development work was excluded.
The native payload, substrate, locked kernel archive, desktop runtime, Smia source
and portable bootstrap were packaged and verified as six signed-manifest artifacts.

## Boot matrix

The matrix ran on September 13, 2026 against a disposable qcow2 overlay of the
retained September 10 installed system, using KVM and UEFI with Secure Boot
disabled. Its backing disk remained unchanged. The initial installation had 522
packages and schema 10. The exact signed bootstrap artifact was used for
migration; later checks used the shipped updater and retained recovery helper.

| Check | Result and observed behavior |
| --- | --- |
| Bootstrap upgrade | Passed. Real reboot and automatic health confirmation retained the original revision, kernel, view and 522 packages. Schema promotion followed successful helper confirmation. |
| Snapshot upgrade | Passed. A complete successor booted with 541 packages, the signed release recorded as installed, healthy state and a matching canonical projection. |
| Rolling upgrade | Passed. Resolution used the configured rolling mirror; the resulting 541-package system booted healthy and retained the rolling source policy. |
| Interrupted-trial fallback | Passed. QMP interrupted execution after authenticated userspace handoff, before health confirmation. The next boot recovered the original software and confirmed it; resuming the staged release subsequently succeeded. |
| Complete rollback | Passed. A real native trial restored the original view and all 522 packages, confirmed healthy. The original application declaration remained byte-identical. |
| Configuration preservation | Passed. Nine tracked identity/configuration/home files survived the transitions. For the rolling test, only the intentionally edited source-policy file received a new expected digest. |
| Cancellation and retry | Passed. A parameter trial was staged, cancelled and staged again without conflicting immutable boot evidence or changing the running software. |
| Native boot parameters | Passed. The retried trial booted with `loglevel=4`; subsequent rollback/update cycles retained exactly one copy of each inherited console/default option. |
| Managed Smia | Passed. The normal user service reconciled the managed source, recorded the exact release digest and retained profile `mango` and local overrides. |

The tracked files were machine identity, hostname, passwd/shadow, the application
and system declarations, source provenance, a home-directory note and Malm's local
overlay. Sensitive contents were not included in release assets.

All 24 matrix steps completed successfully. The digest-bound publication report
contains all six required checks as `passed`; additional checks record parameter
cancellation/retry, boot parameters and managed Smia. Local transcripts, native
state snapshots and per-step exit records accompany that report. A bounded wait
for asynchronous Smia completion corrected an initial harness timing race; the
service itself completed normally without a manual reconciliation.

The final trial-identity fix also passed 191 `pkg-install` library tests, 125
`pkg-cli` binary tests and 24 dependency-boundary tests. Canonical trial policy
identities distinguish new attempts without weakening immutable boot-evidence
validation. Command-line composition tests cover repeated updates and explicit
multiple-console settings.

## Public delivery

GitHub publication completed on September 13, 2026 at 18:24 UTC. The publisher
verified all 28 assets, totaling 6,058,973,870 bytes, before publishing and then
confirmed that the release was immutable.

From the upgraded installed VM, the shipped `/usr/bin/pkg` successfully verified
the public feed with both `releases check` and `update --check`. Discovery returned
the exact signed manifest above and zero pending package changes. A fresh HTTPS
download of the public bootstrap matched its expected SHA-256. The checks left
the canonical native state unchanged, healthy and running, with no staged trial.

## Operational limits

Older installations require two reboot stages: test the recovery helper, then
boot the full release. During the first stage, and after restoring pre-migration
software, use `/herda/state/distribution-release/bootstrap/pkg`; the original
`/usr/bin/pkg` predates the upgraded state format. Existing boot parameters are
preserved, but editing them from the original recovery environment requires
completing the upgrade first.

Desktop configuration recovery remains separate from system software rollback.
The boot matrix covers managed Smia; isolated tests cover custom-source and
conflict handling. This development release does not establish Secure Boot or
general hardware qualification.
