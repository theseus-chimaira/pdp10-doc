# DAIMOS KCORE v1520: native DTFS volume identity groundwork

Date: 2026-11-12. Completed and committed feature stage on
native DTFS removable-volume identity, not a claim of automatic
replacement-media recovery or of Type-551 physical-loss detection.

## New optional on-media identity format

Official native DTFS V1 reserves physical DECtape block 0. Older
native formatters leave that 128-word block entirely zero.
The directory at block `0144` has no spare identity field and
continues to carry the unchanged allocation map, file slots and
native magic. v1520 therefore places an optional four-word
identity record in the previously reserved block 0:

| RBN0 word | 36-bit content |
| --- | --- |
| 0 | SIXBIT `DTVOL1` = octal `0446466575421` |
| 1 | Nonzero random high 36-bit volume identifier |
| 2 | Nonzero random low 36-bit volume identifier |
| 3 | Complement of XOR of words 0,1,2, masked to 36 bits |
| 4..127 | Required zero padding |

An **entirely zero** reserved block 0 continues to mean legacy
native DTFS and is accepted for initial mounting. No unique
identity can be inferred for such media. A nonempty malformed
record is refused, never silently downgraded to legacy.
This checksum detects accidental corruption; it is not a security
MAC. A 72-bit random equality token gives probabilistic identity,
not a guarantee against deliberately cloned tape images.

Host `tools/host/mkdtfs.c` draws 9 bytes from `/dev/urandom`
and formats stamped native media. It refuses to create an
identity without trusted randomness and retains exclusive file
creation; no existing source image is overwritten.

Target `MKFS.DTFS` has no suitable trusted random source, so it
deliberately formats **legacy** native media instead of inventing
a clock/PID-derived token. Crucially, it clears all of reserved
RBN0 before writing a new directory. A failed RBN0 reset aborts
without writing the directory, preventing the new filesystem
from inheriting a stale stamp belonging to an older filesystem.
Target and host paths reuse existing 128-word workspaces.

## Kernel and userspace validation

The real userspace DTFSPROBE, after native directory validation,
physically reads the identity RBN0 using its existing probe
buffer and checks all stamp fields and zero padding or the
entirely zero legacy block. Missing/corrupt identity fails.

The optional native PDP-6 `dtfs_mount_unit` independently checks
the directory and the physical RBN0 before VFS mount admission,
bypassing shared BCACHE. Valid ID halves survive the VFS
publication call on the owner's temporary stack and are then
stored in the already allocated MM mount slab's two SHAPE and
EXTENT words for that DTFS slot. TSFS and DTFS mounts cannot
occupy the same slot, so no additional per-mount data words
are needed. Legacy native media store zero/zero, meaning ID
unavailable. The root TSFS crash-only policy is unchanged.

## Executed tests and measured footprint

Testkit `daimos-dtfs-volume-id-v1520.sh` compiles the actual
host image producer, source-links the real C89-compatible
userspace probe and target `MKFS.DTFS` command, then executes
the production native PDP-6 mount/BCACHE source. Cases cover
two independently stamped full-size images with different IDs;
unchanged native directory; legacy zero RBN0; invalid checksum,
ID half, padding, partial record, wrong directory and physical
I/O failure; target MKFS clear-before-directory ordering and
abort on failed clear; and retention of both stamped ID halves
in their correct dynamic mount-slab words.

Five executed PDP-6 unsafe source mutations are rejected:
ignore checksum, ignore nonzero padding, accept arbitrary
nonempty identity, drop high-half storage, and drop low-half
storage. All v1520 cases PASS. Existing v1519 native format
mount/6 negative mutants, v1518 DTFS cache recovery, v1490
BCACHE ABI/MM refusal, and DTFS chain/create/unlink/rename/
dirfield and TSFS cross-member suites PASS.

Both isolated **256K KCC** and **32K GCC** tape-root KINIT with
DTFS/TSFS MRES assemble and LINK. KCC also compiles the target
`MKFS.DTFS` and DTFSPROBE objects. The strict **32K fixed-KCORE
size gate still FAILS** at `023241` octal against `<020000`.
This is not 32K memory-size acceptance.

Controlled old/new MRES folding against the same fixed-KCORE map
and unchanged other objects reports **+46 optional DTFS/TSFS
MRES code words, +0 new permanent data words, +0 fixed KCORE
words**. Reusing existing two per-slot slab words means zero
dynamic mount-slab RAM growth. The existing native directory
and VFS public ABI are unchanged; the reserved physical block 0
now has an optional stamped V1 extension.

Testkit commit `4e21b76`; DAIMOS commit `65ed0b5b`.
Full technical record: `DAIMOS/design/kcore-dtfs-volume-id-v1520.md`;
testkit: `tests/validation/daimos-dtfs-volume-id-v1520.md`.

## Remains OPEN

v1520 DOES NOT yet compare the saved identity before reusing an
already mounted volume after an error or after a silent physical
tape exchange. A cached hit may still escape detection. The old
FILE/vnode/dirty data/mount-slot lifecycle has not been validated
for live replacement, and a zero-ID legacy tape cannot be
authenticated as the same returning medium. Do not claim online
recovery based solely on header presence or a fresh mount read.

Next non-root P1 is checked mounted-volume identity transition,
cache invalidation and old-handle refusal on a mismatched stamp.
P0 Type-551 authoritative physical loss detection and TSFS root
fatal HALT caller remain OPEN. Mixed devices, swapless PURE/MM/APR
and dynamic FSINFO feature work remain separate, with memory
optimization after feature completion. VERSION 0.5/32K HOLD.
