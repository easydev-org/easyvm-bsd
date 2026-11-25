# easyvm-bsd

Walkthroughs and CI for **BSD** guests, driven by
[`easydev-org/easyvm`](https://github.com/easydev-org/easyvm).

## What lives here

One directory per distribution, each with the `.walk` files its stages run:

| directory | installer | disk | desktop |
|---|---|---|---|
| `freebsd/steps/` | `bsdinstall` (TUI) | UFS | Xfce from `pkg` |
| `ghostbsd/steps/` | GBI (GTK, clicked) | **ZFS** - GBI offers nothing else | MATE, its default |
| `openbsd/steps/` | `install` (TUI) | FFS, sets from the disc | cwm, from `xbase` |
| `netbsd/steps/` | `sysinst` (TUI) | FFS, sets from the disc | Xfce from `pkgin` |

BSD is a different operating system and the SAME machine: all of these boot the way every Linux here
does - UEFI firmware for FreeBSD, GhostBSD and OpenBSD, BIOS for NetBSD's `sysinst` - a virtio disk,
std VGA at the resolution the coordinates were measured at.

### Why GhostBSD is on ZFS

GBI has no UFS option. `ghostbsd/gbi`'s `installType.py` defaults to `'zfs'`, the UFS full-disk radio
is commented out of the source, and `partition.py` appends only `ZFS`, `SWAP` and `UEFI` to the
filesystem combo. The one UFS path left is a manual partition editor, which is not what an unattended
walk should drive. Everything else here is UFS/FFS.

### Why NetBSD's install has no screen gates

NetBSD's `sysinst` draws on a 720x400 VGA text mode with a small low-contrast font, and tesseract does
not read it: MEASURED, probing the menu region of a live installer returned

```
none of "zzzznotpresent" in "j- oie alll i estes Meal AA eee Lelalaentaitor AA ..."
```

The other three installers draw at 640x480 or larger and ARE read. So `netbsd/steps/install.walk`
carries `settle:` instead of a gate on every step, with the sequence measured key by key.

```sh
gh workflow run freebsd.yml -f disks_repository=easydev-org/easyvm-bsd -f from_tag=bsd-freebsd-ssh
```

## What lives elsewhere, on purpose

The **engine** and the **CI machinery** are both in `easydev-org/easyvm`; this workflow calls
`easydev-org/easyvm/.github/workflows/stage.yml@main` for the second.

Disks pull from and publish to this repository's own ghcr package, `easydev-org/easyvm-bsd`.
