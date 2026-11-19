# v1672: native PDP-6 zombie accounting and real PURE process pressure

Date: 2026-11-18. Exact DAIMOS parent v1671:
`19a348898044c441e6cd5d2ccbd09d7e8d361d0d`.

## Important resident accounting correction

DAIMOS writes `VM_SPACE_NONE` (-1, all 36 bits set) to the
three-word process descriptor when EXIT destroys its user image.
The process remains in the ZOMB state until WAIT reaps the slot.
The fixed `SYS_MEMINFO` and `PROCINFO` routines incorrectly
interpreted the high 18-bit half of this sentinel, octal
`0777777`, as a real user image of **262143 words**. A real 48K
PDP-6 run with two zombies had consequently reported
`PROCESS-WORDS 532246` despite only 49152 physically installed
words. This was **an accounting bug**, not genuine allocation.

The native permanent `syscall_info.s` assembler now rejects only
the all-ones VM sentinel when reporting logical user bytes:
`SYS_MEMINFO` does not add those nonexistent zombie VM words;
`PROCINFO` reports zero logical user words for the zombie.
Real resident and swapped logical VM sizes remain valid.
There is no new process field or backing-layer compatibility.
The proper PDP-10 `CAIN` conditional skip is essential: `CAIN`
skips the next instruction when the values are *not equal*.
Inverting this to `CAIE` fails executable correctness testing.

The change adds **four permanent KCORE words** for a necessary
correctness fix, with no additional resident data allocation:

| Compiler profile | Previous fixed KCORE end | v1672 end |
| --- | ---: | ---: |
| GCC lowmem | 024076 octal | 024102 octal |
| KCC FULL | 024353 octal | 024357 octal |

The source-extracted genuine assembler logic passes native
PDP-6 `p10run` tests for zombie, resident, swapped and FREE
process cases. Two separate mutations deleting the PROCINFO
and MEMINFO sentinel guards are executed and rejected. Corrected
KCORE also links successfully into KCC FULL ROOT=auto (both
memory-backend candidates present). Actual 48K/64K diagnostic
low resident is 17729/17745 words, exactly four words larger
than before the fix. The strict swapless-only TSFS KINIT still
reduces transient GCC image size **1877 words** relative to a
source-matched general image, `113736 -> 110211` octal.

## Live 48K/64K RUN/EXIT/WAIT physical acceptance

Each source-matched GCC lowmem TSFS/PURE image (general tape
versus strict v1668 swapless-only tape) is run under SIMH on a
real PDP-6 48K or 64K machine model. Only in the separately
built **test** root media does a C89 PID1 fixture replace INIT
and a minimal user assembly SLEEP/EXIT executable replace
MOUNTALL. The real kernel RUN, scheduler, EXEC loader, MM owner,
EXIT, and WAIT paths execute without stubs, mock processes or
synthetic allocated extents. DSK/DRM AUXSTORE media are detached;
all four DECtapes are copied for each physical test.

PID1 creates **four children at 48K** or **seven at 64K**, then
WAITs for every native EXIT(0). Three precisely synchronized
physical snapshots are taken by holding test PID1 on CTY input:
before launching, all children present, and all children reaped.
The host Ctrl-E interrupts SIMH to examine the original kernel
MM arena/extent tables, resumes with `go`, then releases the
pending fixture input. This avoids a subtle simulator trap:
SIMH breakpoints on *user virtual* code addresses can falsely
fire at identical numeric physical KCORE instruction addresses.
The obsolete address-breakpoint method is forbidden. The boot
script must also omit its trailing `exit` so Ctrl-E returns to
the simulator monitor rather than terminating SIMH.

The original MM decoder verifies physical core size, the
two-word extent table, sorted and nonoverlapping arenas,
unique per-process APR owners, owner kind, pin counts, managed
free+allocated word conservation and each original permanent
resident immediate. The test independently compares every
native `SYS_MEMINFO` result against actual process APR plus
264-word u-area ownership. Four independently executed unsafe
measurements are rejected: impossible arena geometry, missing
CTY release, forged MM free count and duplicated process owner.

### Measured physical memory

| Core | Profile | Baseline MM free | Children live | After WAIT |
| --- | --- | ---: | ---: | ---: |
| 48K | General or strict | 27441 | 18193 | 27441 |
| 64K | General or strict | 43793 | 27609 | 43793 |

| Core | Live children | Process APR owners | Total MM allocated |
| --- | ---: | --- | --- |
| 48K | 4 | 1 -> 5 -> 1 | 3934 -> 13182 -> 3934 |
| 64K | 7 | 1 -> 8 -> 1 | 3950 -> 20134 -> 3950 |

Each child contributes exactly **2048 APR words plus 264 u-area
words**, or **2312 physical words**. Four children therefore
consume and release **9248** words. Seven consume and release
**16184**. PID1 retains its original 3072-word user APR. One
256-word pinned dynamic extent remains unchanged in all stages:
there is no added pin or leaked memory owner after WAIT. The
largest free contiguous block equals the free-word total at
every observed checkpoint. Both boot configurations produce
exactly the same physical results.

The corrected `SYS_MEMINFO` reports 3336 process words at the
48K baseline and after WAIT, and 12584 under four-child load,
consistent with the real VM and per-process u-area ownership.
The general/strict comparison validates no extra resident RAM
from removing an unused *transient* boot candidate. v1668/v1670
boot-time transient KINIT improvement persists, but disappears
after KINIT source reclamation as expected.

## What remains open

Only three stable post-user points were measured, not every
instruction of memory allocation or every PI6 scheduler event.
No forced RAW PURE reclaim was triggered and compressed bitmap
mode3 is still production disabled. Continuous high-water,
forced reclaim/restore, process EXIT races, PI6 timing,
full physical disk/auto-root regressions and strict 32K remain
separate tasks. Production PURE remains opt-in.

Executable source-backed evidence resides in daimos-testkit:
`tests/validation/pure-runtime-pressure-v1672/`.
DAIMOS design source of truth:
`design/kcore-runtime-process-pressure-v1672.md`.
