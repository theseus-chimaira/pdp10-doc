# DAIMOS KCORE v1518: fail-closed DTFS cache after physical I/O error

Date: 2026-11-12. Optional DTFS/TSFS tape-MRES correctness feature,
not a claim of reliable physical tape-removal detection or complete
replacement-volume recovery.

## Accepted source behavior

Before v1518, the shared DTFS directory cache could retain
`dtfs_cache_mount` after a negative physical DTC read. Subsequent
directory operations on the same mount could skip physical I/O,
even though a recent tape operation had failed. The two-entry clean
BCACHE also retained clean entries from before the observed failure.
For a failed write, BCACHE was invalidated *before* hardware I/O,
but the in-memory directory still appeared current after a possible
partial/failed metadata write.

The optional `dtfs_runtime.s` read veneer now preserves the negative
hardware result but clears `dtfs_cache_mount` and both clean BCACHE
tags after a **failed physical read**. The write veneer continues to
invalidate both clean tags before transferring the original DTC
unit/block/buffer arguments, then checks the actual physical write
result and clears the directory mount identity only on error.
Success paths keep their existing semantics. No forced VFS unmount,
MM reclaim, extra process/device state or data words are introduced.

The TSFS root also uses the common DTC read veneer. It benefits from
fail-closed cache invalidation after transient read failure without
changing the v1515 confirmed-root-media-loss fatal policy.

## Validation and measured cost

Testkit v1518 `daimos-dtfs-io-error-cache-v1518.{s,sh}` directly
links the real PDP-6 DTFS physical veneers, native `dtfs_load`,
and complete KCORE BCACHE assembler. The boot-patched raw DTC
I/O slots alone are directed to deterministic hardware callbacks.
The native oracle PASSES: initial clean cached directory,
observed physical read failure, rejected stale directory/BCACHE,
another real failing directory request, valid read after the *same*
medium returns, failed write clearing the directory identity,
and successful write retaining its known-valid directory identity.

Four **executed** source mutants are rejected: stale directory
on read error, stale BCACHE on read error, stale directory on
write error, and skipping write-side cache invalidation. The old
v1490 BCACHE source-linked MM-refusal/raw DTC ABI test and existing
DTFS chain-walk/create/unlink/rename/directory-field and TSFS
cross-member read regressions all PASS.

Both isolated full 256K KCC and low-memory 32K GCC tape-root
KINIT with DTFS/TSFS MRES assemble and LINK. Strict fixed-KCORE
32K size gate still **FAILS** independently at `023241` octal
against strict `<020000` octal. It must not be called accepted.

Controlled old/new optional DTFS/TSFS MRES linking, with identical
other objects and independent folding plans, measures exactly
**+6 optional MRES instructions, +0 permanent data words, +0
fixed KCORE words**. Public ABI and on-media layout are unchanged.

Testkit `71027aa`; DAIMOS `7b924159`. Full source/size/test detail:
`DAIMOS/design/kcore-dtfs-io-error-cache-v1518.md` and
`daimos-testkit/tests/validation/daimos-dtfs-io-error-cache-v1518.md`.

## Explicitly outstanding boundaries

This feature invalidates previously trusted clean data **only once
a physical operation has returned error**. A completely cached
volume can still serve hits during a silent disconnect, because
the Type-551 provides no currently accepted authoritative removal
notification. Reinserted **different media** must not automatically
inherit prior open FILE, vnode, dirty data or mount identity merely
because it contains a parseable directory. Checked unique volume
identity, deliberate revalidation/remount and full physical fault/
recovery tests remain OPEN for non-root DTFS.

P0 root-set DECtape loss detection distinct from ordinary Type-136
error/timeout and physical v1515 fatal-stop dispatch also remain OPEN.
Mixed storage providers, F1 swapless PURE/MM/APR and F3 FSINFO
dynamic enumeration are still KCORE feature gates; compression and
memory optimization follow feature completion.

VERSION 0.5 and strict 32K HOLD remain; MRES unloading post-1.0.
