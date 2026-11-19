# PDP-6 v1656: original source-linked DTFS V1 integration reaccepted

Date: 2026-11-17. No production kernel, MRES, VFS, MM or
resident RAM change. Original PDP-6 source-linked v1518
raw DTC read/write, real BCACHE, native directory loader
and v1522 mounted volume ID/revalidation integration
now both PASS with the current original kernel source.

## Fixture problem and correction

v1518's original test predated the v1561 installed
physical-unit first-write intent gate and left VFS
mount owner tables uninitialized. Its test now links
the unmodified v1561 assembly gate, with an actual
valid V1 secondary DTFS DTC3 mount2 and VFS limit2.
V1 physical write admission must traverse the original
gate. A fixture-only V2 first-write operation fails
closed and must never run in these V1 cases.
The actual pre-write BCACHE invalidate source
mutation was updated; four independently executed
unsafe mutations now fail their original native
PDP-6 oracle.

v1522's old source section ended at a deleted
historical comment. The testkit now extracts
the current exact dtfs_space function to the v1575
discard section, preserving its actual FSINFO
logic. It also links the real v1561 V1 mounted
source owner, actual V1 parser, V1/V2 recovery,
raw DTC wrappers and complete original BCACHE.
Eight unsafe mutations execute and are rejected:
read/write errors, both 36-bit volume ID halves,
zero/legacy return, WRITE_UNCERTAIN, retaining
saved mount slot and workspace BCACHE invalidation.

## Native current-source evidence

The versioned testkit runner executes original
v1518 and v1522 integration, followed by v1655's
82-case V1/V2/physical DIRTY/intent/MM owner matrix.
All PASS. **51 distinct compiled/executed unsafe
source mutations are rejected.** No mutation or
test-only provider stub is linked in production.

Testkit branch:
test-kcore-dtfs-integration-v1656-20261117,
commit a507c17.
Runner:
tests/validation/dtfs-integration-v1656/run-v1656.sh.
Full DAIMOS report:
design/kcore-dtfs-original-integration-v1656.md.

Versioned native-original evidence:
$TMPDIR/daimos-kcore-v1656-evidence-20261117.tar.gz
SHA256 7393d83105e281b66bf24852dfbb08e864b9b198978d64002a26f7b0896b1ec5.

The v1655 stale original v1518/v1522 source-linked
native integration gate is now CLOSED. This does not
prove a new physical 48K independent Type-551
device race or mechanical reel-retention protection.
The accepted v1575 operator-trusted removable writable
media contract stays explicit.

Further PDP-6 KCORE release work: complete actual
whole-machine physical peak including removable MRES,
BCACHE and provider scratch; permanently uncertain
MM owner fail-closed policy; measured optional suffix
compression YES/NO; final matched-current-source
native/physical kernel freeze. Strict 32K footprint/
boot acceptance is explicitly DEFERRED.
