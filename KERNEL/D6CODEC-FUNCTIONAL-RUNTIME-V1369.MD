# v1369: native grouped EXEC, PURE and TSFS integration tests

Date: 2026-11-08. Scope: test-only grouped DXR2 producer and actual
PDP-6 runtime functions linked with controlled VFS/VM providers.
Production compressed media and KCORE remain on D6LZ36.

The grouped runtime frontend from v1367 and the full-KCORE link
measurements from v1368 did not by themselves prove that real
EXEC/PURE/TSFS callers worked with grouped-format files. v1369
addresses those functional gaps before further size optimization.

## Executable production and metadata

`daimos-testkit/tests/validation/d6codec-runtime-exec-v1369`
contains a versioned grouped DXR2 writer that validates a real
uncompressed DXR1/DXR2 binary and preserves its entry, original
flags other than COMPRESSED, image length, BSS size, TX2 text count
and relocation bitmap. It verifies an independent grouped roundtrip
before publishing a new versioned file. It rejects precompressed
inputs and refuses overwriting the original or a legacy-named DXR.

The DXR compressed flag has **no codec identity bit**. D6LZ36 and
grouped DXR2 cannot coexist on production media under the current
format. Final conversion must replace the whole artifact set and
all consumers at once; no permanent dual-codec decoder is planned.

## Real EXEC and PURE native PDP-6 paths

The test uses unchanged production `exec_load_process` and an exact
extraction of `proc_swap_reload_pure` with the staged grouped VFS
adapter, plus deterministic filesystem and VM-allocation stubs.
Two real linked DXR test images PASS:

- An actual PDP-6 C89-compiled two-instruction executable is loaded
  and called in its decoded VM area; it returns octal 012345.
- A 390-word real tape MRES DXR is grouped to 379 compressed words
  and decoded through seven-word short VFS refills, with complete
  word-for-word output comparison.

Both ordinary and PURE-flagged EXEC pass. Evicted PURE text is
reloaded via the actual routine's trusted immutable backing path.
Malformed token and VFS provider failure cases return failure,
destroy the staged VM and refrain from publishing its entry or
attaching swap backing. Corrupt PURE reload is also rejected.
The PURE backing vnode unpacker is stubbed, so its filesystem and
mount lifetime are not demonstrated here.

## Real TSFS restart read/copy/free

The unchanged `tsfs_read_words` consumes a grouped restart extent
of 0400 octal decoded words encoded in 35 compressed words, below
the single-block 0176-word compressed bound. Native tests verify
correct read at logical offset 010, output copy and transient MM
allocation release. A malformed grouped control returns negative
and still frees the buffer without publishing invalid output.

Run:

    DAIMOS_REPO=/path/to/DAIMOS sh \
        tests/validation/d6codec-runtime-exec-v1369/run-v1369.sh

All v1369 tests PASS in native PDP-6 simulation. They are functional
tests of unchanged runtime code but NOT an end-to-end OS boot: real
VFS providers, VM allocation, PURE mount identity, filesystem media
and the complete KINIT/INIT/LOGIN lifecycle are not yet validated
with grouped streams. The v1368 +216/+256-word KCORE RAM overhead
is unchanged. The next gate is a coherent isolated grouped runtime
image and full cold boot, with nested provider stack measurements
and rollback, before considering any production format switch.

## v1370 subsequent full minimal-system cold boot

The complete grouped KCORE, all 27 re-linked MRES packages and an
actual DSK270/D6FS root now boot past KINIT into INIT, grouped
LOGIN and DSH. A grouped ECHO executable runs through the real
VFS/EXEC path and DSH exit returns to LOGIN. The test root has
15 nodes, not the full distribution. See
`KERNEL/D6CODEC-FULL-BOOT-V1370.MD`.
