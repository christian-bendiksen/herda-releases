# Herda system updates

Signed development releases for updating an installed Herda system without reinstalling it. Desktop configuration remains managed separately through Malm and Smia.

The first release is [`dev-20260913.5`](https://github.com/christian-bendiksen/herda-releases/releases/tag/dev-20260913.5). It supports existing native Herda development installations with Secure Boot disabled. It preserves the configured snapshot or rolling update policy, application declarations, account identity, home files, selected Malm profile, and local overrides.

## First update from older installation media

The first upgrade has two reboot stages. First, Herda verifies a new recovery helper while retaining your existing software. Then it prepares and boots the complete new system.

Download the portable updater and public certificate. Verify the updater hash before running it:

```sh
release_url=https://github.com/christian-bendiksen/herda-releases/releases/download/dev-20260913.5
curl --fail --location "$release_url/b5b4116de629720b513b20049710d2e7e56a07dc56821d2a23df86719d8622e2.chunk" --output herda-pkg
printf '%s  %s\n' b5b4116de629720b513b20049710d2e7e56a07dc56821d2a23df86719d8622e2 herda-pkg | sha256sum --check -
```

After the checksum reports `herda-pkg: OK`, enroll the publisher and prepare the update:

```sh
curl --fail --location "$release_url/herda-release-authority.pgp" --output herda-release-authority.pgp
chmod +x herda-pkg
sudo ./herda-pkg releases enroll \
  --certificate ./herda-release-authority.pgp \
  --fingerprint 60ad738c0df1ee6d782811203ab084d08f0f0917
sudo ./herda-pkg update
```

When the updater says the recovery trial is ready, reboot:

```sh
sudo reboot
```

After logging back in, use the retained updater to prepare the complete release:

```sh
sudo /herda/state/distribution-release/bootstrap/pkg update --resume
```

When staging completes, reboot again. The original system remains the fallback until the new boot passes health confirmation.

```sh
sudo reboot
```

After that boot, the normal `pkg` command is the new updater:

```sh
sudo pkg status
sudo pkg releases status
```

The publisher fingerprint above identifies the release authority; a downloaded certificate alone does not establish trust. No private publisher or installer key is distributed. Your installation creates and retains its own private boot-signing authority.

## Subsequent updates

```sh
sudo pkg update --check
sudo pkg update
sudo reboot
```

Use `sudo pkg update --resume` after an interrupted preparation. Use `sudo pkg update --cancel` to cancel an unobserved trial. Downloads are verified, resumable, and protected against sequence rollback. A cancelled download retains replay protection and reusable cache data.

`sudo pkg update --packages-only` selects the ordinary package update path. Complete Herda releases replace the distribution runtime and boot substrate together; snapshot installations advance to the release snapshot, and rolling installations retain their configured mirror.

## Restore an earlier system

```sh
sudo pkg history
sudo pkg rollback --to REVISION
sudo reboot
```

Rollback across complete system versions uses a verified trial boot. Desktop settings have their own Malm history and are not automatically rolled back with system software.

If you restore the original software from older installation media, its original `/usr/bin/pkg` predates the upgraded state format. Use the retained portable updater in that recovery environment:

```sh
sudo /herda/state/distribution-release/bootstrap/pkg status
sudo /herda/state/distribution-release/bootstrap/pkg update
```

Complete the system upgrade before editing boot parameters from that older recovery environment. Existing parameters remain preserved.

## Desktop reconciliation

After a confirmed update, Herda reconciles distribution-managed Smia sources using Malm's active source. It keeps the selected profile and local overrides. Custom Git checkouts and conflicting local changes remain under their owner's control.

The user service is `herda-smia-reconcile.service`; its result is recorded at `~/.local/state/herda/smia-update/status.json`.

## Release verification

Each immutable release contains a signed canonical manifest, detached signature envelope, public certificate, exact source revisions, and SHA-256 addressed chunks. Chunks are transport files consumed by `pkg`, not installation images.

This release's manifest SHA-256 is `b2b7e474c81d3f86611904901e8d7e10efbe72dfa2fcc0fc65033adf541838f0`.

Publication requires actual installed-system boot checks for bootstrap migration, snapshot and rolling upgrades, interrupted-trial fallback, complete rollback, and configuration preservation. The first release also checks cancellation followed by retry, a native boot-parameter change, and managed Smia reconciliation.
