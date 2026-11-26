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
| `midnightbsd/steps/` | `bsdinstall` (TUI) | UFS | its default |
| `netbsd/steps/` | `sysinst` (TUI) | FFS, sets from the disc | Xfce from `pkgin` |

BSD is a different operating system and the SAME machine: all of these boot the way every Linux here
does - UEFI firmware for FreeBSD, GhostBSD and OpenBSD, BIOS for NetBSD's `sysinst` and for MidnightBSD
(whose 4.0.9 keyboard does not answer under OVMF) - a virtio disk, std VGA at the resolution the
coordinates were measured at.

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

### The two endings that decide whether a BSD disk is usable

Both were found by mounting the published disk afterwards, not by reading the walk.

**NetBSD: `d: Reboot the computer`, not `r` at the exit prompt.** `x: Finished configuring` leads to
the completion notice, then the install system's main menu. Answering the exit prompt with `r`
reboots into the installer (the medium is still `bootindex=0`); the stage then kills qemu mid-write,
and the next `fsck` throws every name into `lost+found` - MEASURED, `2550 root wheel 182272 lost+found`
and `ls /mnt/netbsd -> No such file or directory`. `d` on the main menu syncs and unmounts first, after
which the same disk lists `bin boot dev etc kern lib data libexec netbsd proc rescue root sbin stand
tmp usr var` and boots to `Login:` on its own. The timezone list also has to be left with `x`
(`Press 'x' followed by RETURN to quit the timezone selection.`); a bare RETURN only picks an entry.

**MidnightBSD: the installer writes an fstab its own `mount(8)` rejects.** Line one of the generated
`/etc/fstab` is the header comment, and this release reads it as a malformed entry:

```
fstab: /etc/fstab:1: Inappropriate file type or format
```

so the root line after it is never applied and the boot stops at `mountroot>`. Rewriting the file
without the header makes the disk mount itself; `vfs.root.mountfrom` in `/boot/loader.conf` did NOT
take effect on 4.0.9 and is not what fixes it.

```sh
gh workflow run freebsd.yml -f disks_repository=easydev-org/easyvm-bsd -f from_tag=bsd-freebsd-ssh
```

## What lives elsewhere, on purpose

The **engine** and the **CI machinery** are both in `easydev-org/easyvm`; this workflow calls
`easydev-org/easyvm/.github/workflows/stage.yml@main` for the second.

Disks pull from and publish to this repository's own ghcr package, `easydev-org/easyvm-bsd`.
