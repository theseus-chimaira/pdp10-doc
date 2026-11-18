# v1666: split PDP-6 terminal release into selected memory MRES

Date: 2026-11-18. Parent DAIMOS v1665:
`08a31f16c1c5a7793e8bc59995efaed238954f31`.

## Why a boot-selected terminal release

v1665 makes KINIT select one memory-management service from the
actual root controller class and positively validated BACKSTORE
capacity. Still, the fixed `proc_swap_detach()` had interpreted
disk swap blocks itself and linked a separate RAW PURE MM-owner
scanner, wasting KCORE words on systems that could only use one
backend. v1666 eliminates that dual permanent implementation.

The fixed `proc_swap_entry_v481.s` adds one default-refusing
`proc_swap_term_v1666` JRST alongside the existing installed
`proc_swap_reclaim` and `proc_swap_service_one` entries. KINIT
patches the third entry exactly once, from the same MRES as the
other two. The disk MRES exports its terminal at index 7, PURE
at index 2. KCORE's common `proc_swap_detach()` checks mode0;
if already RESIDENT it can complete metadata cleanup without
calling any backend. For nonresident modes it requires successful
`proc_swap_term_v1666(slot, descriptor, saved_record)` before
clearing the swap record, stable u-area backing and mount-staging
credential bits. Refused release leaves those fields unchanged.
An uninstalled MRES returns -1 for any nonresident release.

## The two independently owned implementations

Disk swap MRES now includes original PDP-6 ASM leaf
`proc_swap_mres_terminal_v1666`. It requires exact mode1,
zero process APR, nonzero exact disk block length and sufficient
accounted swap blocks. It calls original `backstore_free`
without requiring new allocation to remain enabled. Only when
the backing allocation is successfully released does it reduce
the used-block count and publish mode1->mode0. An impossible
post-free publication mismatch remains a fatal invariant, not a
second fallible free or guessed recovery.

PURE MRES now contains the exact RAW PACKED_PROCESS owner release
originally named `pdp6_pure_detach_v1609`, formerly part of fixed
KCORE's `proc_swap.s`. It continues to validate TRANSITION,
the stable u-area and saved executable identity, absent APR,
valid PURE suffix size, exactly one unpinned process-owned MM
extent and original MM-free success. v1666 adds exact mode2
verification BEFORE attempting the free, so a corrupt mode
cannot retire the wrong kind of owner. This module knows no
disk-block allocation details and uses no extra per-process word.

The fixed KCORE still refuses compressed mode3 in
`vm_space_destroy`; the staged v1659..v1664 bitmap compressor,
commit and FULL APR restore remain unlinked. The v1609 RAW
terminal is not a compressed terminal dispatcher. A subsequent
PURE-only integration must add mode3 release/restore inside PURE
MRES before admission becomes possible. The disk MRES should
remain completely unaware of bitmap ownership.

## Measured net RAM change against accepted v1665

| Source-built profile | Fixed KCORE saving | Installed MRES growth | Net permanent saving |
| --- | ---: | ---: | ---: |
| KCC FULL, DISK | 114 words | 42 words | **72 words** |
| KCC FULL, PURE | 114 words | 90 words | **24 words** |
| GCC lowmem TSFS, PURE | 111 words | 90 words | **21 words** |

KCC FULL `__kcore_low_end` changes octal 024535->024353.
KCC DISK MRES goes 1,795->1,837 resident words (128 BSS
unchanged), PURE MRES 2,217->2,307 resident words (zero BSS).
The KCC FULL ROOT=auto KINIT end grows from octal 140154 to
140230 (44 transient words); unused candidates still occupy
transient KINIT input, but only the selected MRES is installed.

GCC lowmem fixed bound changes octal 024255->024076. GCC PURE
MRES increases from 1,773 to 1,863 resident words. These are
linked-size comparisons from corresponding accepted source-built
v1665 images; not reductions in measured physical high-water or
process RAM consumption. Dedicated swapless-only candidate
removal remains open.

## Acceptance evidence

The testkit links the actual source KCORE `proc_swap_detach`
with the original source-extracted PDP-6 disk terminal assembler,
real storage-mode assembler and original `vm_space_destroy` leaf.
The native cases cover disk success/repeated detach, refused
free, zero/insufficient blocks, absent record, wrong mode2/mode3,
still-mapped APR, SIDL staged mount, and RESIDENT bypass. Native
tests also use the actual default-refusing KCORE JRST without
an installed MRES, ensuring mode1 cannot be silently discarded.
Both independently executed unsafe source mutations (uninstalled
terminal falsely succeeds; disk terminal ignores live APR) are
rejected by the native PDP-6 oracle. The historical v1609 RAW
teardown and source mutation, v880/v957 disk host tests, v1613/
v1665 source-linked KINIT selector tests and v1659..v1664 staged
bitmap/FULL restore native tests all PASS.

The actual source-built 64K PDP-6 with GCC lowmem TSFS on three
DECtapes, no attached auxiliary disk/drum backing and PURE
opt-in, reaches `MEM 64 K`, `DTC OK`, `PURE LOADED` and `INIT V1`
without a root or INIT failure. KCC FULL ROOT=auto image links
with both candidates and the single fixed terminal trampoline;
the fixed KCORE map contains `proc_swap_term_v1666` but NOT
`pdp6_pure_detach_v1609`, which appears only in the PURE MRES
map. The disk terminal similarly appears only in the disk MRES.

## Remaining acceptance gates

Mode3 compressed terminals/RUN/EXIT/WAIT and PI6 scheduler are
not yet live; integrating them must remain isolated to PURE MRES.
Full 48K/64K physical peak and net memory accounting, a dedicated
swapless-only TSFS image without disk candidate, and strict 32K
physical acceptance are open. PURE default OFF, mode3 disabled.
