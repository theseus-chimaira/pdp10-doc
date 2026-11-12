# DAIMOS KCORE v1523: FILE and cwd ownership before unmount

Date: 2026-11-12. Following v1522's non-root DTFS mounted-identity
guard, this increment closes a separate **source-linked native
KCORE mount-lifetime defect**. It does not constitute a fully
accepted physical removable-volume recovery or P0-A fatal
device-loss classifier. VERSION remains 0.5.

## Found and corrected

The earlier `vfs_unmount` checked PIPE and process executable
BACKSTORE owners, but did not check ordinary FILE/cwd references
from another process, including one whose user VM had been
swapped. A recycled six-bit mount ID could therefore allow a
still-open vnode to refer to a new, unrelated volume.
Furthermore, one owner check derived the mount ID from AC2
after a different checker had already clobbered that register.

Fixed KCORE adds `file_mount_busy_v1523`. It uses the existing
three-word process descriptors and their owned stable u-area
base pointers, scanning cwd at 025 and all sixteen two-word
FILE descriptors beginning at 026. It compares the actual full
six-bit public mount ID in the vnode. No separate owner
registry, permanent bitmap, reference counter, process field
or allocated MM data is added; swapped processes are covered.

`vfs_unmount` now preserves the original mount ID across PIPE,
SWAP and FILE checks. It rejects busy owners before potentially
sleeping provider SYNC/PREPARE and checks again immediately
before namespace/slot retirement, preventing a new owner
created during SYNC from being silently stranded. The
original mount, FILE/cwd and storage ownership survive a
refusal; healthy unmount still retires the same slot once.

## Executed evidence

Native PDP-6 `daimos-testkit/tests/validation/
daimos-file-mount-busy-v1523.sh` source-links the real VFS
and FILE assembly and tests current open FILE, remote
swapped-process FD15, remote cwd, stale unowned u-area,
physical SYNC refusal, new owner during SYNC and safe
post-close retirement with independent root preservation.
Seven executed unsafe source mutations are rejected,
including both old public-mount-ID clobber paths.

The existing v1514 non-root failed-media unmount/return
native regression and its three mutations PASS. The
isolated 256K KCC tape-root full image assembles and
links. Matched KCORE old/new links using the same remaining
objects and independent fold plans measure **+54 fixed
resident instruction words, zero new permanent data words**.
The strict 32K size gate remains on HOLD.

## Physical-guest boundary

An attempted full booted DTC3 mounted-cwd/unmount fixture
reaches a functional DSH login. In initial shortened
diagnostic attempts, target FSCK.DTFS or MKFS.DTFS on the
disposable DTC3 image did not finish before the fixture
timeout; those attempts cannot prove that physical
mount/unmount works. Full guest acceptance still requires
a real successful secondary DTFS mount, cwd refusal,
post-close retirement, plus actual error/return/replaced
media under live open FILE and dirty write lifetimes.
No physical PASS is inferred from the native tests.

P0-A authoritative Type-551 root/BACKSTORE loss detection,
and complete P0-B physical media/dirty-write recovery,
remain open; later P1 mixed storage, FSINFO and
uncompressed PURE likewise remain open.

Primary source:
`DAIMOS/design/kcore-file-mount-lifetime-v1523.md`.
