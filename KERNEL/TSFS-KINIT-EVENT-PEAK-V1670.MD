# v1670: physical 48K/64K KINIT allocator-event high-water

Date: 2026-11-18. Accepted DAIMOS parent v1669:
`e2c1d0d68eb1e46e9a7b6b27f79863ce10d9cc91`.

## Why a trace was necessary

The v1668 swapless-only TSFS boot image removes a DISK SWAP MRES
candidate from transient KINIT, reducing the GCC lowmem linked
image by 1877 PDP-6 36-bit words without changing permanent
KCORE or installed PURE residency. v1669 confirmed that a real
48K/64K physical pre-reclamation checkpoint has only **66 more
total free MM words**, but a **1877-word larger contiguous free
region**. Static source/image addresses and a single checkpoint
cannot establish maximum boot-time nonfree memory.

v1670 adds an event-boundary trace without modifying any DAIMOS
kernel or KINIT executable. daimos-testkit controls SIMH using
breakpoints at the **actual** linker-map addresses for the
existing `mres_init` start, `mm_extent_insert`,
`mm_boot_reserve`, `mm_add_free`, `mm_free`, possible process
resize/compaction paths, and the final `vm_enter_initial_user`
entry. Every time a memory-mutating operation begins, the
simulator reads the physical permanent MM arena and extent
descriptors and resident immediate. The initial retained sample
is AFTER MM initialization at `mres_init`, and the final sample
is BEFORE the first user instruction with late-KINIT memory
already reclaimed. No memory scanner, resident allocation or
additional runtime MM field is installed.

## Methodological guarantees and limitations

The test verifies the actual CPU core size and MM extent table
capacity, nonoverlapping sorted physical arenas/extents, valid
owner types, and exactly conserved total managed arena words
across each sampled state. The test aborts on missing early/late
markers, corrupt MM fields, an unexpected compaction or direct
process extent resize, or failure to reach the first-user handoff.
Three independent unsafe physical trace/report variants are
executed and rejected: impossible arena geometry, missing final
user-entry event and forged sampled maximum. Each image uses
isolated copies of all four DECtape media, with DSK/DRM absent.

Sampling at MM mutation entries makes each reading a stable
allocator state before the next operation, and the final
VM-entry sample captures the post-reclaim state. It DOES NOT
prove a maximum inside every individual allocator instruction,
nor account for runtime process/VM memory pressure after the
first user enters. The reported maximum is therefore the
**physical stable MM event-boundary high-water**, not a fully
continuous instruction-level or lifetime RAM high-water.

## Actual PDP-6 results

| Physical core | General samples | Strict samples | Highest nonfree event |
| --- | ---: | ---: | --- |
| 48K | 53 | 52 | `mres_init` |
| 64K | 53 | 52 | `mres_init` |

Both profiles execute 20 insertion and 12 permanent-reservation
entries. General boot publishes 19 free-arena operations after
MM readiness, strict boot 18. Neither guest triggers an ordinary
MM free, process extent resize or compaction entry in this
initialization interval. All observed MM states pass real-word
conservation, source-linked owner and core-bounds validation.

| Installed core | General minimum free | Strict minimum free | Gain |
| --- | ---: | ---: | ---: |
| 48K | 11944 | 13821 | **1877 words** |
| 64K | 28312 | 30189 | **1877 words** |

| Installed core | General highest nonfree | Strict highest nonfree | Change |
| --- | ---: | ---: | ---: |
| 48K | 37208 | 35331 | **-1877 words** |
| 64K | 37224 | 35347 | **-1877 words** |

The maximum occupied/reserved core from all sampled MM event
boundaries is observed at initialized `mres_init` start in
every case, BEFORE permanent MRES copying and reclaim. Thus
the v1668 1877-word linked image reduction is ALSO an actual
**1877-word improvement at the sampled boot high-water** for
both physical 48K and 64K guest sizes.

The first-user-entry snapshot is identical general/strict:

| Installed core | Free MM words | Low resident | Live MM allocations |
| --- | ---: | ---: | ---: |
| 48K | 26421 | 17725 | 4958 words (8 extents) |
| 64K | 42773 | 17741 | 4974 words (8 extents) |

The process APR owns 4096 of the allocated words; the remaining
862/878 are kernel-dynamic owners. Normal and strict images
install exactly one identical PURE MRES, so after reclaiming
transient KINIT memory their managed free total is identical.
The gain at boot does NOT persist as additional free runtime
core. This result also explains why v1669's pre-reclamation
one-time +66 net free and +1877 largest free figures are not
contradictory: they refer to different measured moments.

## Open work

Exact instruction-level allocator micropeaks and KINIT stack
watermarks, 48K/64K post-user process-pressure episodes,
real compressed mode3 creation/restore/terminal paths,
RUN/EXIT/WAIT/PI6 concurrency and strict physical 32K remain
separate acceptance gates. v1670 changes no DAIMOS production
kernel source and does not enable mode3 in a running guest.

Original source-linked test entry:
`daimos-testkit/tests/validation/pure-kinit-peak-v1670/`.
DAIMOS design record:
`design/kcore-kinit-event-peak-v1670.md`.
