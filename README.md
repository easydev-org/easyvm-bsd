# easyvm-bsd

Walkthroughs and CI for **BSD** guests, driven by
[`easydev-org/easyvm`](https://github.com/easydev-org/easyvm).

## What lives here

`freebsd/steps/` - the `.walk` files for `install`, `ssh` and `vm-configure`, plus the reference images
they compare against.

BSD is a different operating system and the SAME machine: GhostBSD and FreeBSD boot exactly the way
every Linux here does - UEFI firmware, a virtio disk, std VGA at the resolution the coordinates were
measured at. The name appears in logs and in a file's `os:`, and nothing else about the guest changes.

```sh
gh workflow run freebsd.yml -f disks_repository=easydev-org/easyvm-bsd -f from_tag=bsd-freebsd-ssh
```

## What lives elsewhere, on purpose

The **engine** and the **CI machinery** are both in `easydev-org/easyvm`; this workflow calls
`easydev-org/easyvm/.github/workflows/stage.yml@main` for the second.

Disks pull from and publish to this repository's own ghcr package, `easydev-org/easyvm-bsd`.
