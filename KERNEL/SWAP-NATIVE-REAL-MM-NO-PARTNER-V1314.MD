# DAIMOS v1314: Native PDP-6 real-MM no-partner pressure validation

Date: 2026-11-07. Production DAIMOS baseline `9475f50a`.
This is an *additional* native integration test for the existing
slot-0 swap-in service. No production MM, scheduler, swap, KCORE or
MRES source changes were made. The F12/F13 requester-only OOM
acceptance remains 64/64 for the strict 256K release profile.

## Test design and what is actually executed

`daimos-testkit/tests/validation/daimos-swap-real-mm-no-partner-v1313.sh`
extracts directly from the current production assembly:

* `mm_find_fit`, `mm_extent_insert`, `mm_alloc_aligned_noreclaim`;
* `mm_arena_fits_v1260`, `mm_compact`, `mm_alloc_aligned`, `mm_free`;
* `proc_swap_mres_service_v481`, including candidate scanning,
  retryable-pressure handling, backing retention and cooldown.

It executes these leaves on the PDP-6 emulator as actual 36-bit
instructions. A single 010000-octal-word physical arena beginning
at word 02000 is fully occupied by a correctly typed kernel-dynamic
extent owned by slot 3. A swapped RUN process in slot 2 needs a
02000-word process extent. The real MM allocator produces typed
`MM_ERR_NOMEM` (-1) after its bounded two filesystem-reclaim and one
process-reclaim attempts. The deliberately non-freeing reclaim
stubs cannot dislodge the occupied extent. The three candidate
admissions are rejected. The unchanged slot-0 service must then
preserve the swap record, avoid TERM and publish the 0100-octal
pressure cooldown without trying to exchange a process.

The test separately invokes the production `mm_free` to release
the occupied physical extent with its exact type/owner. On a new
swap-service request, the same production allocator succeeds and
owns a 02000-word process extent on behalf of slot 2. The slot-0
service grants the one-shot READY_ONCE scheduler hint; the test
checks no incorrect event or destructive exchange occurred.

## Results

The positive native PDP-6 test PASSes. Four executable assembler
mutations are each detected as failures: collapsing typed NOMEM,
skipping filesystem reclaim, skipping process reclaim and losing
the pressure cooldown. These are real machine execution tests,
not pattern-only source guards. Neighboring v756 typed-result,
v1300 no-partner source guard, v1301 native service, v1111 MM
typed exhaustion and v1260 arena feasibility tests all PASS.
The native v1314 testkit source is committed as `d40a1158`.

The disk read/restore publication and the exchange-admission
result are intentionally test-controlled stubs. Consequently this
**does not** demonstrate full-system disk read/rollback or liveness
under a booted 256K multi-process workload, nor does it prove the
PI6 clock's cooldown aging with independent runnable owners.
The earlier v1308 full 24K-BSS guest completed ordinary 51-out/
51-in swapping, but never entered this MM resource failure path.
Production no-partner liveness remains OPEN. There is no evidence
requiring the rejected 16-word v1208/v1211 reclaim fallback.

## Retained evidence and next gate

Native test sources, original production assembly selection, positive
map/report, four negative mutants and logs, input hashes and SHA256
manifest are retained in
`$TMPDIR/daimos-swap-real-mm-no-partner-v1314-evidence-r1.tar.gz`.
SHA256: `c30cccf491176a4488ffcf8933dff0b0485ce85dc198c3d10db60e06c884ed92`.

The remaining acceptance must be a *booted* 256K PDP-6 case with
genuine allocated extents that prevent swap-in, no eligible equal-
size exchange partner, independent code still making progress,
then a genuine freeing event, bounded cooldown expiration, actual
backstore restore and complete release/rollback accounting. All
claims must be backed by observed MM and swap counters rather
than synthetic service status or the mere successful termination
of an ordinary-swapping guest.
