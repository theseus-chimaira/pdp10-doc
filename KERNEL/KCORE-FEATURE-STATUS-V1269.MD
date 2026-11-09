# DAIMOS PDP-6 KCORE feature status v1269

**Historical snapshot; superseded for current release gates by
`KERNEL/KCORE-CONSOLIDATED-STATUS-V1286.MD`.** In particular, the
F07/E13/F11 OPEN statements below are the original v1269 status,
not current blockers: v1284 closed those gates (62/64 accepted), with
only F12/F13 OOM activation/signoff outstanding. Keep the older
measurements and chronology unchanged for reproducibility.

**2026-11-09 v1375 decision:** The fourth compressed-disk-swap feature
track below is now CANCELLED. It has been replaced by swapless-only
PURE-text dropping and opt-in compression of mutable RAM; see
`KERNEL/SWAPLESS-MEMORY.MD`. Historical acceptance and sizing
measurements remain unchanged.

Status date: 2026-11-07. DAIMOS source baseline: `1e528f2a`.
The VERSION file remains `0.5`. The current release work targets
version 0.9. This report distinguishes implementation, native test
acceptance, optional experiments and required future integration.

## Implemented and exercised kernel services

- KINIT can bring up a kernel-only 32K PDP-6; 96K and 256K full
  configurations have boot/login/command acceptance on SIMH.
- Native user/process control includes PI6 scheduling, clock sleeps,
  real-time ownership, EXEC/RUN/EXIT/WAIT, process events, PID reuse,
  terminal sessions and HUP, and typed failure/rollback paths.
- The PDP-6 APR VM uses 02000 octal (1024 decimal) word mapping
  alignment. General MM remains word-granular where permitted.
  Process VM, BRK, MM descriptors, pinning, compaction and backed
  raw process swap have native and full-system coverage.
- VFS/provider services, D6FS disk roots, other supported filesystem
  paths, CTY/TTY, PIPE/FIFO, disk/drum/tape storage and device
  controllers have operational implementations. Device class support
  is not a claim of exhaustive physical-hardware acceptance.
- PCLK and optional software wall-clock time, privileged SETTIME and
  DATE SET have native 50/60 Hz PCLK-present/absent regression evidence.
- KCORE and MRES use installed gateway/package interfaces. Boot-loaded
  MRES are currently permanent; runtime unloading/reclaim is not an
  active first-release feature.

## Active pre-final-KCORE feature-freeze requirements

The four accepted active decisions are authoritative in DAIMOS
`TODO.MD` at "Pre-final-KCORE functional freeze inventory":

1. D6 codec: staged v053/v055 components exist, but the grouped-control
   streaming codec has not replaced active D6LZ36 producers and loaders.
   Atomic format cutover, native boot/EXEC/rollback, and final linked
   RAM/stack cost remain outstanding.
2. Shared storage: AUXST1/BACKSTORE, bounded DEVICE/VOLUME/REGION,
   BCACHE and LOGSTORE work has landed incrementally; finish remaining
   production consumer migration, combined storage/media, runtime
   faults, recovery and dynamic RAM accounting. Do not reimplement
   already accepted AUXSTORE interfaces.
3. Privileged MEMFS raw XIP: distinct PURE producer, pinned admission,
   low-level instruction fetch and VM release primitives exist.
   Production restricted no-writing EXEC/RUN, complete scheduled
   process lifetime, swap/BRK exclusion and rollback are not accepted.
4. Superseded: compressed disk swap is cancelled as of v1375.
   Swapless-only PURE omission and opt-in mutable RAM compression
   replace it; neither is production integrated yet.

TSFS v1.2 sharded-metadata development is explicitly POST-KCORE, but
before 0.9 release. Existing TSFS boot/interface regression remains
mandatory. Dynamic/unloadable MRES, general overlays and full 32K DSH
operation are separately deferred, not silently implemented.

## Other kernel-adjacent 0.9 work

- Online D6FS packer, safe MKSWAP provisioning, mixed tape/drum and
  disk/drum inventories, complete BLOCKSET ownership and fault testing.
- The mount policy/layout is staged for 26 slots, with a discovered
  inventory of at most 17 potential volumes; fully activated safe
  provider/FILE/PURE metadata remains incomplete. Older 1..16 notes
  are superseded by this broader staged design. Do not claim all
  26 mounts are available in the live boot configuration.
- Finish final GCC/KCC, simulator/media, KINIT failure-path and
  removable provider lifetime cross-regressions.

## OOM and F07 acceptance

59 of 64 active OOM gates are recorded complete. The A04/F02 release
matrix is accepted. E13 platform/media qualification, F07 maximum
process/descriptor stress and F11 exact-candidate signoff are open.
F12/F13 are post-enable steps. Requester OOM remains production OFF.

The 24-process F07 full PDP-6 guest PASSES against clean committed
DAIMOS `9a841363` without v1208/v1211: 18 extra RUN admissions, real
swap under 24 user slots, held provider lease, requester-only physical
OOM, 18 individual TERM/WAIT/reaps, and final KWP HALT. v1150/v1159,
v1203/v1205/v1207 and evidence hashes pass. A separate experiment
with v1208/v1211 also passes but adds 16 linked SWAP MRES words in
the identically instrumented comparison; its no-equal-partner
necessity, rollback and CPU benefit are not established. Keep those
uncommitted changes separate. See `OOM-F07-24PROCESS-SWAP-CLEANUP-V1264.MD`.

F07 still lacks the full 64-slot and simultaneous high descriptor/
APR/provider/stack load matrix. The observed 24-process snapshot had
12 resident and 12 nonresident slots, 45/136 MM descriptors, 8,065
APR-rounding words, 1,792 swap backing blocks, a 128-word shared
provider workspace, a 180/206-word private kernel stack high-water,
and a 112/256-word idle stack high-water.

## RAM and final optimization

The strict fixed KCORE target is below octal 020000 (8192 words).
Historical GCC v735 KCORE was 8728 words, packaged full KCORE+MRES
17158 words, and single-root endpoint 17205 words. These package
counts include optional modules and are not actual loaded-resident
RAM or the current revision's measured linked size.

The later v1174 KCC 256K inventory records 9944 KCORE words, 272
fixed MM-descriptor words and 8837 installed MRES words, 19053 words
plus a 48-word origin, or 2717 beyond the 16384-word permanent
low-core boundary. These different revisions/profiles must not be
subtracted to assert an optimization delta. Static 16K residency
remains a KCORE/MRES size objective, not a requester-OOM gate.

After functional freeze, measure installed/resident MRES and peak
physical memory, then run source-level data layout and register/
stack-liveness optimization, cold-path P10FOLD cross-domain savings,
boot-constant specialization and final linked-image dead-data audit.
The broad cross-domain fold census suggests at most about 46 one-JRST
words; proceed only if a cold-path plan yields more than about 20
permanent words saved with no KCORE growth or hot-path regression.
Use exact same-profile linked images and native daimos-testkit tests.

## Recommended next sequence

1. Preserve clean F07 evidence; qualify any proposed swap fallback
   in a distinct no-eligible-exchange case before accepting +16 words.
2. Complete 64-slot/high-descriptor F07 and E13/F11 release matrices,
   without enabling OOM prematurely.
3. Finish D6 codec, shared storage, XIP and the swapless-only
   PURE/RAM-reclamation track with RAM/CPU benchmarks.
4. Freeze kernel interfaces and resident feature set; measure current
   32K/48K/256K RAM peaks and perform the final code/data sweep.
5. Work on deferred TSFS v1.2 after KCORE completion and optimization.
