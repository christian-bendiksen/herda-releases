# Herda system updates

Signed development releases update an installed Herda system without reinstalling it. Desktop configuration remains managed separately through Malm and Smia.

The current release is [`dev-20260913.7`](https://github.com/christian-bendiksen/herda-releases/releases/tag/dev-20260913.7). Use current installation media with native boot support. Older pre-release installations are no longer supported: install from the new USB image. The portable updater and two-stage migration have been removed.

## Update the system

Current media includes the updater and public publisher certificate. After the first healthy installed boot, `herda-release-enroll.timer` enrolls the publisher automatically. It never downloads or installs updates automatically.

```sh
sudo pkg releases status
sudo pkg update --check
sudo pkg update
sudo reboot
```

The first update and subsequent updates use the same flow: review, download, prepare the complete system, then one trial reboot. The previous healthy system remains the fallback until the new system passes boot confirmation. There is no preliminary helper trial.

Use `sudo pkg update --resume` after interrupted preparation. Use `sudo pkg update --cancel` to cancel an unobserved trial. Downloads are verified, resumable, and protected against sequence rollback; cancellation preserves replay protection and reusable cache data.

`sudo pkg update --packages-only` selects ordinary package updates. Complete Herda releases replace the distribution runtime and boot substrate together; snapshot installations advance to the release snapshot, and rolling installations retain their configured mirror. Application declarations, account identity, home files, and local configuration stay on the installation.

## Restore an earlier system

```sh
sudo pkg history
sudo pkg rollback --to REVISION
sudo reboot
```

Rollback across complete system versions uses a verified trial boot, including rollback to the original installation. That installation already has the current update protocol. No portable updater or persistent service override is required.

Desktop settings have their own Malm history and are not automatically rolled back with system software. After a confirmed update, `herda-smia-reconcile.service` reconciles distribution-managed Smia sources, preserving the selected profile and local overrides. Custom Git checkouts and conflicting local changes stay under their owner's control. Its result is recorded at `~/.local/state/herda/smia-update/status.json`.

## Release verification

The publisher fingerprint is `60ad738c0df1ee6d782811203ab084d08f0f0917`. Current media pins its public certificate. No private release-publisher key is distributed; each installation generates its own private authority for update boot artifacts. Development installations currently require Secure Boot disabled; package, release, and boot-artifact signatures are still checked.

Releases use signed metadata, five artifact roles (native payload, substrate, kernel, desktop runtime, and Smia), and immutable GitHub assets. Updater API 2 removes the portable bootstrap artifact. Unsigned, expired, incompatible, or replayed releases are refused.

See [qualification](QUALIFICATION.md) for the exact release and test evidence, and [the source documentation](https://github.com/christian-bendiksen/herda/blob/main/docs/system-releases.md) for the complete workflow.

Known issue: qualification observed Btrfs/fs-verity kernel warnings despite successful complete image readbacks. See [the qualification limits](QUALIFICATION.md#known-issue-and-limits).
