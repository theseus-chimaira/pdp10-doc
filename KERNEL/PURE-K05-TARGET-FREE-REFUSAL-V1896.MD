# PDP-6 K05 v1896: genuine native PROCESS-target rollback free refusal

Date: 2026-11-23. **Scoped original 48K PDP-6 physical PASS**.
K05-K09 remain OPEN. RAW PURE and mode3 remain OFF. No change to
production fixed KCORE, optional PURE MRES, MM code or RAM footprint.

## Contract

Original `proc_pure_ram_restore_v1391` retains one authoritative
PACKED RAW suffix until it has reconstructed a complete PROCESS
target and proven a safe owner transition. Original v1839's
post-error MM scan distinguishes source free committed, exact two
unpinned owners allowing target rollback, and an uncertain owner
set requiring quarantine. Even when exact two owners existed at
the start of cleanup, a **new pin may prevent target `mm_free`**.
If that original target release fails, the kernel must preserve
both real original MM reservations and slot3 TRANSITION, without
publishing a speculative APR or claiming that target was freed.

Earlier physical acceptance isolated v1893 source-free completed
with a false failure reply, v1894 genuine PACKED source-free
refusal followed by source-pin release and successful target
rollback, and v1895 a permanently pinned source that blocked
even choosing target cleanup. v1896 now **reaches and executes
the original target cleanup call** and forces the original native
`mm_free(PROCESS)` itself to refuse.

## Original fixed-KCORE instructions and full machine

The same source-built original v1855 48K PDP-6 production fixed
KCORE and removable PURE MRES are used byte-identically. The
test-only v1892 parent INIT retains original five-child RUN/BRK/
PIPE pressure, writable-data sentinel checks and compressed
secondary DTC3 executables while allowing already-resident parent
user `HB` continuation instead of waiting for a deliberately
quarantined child. No original kernel, MM, loader, media or user
child code changes.

The original installed PURE base `032417` octal comes from the
actual running KCORE reclaim JRST, corroborated by compiled
original `proc_pure_full_v1611.dobj` and the linked MRES image.
Its original FULL `mm_free` relocations are `.text+000674`
for PACKED source and `.text+000755` for temporary PROCESS
target, relative to original function `.text+000504`.
Actual verified physical instructions (all octal):

* original bounded FULL loader `036202`;
* original PACKED-source `PUSHJ 17,mm_free` `035310`, return
  `035311`;
* original `fail_target` cleanup `PUSHJ 17,mm_free` **`035371`**,
  return **`035372`**;
* original FULL service return `033067`;
* original MM metadata for source `024703`, target `024705`.

The original pinned FULL has EXACT slot3 RAW source PACKED
`111200..120000` pin1 and separately complete PROCESS target
`120000..130000` pin1, plus independent already-resident parent
slot1 PROCESS `130000..140000` pin0; original MM count 18.
The original loader completes its full data reconstruction and
original successful unpins leave both slot3 owner pins zero.
As in v1894, a one-time test-only external source pin makes
original native `mm_free(PACKED)` genuinely return -5 at `035311`
without deleting either extent. The test-only source pin is
then released; the original v1839 scan confirms exactly TWO
unpinned original owners, and the C transaction chooses rollback.
The actual original `fail_target` target-free call at `035371`
is reached with source pin0, target pin0, MM count 18 and all
unrelated owners and selected TSFS ROOT unchanged.

Only then the private simulator raises the original target
metadata pin by ONE; this is a test-only physical owner mutation:

    024705: 000001000003 -> 000011000003

The ORIGINAL unmodified fixed-KCORE `mm_free(PROCESS)` runs and
refuses the pinned target, returning actual `MM_ERR_BUSY=-5`
as physical AC1 `777777777773` at original `035372`. No target
extent is deleted. The test-only external target pin is NOT
released. At original service return `033067`, the original FULL
reports -5, keeps BOTH MM owners (source pin0, target pin1),
and leaves original slot3 TRANSITION and VMbase zero. Its original
process meta at `044723` is `046031600401` (TRANSITION), and
VM word `044724=010000000000`. There is no falsely published
runnable APR, and the authoritative PACKED RAW suffix is safe.

The original full MM extent table has 18 descriptors at every
stage. All unrelated MM owner/type/size/base/pin records and
selected ROOT `070101000000` octal remain unchanged through
the failed target cleanup and two later user-mode heartbeat
stops. The real parent slot1 remains unquarantined, SRUN with
resident APR `130000..140000` pin0, executing from original
`proc_current_slot=1` at BOTH new `HB` events. Real fixed-KCORE
PCLK advances from `015401` to `015564` octal between the HBs.

## Physical MM checkpoints

| Original checkpoint | MM extents | PACKED source pin | PROCESS target pin |
|---|---:|---:|---:|
| Original pinned FULL | 18 | 1 | 1 |
| Original source free invocation | 18 | 0 | 0 |
| Genuine source `mm_free` -5 | 18 | 1 | 0 |
| External source pin released | 18 | 0 | 0 |
| Original target cleanup free invocation | 18 | 0 | 0 |
| Genuine target `mm_free` -5 | 18 | 0 | 1 |
| Original FULL reports failed rollback | 18 | 0 | 1 |
| First and second new original parent HB | 18 | 0 | 1 |

The independent read-only acceptance validates original production
binary hashes, original DOBJ-to-linked-image native `mm_free`
locations, BOTH real source/target -5 replies, owner geometry,
unchanged ROOT/MM, original TRANSITION/VM0 and independent parent
dispatch/PROCESS APR/PCLK. **THIRTEEN separate executed unsafe
physical transcript mutations** are rejected at intended checks:
source free error lost, original source pin not released, target
already pinned at cleanup, missing target external pin, target
free falsely succeeded, target/source owner corruption, false
RESIDENT/VM publication, falsely successful original FULL return,
wrong parent current process, nonadvancing PCLK or missing HB.

Accepted normalized original PDP-6 physical SHA256:
`b408a84b27b39b351e56196356fb4abaac2b60c64175abe4bb74d38382ab7601`.
Original fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Original optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Real physical evidence:
`$TMPDIR/daimos-k05-target-free-refusal-v1896-real/`.
Detailed DAIMOS design/state:
`design/kcore-k05-physical-target-free-refusal-v1896.md`.
Testkit source and runnable audit:
`tests/validation/pure-k05-target-free-refusal-v1896/`.
Targets `make test-pure-k05-target-free-refusal-physical-v1896`
and `make test-pure-k05-target-free-refusal-audit-v1896`.

## Scope

This closes genuine original rollback `mm_free(PROCESS)` refusal
under a permanently held target pin, without unsafe deletion or
APR publication, and independent resident parent progress. The
test controller deliberately exits after two new parent HBs;
it does NOT demonstrate release of the target pin, quarantined
owner terminal cleanup, child EXIT/WAIT/HALT, successful target
free with lost result, foreign owner, permanent allocator/unpin
uncertainty, arbitrary PI6 timings, 64K/KA10 or K05-K09 release.
