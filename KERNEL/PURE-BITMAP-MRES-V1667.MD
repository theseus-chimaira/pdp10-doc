# v1667: test-only bitmap mode3 inside TSFS swapless PURE MRES

Date: 2026-11-18. Exact DAIMOS parent v1666:
`0d30d2c4eac0fd3466f0bdeeb6c69f35931c334e`.

## Architecture

The source-built KINIT root selection from v1665 chooses ONE
installed memory service: DISK MRES for a root or AUXST1 carrying
valid nonzero BACKSTORE, or PURE MRES for a selected TSFS root
without validated BACKSTORE. The terminal refactor in v1666
places backend-specific owner release behind a boot-patched
fixed JRST, avoiding dual MM compatibility code in permanent
KCORE. v1667 stages optional compressed mode3 terminal release
and FULL restoration **exclusively in PURE MRES**. The fixed
KCORE and the DISK MRES carry none of the bitmap codec/ownership
code and acquire no new process metadata.

The new build-only switch `DAIMOS_PBM_V1667=1` requires both
`DAIMOS_PURE_MRES_ENABLED=1` and TSFS-capable ROOT=tape or
ROOT=auto. Incompatible disk/drum-root or PURE-disabled builds
abort at Make configuration. An auto-root image can still carry
both MRES candidates in transient KINIT storage, but KINIT does
not install the PURE candidate unless actual TSFS/no-BACKSTORE
discovery authorizes it. Both opt-in switches default to zero.

The opt-in PURE terminal export index2 is
`proc_pure_term_v1667(slot, proc, backing)`, which reads the
existing two-bit storage map only under the process lifecycle
contract. A mode2 owner goes to the unchanged exact v1609 RAW
MM terminal assembly; a mode3 owner goes to the original v1661
compressed owner terminal. A mode1/RESIDENT owner is not freed
by this service. The common KCORE `proc_swap_detach` still owns
swap-record/u-area retirement after successful MM release.

The original v1612 PURE slot-0 RUN service conditionally checks
mode3 only when v1667 is enabled. It calls the v1664 bounded
FULL restoration with the existing executable-loader ABI and
the original MM allocator, whole-stream v1660 mask/literal
preflight and 30-word v1451 decode. After complete APR
reconstruction, the v1664 atomic MM finish releases compressed
source and destination transaction pins, and PI-excluded
nonfallible stores publish APR, mode0, backing and TRANSITION.
The v1667 retry helper reuses the exact owner verification from
v1664 to release failed TRANSITION only if one unpinned bitmap
source remains, no unexpected APR/owner exists, and immutable
backing identity remains valid. Uncertain rollback stays held.

## No production bitmap activation yet

The existing fixed `vm_space_destroy` STILL refuses mode3, and
normal PURE memory pressure only produces RAW mode2. The v1663
bitmap compressor remains unlinked from the real reclaim path.
Consequently no running guest can create a compressed process
through ordinary reclaim, and mode3 terminal/RUN/EXIT/WAIT/PI6
are not yet accepted as a production lifecycle. The opt-in mode3
service and terminal are exercised by original-MM source-linked
PDP-6 native fixtures, not by an enabled production scheduler.
Enabling the stage flag only adds code until actual compression
and safe lifecycle admission are proven. Do NOT enable it as a
RAM optimization on real installations at this stage.

## Native and linked-image evidence

The v1667 testkit runs 21 native C/ASM RAW/mode3 terminal cases
through actual fixed `proc_swap_detach`, original RAW v1609 ASM,
v1661 bitmap terminal and new PURE dispatcher. A separately
compiled unsafe mutant routing mode3 into RAW-only release is
rejected at PDP-6 runtime. The original v1664 source-linked
PDP-6 MM allocator/bitmap codec/FULL restore suite runs first;
16 additional real v1612 PURE service cases verify slot-0
successful 4,096-word FULL APR reconstruction (including zero
suffix), READY_ONCE and corrupted-owner/failed-allocation/
uncertain-rollback refusal. Exact Makefile policy and KCC/GCC
linker-map checks confirm that all compressed helper symbols
exist only in PURE MRES and fixed KCORE remains unchanged.
An independently compiled unsafe retry mutation bypassing the
exact failed-restore MM ownership proof is executed and rejected:
an uncertain pin never authorizes TRANSITION unlock.

| Installed component | v1666 baseline | v1667 stage | Extra words |
| --- | ---: | ---: | ---: |
| KCC FULL PURE MRES | 2307 | 3992 | 1685 |
| GCC lowmem PURE MRES | 1863 | 3186 | 1323 |
| KCC FULL KCORE low end (octal) | 024353 | 024353 | 0 |
| GCC lowmem KCORE low end (octal) | 024076 | 024076 | 0 |

Transient KINIT source-image end also grows from octal
`140230` to `143612` with KCC FULL (+1778 words), and from
`113732` to `116516` in GCC lowmem (+1396 words). This
boot-image cost is separate from installed MRES occupancy.

This is installed bitmap CODE growth, not live RAM reduction.
An independent normal v1667 KCC FULL ROOT=auto image built with
PURE enabled but `DAIMOS_PBM_V1667=0` confirms the original
v1666 size exactly: KCORE end `024353` octal, ordinary PURE
MRES **2307** words, KINIT end `140230` octal. Therefore
staging does not increase ordinary uncompressed PURE residency.
The v1663 3456-word RAW example encodes to 161 words in native
tests (3295 process words saved), but a real process has not
undergone this lifecycle, and an APR may coexist with compressed
source during FULL restore. Neither physical peak RAM nor net
system RAM has been measured. The stage must not replace measured
physical guest acceptance or justify an automatic mode3 enable.

The source-built staged GCC lowmem image also boots an actual
64K PDP-6 under SIMH from three TSFS DECtapes without AUXST1:
`MEM 64 K`, `DTC OK`, `PURE LOADED`, and `INIT V1`. An initial
smoke assertion mistakenly checked a v1667 fixed terminal stub
and stopped before launching SIMH; checking the unchanged v1666
KCORE entry corrected the test and the SAME built image PASSED.
No compressed process was created or restored by this boot.

## Next acceptance sequence

Safely attach v1663 bitmap preparation to PURE-only memory
pressure reclaim, allow fixed mode3 terminal VM teardown only
under a positively installed PURE service, and validate physical
process RUN, EXIT, WAIT and PI6 wake/reaper interleavings.
Measure actual MM net and peak use in 48K/64K PDP-6 guests, and
remove unused disk-swap candidate from a separate TSFS-only
swapless boot image while refusing positive BACKSTORE. Strict
32K physical acceptance remains deferred; PURE and bitmap mode3
production default OFF.
