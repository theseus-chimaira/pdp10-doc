# DAIMOS TSFS boot-root-only acceptance contract (V165)

Date: 2026-10-22

TSFS is only usable as the filesystem selected at boot. A later
secondary TSFS mount is explicitly unsupported, even if historical
experiments found a path that appeared to permit it. TSFS images are
created entirely in userspace; media writes use `dd`. Neither the
test suite nor documentation should demand that a D6FS-root kernel
load a dormant TSFS MRES merely to mount an additional tape.

## Obsolete tests removed

- `daimos-tsfs-mount-v1.sh`, which invoked `MOUNT.TSFS`
  after a D6FS-root boot
- `daimos-tsfs-dir-v2.sh`, which relied on that same secondary mount
  for directory/stat checks
- `daimos-tsfs-scan-v1.sh` and its v1 C fixture, which were
  superseded by v2 and still exercised the legacy mount handoff

Their targets have been removed from the Makefile, command coverage
and both active group-027 regression matrix definitions. The v2 scan
fixture retains media scanning, member ordering, eight-member
handling and ninth-member rejection; the obsolete mount-handoff
assertions were dropped. The active v2 scan test passes.

The DAIMOS userspace builder also no longer builds or packages
`MOUNT.TSFS`; its source and standalone SIXMD manual were removed.
The user-level mount syscall stub, its extension dispatch entry and
the userspace mount-handoff builder are also removed, together with
their private metadata validation code. The extension selector now
uses the existing unsupported-operation handler.

KINIT's boot-root path must remain available: `tsfs_boot_mount_root`
invokes `FS_MRES_OP_MOUNT_UNIT` against the resident TSFS provider,
which still calls `tsfs_mount_set`. That operation is **not**
the obsolete `MOUNT.TSFS` user command and was deliberately retained.
The shortened userspace TSFS scanner continues to pass its eight-member
and ninth-member rejection tests. C99 scanner compilation, libc syscall
assembly, manual validation and a production-image dependency preflight
have also passed. A fresh tape-root full boot remains to be retested.

## Remaining group 027

Maintain and validate DECtape block/stream/reverse I/O, TSFS
image creation, scanner correctness, eight-member boot selection,
multi-extent reading and compressed extents. Retest those paths
under KCC and GCC. No secondary TSFS mount acceptance is required.
