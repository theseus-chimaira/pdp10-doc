# v1462: branch integration and retained release codec

Date: 2026-11-10.

DAIMOS, daimos-testkit, and pdp10-doc feature histories were merged into
dedicated DevSpace2 integration worktrees, starting from their latest
accepted `main` commits. Native toolchain/compiler/assembler/linker
branches, historical F10/E07 checks and the complete v1420-v1461 codec
experiments were included. Two divergent codec integration histories
(v1459 KCORE/Stage-1 and v1461 TSFS media/entry fixes) were merged rather
than silently dropping one. Every fetched `origin/*` branch tip is an
ancestor of the resulting integration commit.

## Production cutover decision

The alternative grouped/block D6 codec remains **deferred** until the
other memory features are complete. Newly versioned native PDP-6 codec
sources, host encoders, standalone tests and experimental documentation
remain present in their original versioned files and Git history. The
production assembly, header, boot/Stage-1, KCORE, swap/VFS, host tool
and userland input selection, however, is restored from the pre-codec
integration commit. The original `d6lz.h` and `d6lz.c` were explicitly
restored after Git's rename detection initially omitted them.

The historical v1458/v1461 branch reports provide valuable acceptance
evidence for those isolated experimental builds. They are **not** evidence
that the new codec has been activated or acceptance-tested in the newly
merged default system. Production remains on the existing D6LZ36 wire
format, retaining its previous swap and filesystem compatibility.

## Integrated release checks

- Exact lowmem disk-root GCC KCORE: **9687 words**, unchanged.
- Movable SWAP MRES: **1762 words**, unchanged.
- Combined linked KCORE+MRES inventory: **16367 words**, unchanged.
- Strict single-root 16K permanent address placement: **31 words over**,
  a pre-existing unmet acceptance gate rather than a new regression.
- Production-linked native PDP-6 BACKSTORE owner, swap-in refused-release
  and retry, exchange accounting, cancellation and metadata checks PASS.
- The previously accepted strict 256K F07 guest test remains relevant to
  unchanged release code paths, but its v1406 result is not represented as
  a fresh full-system test of all optional merged components.

Remaining correctness work includes the slot-0 event cleanup after a
refused BACKSTORE free, privileged MEMFS XIP RUN/EXEC, swapless PURE RAM
reclamation, and the 32K permanent placement gate. The block-codec
cutover must have its own eventual explicit decision and validation.
