# v1669: actual PDP-6 MM accounting at 48K/64K TSFS KINIT handoff

Date: 2026-11-18. DAIMOS baseline v1668:
`d2c9ed455966d4be611048afb63abc7566ba2be5`.

## Physical memory observations

The v1668 swapless-only TSFS image removes a transient disk-swap
MRES candidate, lowering the GCC lowmem linked KINIT image end
by 1877 words while leaving permanent KCORE and PURE MRES
unchanged. This does not automatically mean 1877 more managed
free words at every stage of boot: other transient source ranges
and the disposable stack change their placement as KINIT code
and package tables move.

v1669 therefore uses SIMH's actual PDP-6 execution breakpoint
to stop at the genuine `kinit_late_start` instruction in four
source-matched GCC lowmem guests: general ROOT=tape/PURE and
strict ROOT=tape/PURE-only, each with 48K and 64K of physical
core. All use a genuine three-member TSFS root and NO attached
auxiliary DSK or DRM swap backing. No new KCORE instrumentation
or extra process word is linked for the measurement.

From the unmodified permanent MM table, the simulator reads
`mm_core_words`, `mm_extent_capacity`, `mm_arena_count`, each
packed arena size/base, `mm_extent_count`, every live two-word
extent, and `sys_resident_words_immediate`. The testkit checks
actual capacity, sorted nonoverlapping arenas/owners, physical
bounds and the invariant that every arena's managed word count
equals its free gaps plus exact MM extent words. All four
snapshots PASS, and separate mutations with impossible arena
geometry, a missing descriptor count and forged free total are
executed and rejected by the original-map oracle.

| Physical core | General managed free | Strict managed free | Change |
| --- | ---: | ---: | ---: |
| 48K | 21947 | 22013 | **+66** |
| 64K | 38299 | 38365 | **+66** |

| Physical core | General largest free | Strict largest free | Change |
| --- | ---: | ---: | ---: |
| 48K | 10022 | 11899 | **+1877** |
| 64K | 26406 | 28283 | **+1877** |

Both 48K guests report identical low resident **17725** words,
and identical dynamic MM allocations **598** words in six
extents. The 64K pair likewise has resident **17741** words
and dynamic MM allocations **614** words in six extents. All
allocated extents at this checkpoint are kernel-dynamic (type3).

## Why the image reduction and total free improvement differ

The high arena begins above `__kinit_image_end` and its reserved
stack. The strict image end is 1877 words lower, so this arena
is exactly 1877 words larger. Another, earlier managed arena
becomes 1811 words smaller as the transient KINIT late/source
segments shift. The start of protected late KINIT moves down
by 66 words. At the checkpoint, the net sum of all managed
free gaps therefore increases only **1877 - 1811 = 66 words**.

A large, contiguous PDP-6 APR needs one sufficiently large free
extent; gaining 1877 words in the largest high-memory gap may
still be valuable even where total free words grow only 66.
Conversely the 1877 word linker-image shrink is NOT permission
to claim 1877 more free runtime words or a net compressed RAM
benefit. All these measurements are actual physical *one-time*
snapshots at the pre-reclamation KINIT checkpoint.

## Additional 48K physical smoke test

The unchanged source-built strict 64K-capable image was also
run on SIMH with core set to 48K and no AUXSTORE backing. The
real console showed `MEM 48 K`, `DTC OK`, `PURE LOADED`, and
`INIT V1`, and the guest stayed alive until a controlled
85-second timeout. The same observation was reproduced by
`run-runtime48-v1669.sh` in the separate daimos-testkit.

The sustained 48K INIT boot validates reachability, NOT peak
process-pressure capacity, strict physical 32K acceptance or
future mode3 compression admission. The earlier v1668 physical
64K successful and positive BACKSTORE SWAP FAIL/HALT tests
remain valid.

## Explicitly outstanding

The actual all-time peak during KINIT allocation, source-image
reclamation, INIT creation and process pressure has NOT been
captured by four fixed checkpoints. Continuous allocator-event
instrumentation, KINIT stack-watermark bounds, post-reclamation
48K/64K MM-owner pressure tests, production bitmap compressor,
RUN/EXIT/WAIT/PI6 admission and strict physical 32K remain open.
No v1669 kernel feature or permanent RAM allocation was added.

The testkit path is
`tests/validation/pure-kinit-physical-memory-v1669/`;
DAIMOS code/design record is
`design/kcore-tsfs-physical-mm-checkpoint-v1669.md`.
