# v669 diagnostic refinement — meaning of ?RT

`?RT` is `KINIT_ERR_RT` in `kinit_boot()`: root mount failed or
`root_has_init()` did not find a regular `/SYSTEM/INIT`. The failure
occurs before scheduler pressure begins; it is not evidence that
BACKING rollback itself failed. Testkit v669 now has a root-media
preflight, but which of these two root checks failed remains unknown.

# v669 — full-system swap acceptance still blocked

Isolated native PDP-6 BACKING/MM rollback tests v664-v668 PASS, but the
historic full-system testkit v298 has a stale assumption that
`proc_swap_out` is an absolute KCORE symbol. It now resides in the
loadable SWAP MRES. The corrected testkit v669 runner reaches simulator
execution; without high-volume tracing, the guest prints `?RT` and
halts at `065206` before completion of the pressure workload. A
traced run timed out after repeated breakpoints. These are **not**
full-system rollback acceptance passes. Diagnose boot/startup before
resuming scheduler fault injection. No production code changed.

# v668 — verified recoverable MM-free rollback

An isolated native PDP-6 test now executes the production swap-out
late rollback branch with `mm_unpin` successful and `mm_free` failing.
It confirms the process remains resident, the original swap record is
unchanged, the temporary BACKING allocation is freed, and TRANSITION
and the provisional u-area backing field are cleared. Testkit v668
PASS. This is not a full scheduler-level injected fault or an assertion
of physical multi-block I/O atomicity. No production code changed.

# v667 — unsafe late swap-out unpin cleanup fixed

The v666 audit did not establish safe behavior when `mm_unpin` itself
failed: the prior rollback could clear TRANSITION and resume a process
with unresolved physical pin ownership. The PDP-6 swap-out assembly now
routes either failed unpin (normal postwrite path or error cleanup)
to the existing storage/MM invariant handler, rather than silently
resuming. A successful unpin followed by failed `mm_free` retains its
existing recoverable prepublication rollback.

Testkit v667 source and isolated native PDP-6 fault branches PASS,
the adjusted v666 regression PASSes, and a fresh GCC 32K disk-root
image boots through `proc_boot_init` with BSET/D6FS loaded. Full-system
fault/scheduler testing is not implied. This is a correctness fix,
not KCORE size optimization.

# v666 — swap late-memory-release rollback audit

The PDP-6 `proc_swap_out` implementation writes a provisional BACKING
span, then attempts `mm_unpin` and `mm_free`. Failure of either takes
the prepublication rollback path: release provisional BACKING, clear
the temporary u-area backing field and TRANSITION. Only successful
memory release leads to DISK mode publication. A new testkit v666
source control-flow regression PASSes, as does the earlier native v665
partial-write test. Live fault-injected MM unpin/free and scheduler
recovery remain unverified; no production change was required.

# v665 — multi-block BACKING failure contract

Native PDP-6 instruction test of the production BACKING transfer
routine confirms that a failing second block of a three-block write
returns an error and never writes the third. Separate source checks
confirm swap-out and MEMFS eviction publish their new backing records
only after successful writes, freeing abandoned allocations on the
write-error branch. Both testkit v665 checks PASS.

This is **failure atomicity of publication**, not physical atomicity
of multi-block storage: earlier blocks can already have been written.
Full-system in-flight controller faults, late MM rollback paths and
persistent snapshot recovery still need acceptance tests. No change
to production code, media layout or KCORE size optimization.

# v664 — isolated native BACKING failure coverage

Native PDP-6 execution verifies that an unavailable multi-device
VOLUME mapper cannot silently fall back to direct DEVICE addressing.
The direct-device path remains operational. This test uses the real
BACKING transfer assembly with instrumented device entry points;
production KCORE and AUXST1 formats are unchanged. Full-system
partial-transfer recovery remains pending.

# v663 — VOLUME compatibility evidence

The existing native multi-member D6FS suite PASSed on two/four-member
HDD volumes after the DEVICE/VOLUME split. Backing-mapper fault-injection
is still open; see KERNEL/BLOCKSET.MD. The testkit v663 launcher avoids
host relay binaries inherited from a different CPU architecture.

# v662 — BACKING runtime consumers

Process swap initialization and MonitorFS SWAP statistics both query
`backing_region_blocks()` for the logical capacity. The historical
`blockset_tail_blocks()` symbol remains a binary-compatible alias;
the linked map places both at octal `005071`. The internal shared
transfer entry is named `backing_region_io` while preserving the
existing `backstore_read/write` and legacy BLOCKSET entry labels.
The change retains a single DEVICE/VOLUME I/O path, no extra resident
state or dispatch layer, and unchanged AUXST1/BLOCKSET formats.

Testkit v662, v660, v659 PASS. The fresh GCC 32K image builds and
boots on PDP-6 SIMH to `proc_boot_init` with BSET/D6FS loaded; D6FS
checks CLEAN. Native multi-member mapper-loss injection remains open.

# v661 — LOG boot REGION routing

LOG bootstrap reads and writes now call the bounded AUXST1 REGION
operations directly. The intermediate `auxstore_boot_log_read/write`
wrappers have been deleted. The preexisting root-backed BLOCKSET LOG
fallback is retained. LOG is a durable record ring, BACKING owns the
temporary swap/MEMFS allocation bitmap, and CACHE contains only
discardable copies; these are independent policies sharing only
physical addressing. No persistent KCORE dispatch layer was added.

# v660 — DEVICE/VOLUME/REGION resident state names

The backing bridge retains its existing three resident configuration
words, now named `device_region_map`, `volume_data_blocks` and
`backing_region_count`. The historical `blockset_direct_*` names remain
compatibility aliases on the **same addresses**; the old tail-count
function is likewise aliased by `backing_region_blocks()`. Process swap
boot calls the latter. These are semantic source names only: no new
storage dispatch layer or persistent metadata format is introduced.

Source regressions v658-v660 PASS, fresh native GCC 32K disk build
PASS, D6FS checker CLEAN, and PDP-6 SIMH reaches `proc_boot_init` with
BSET/D6FS loaded. Multi-member mapper-loss fault injection remains
open. No KCORE size optimization is part of this change.

# v659 — VOLUME mapper-unavailable behavior

The permanent BACKING transfer service now fails closed when a
multi-device VOLUME is configured but its BLOCKSET mapper is absent.
Previously it fell through to direct DEVICE arithmetic, potentially
addressing the wrong sectors. A direct DEVICE range still bypasses
the VOLUME mapper. This changes neither AUXST1 media layout nor
BLOCKSET binary service ABI.

The v659 fail-closed source test and v658 routing test PASS. The
fresh native 32K GCC D6FS/disk image builds and boots through
`proc_boot_init` with BSET/D6FS loaded. Native mapper-loss fault
injection remains outstanding. No code-size optimization was run.

# v658 — separate direct DEVICE regions from VOLUME tail mapping

The permanent storage bridge now exposes `device_range_configure` for
single physical DSK/DRM regions and `volume_tail_configure` for
multi-member root tails translated by the existing BLOCKSET MRES.
`blockset_direct_configure` remains an ABI-compatible symbol alias.
Both I/O paths use one common physical DSK270/DRM236 dispatcher;
neither AUXST1 nor BLOCKSET on-media formats change.

The intended nomenclature is DEVICE (one physical drive), VOLUME
(optional multi-device block mapping), REGION (bounded block range),
BACKING (shared authoritative swap/MEMFS space), LOG (persistent records)
and CACHE (discardable copies). This terminology is a staged source
migration, not a new stack of generic runtime storage layers.

Fresh 32K lowmem GCC PDP-6 SIMH D6FS disk boot reached
`proc_boot_init` with BSET and D6FS loaded. Testkit direct/volume
routing inspection PASS. Multi-member and fault-path integration
remain open; DTFS/TSFS features are deferred for now.

# v657 — shared AUXSTORE raw-region I/O (first production feature increment)

Production AUXSTORE bootstrap exports `auxstore_boot_region_read` and
`auxstore_boot_region_write` for BACKSTORE, LOGSTORE and CACHESTORE,
using partition-relative block addresses. The region selector and
block offset must be valid; no request may escape into an adjacent
partition. LOGSTORE bootstrap read/write now uses this shared
interface. Existing raw BACKSTORE allocation and physical I/O are
preserved pending migration and rollback/recovery validation.

Host testkit v657 region bounds PASS; existing AUXSTORE choice v492
and D6FS overlap v507 PASS. This is feature implementation, not a
KCORE optimization, and does not close the remaining BSTORE migration.

The online D6FS packer and agreed storage/admin extensions are in
version 0.9 release scope. DTFS/TSFS features are temporarily deferred
while KCORE feature implementation proceeds.

### v484: BACKSTORE allocator resident split (2026-10-27)

The authoritative bitmap and four associated state words stay in KCORE,
shared by process swap and MEMFS. The allocator TEXT has moved into the
permanently installed SWAP/shared MRES. KCORE exposes the original four
`backstore_*` functions through one-word JRST entries patched by SWAP
MINIT. Neither the API nor the single-bitmap ownership changes; this
MRES is installed on the existing boot path and is not a removable
optional module while its clients are active.

On the PDP-6 full GCC lowmem link KCORE shrank 8652 -> 8548 words, and
shared SWAP MRES grew 308 -> 412 words, giving no change in combined
permanent occupancy (disk 13415, tape 13339 words). The corrected build
puts `kcore_end.dobj` last: backing state occupies 020620..020623,
`__kcore_low_end` and dynamically allocated MM extents begin at 020624.
The earlier experimental KINIT failure resulted from this boundary
overlap, not a need for another allocator or a change in storage ABI.
Native 32K swap-wait and swap-stress and the standalone PDP-6 BACKSTORE
fixture pass. The boundary regression lives in daimos-testkit.

### v481 KCORE reduction boundary (2026-10-27)

The process swap *transfer/reclaim* implementation now resides in a pinned
SWAP MRES. This does **not** transfer ownership of the common BACKSTORE
bitmap. `backstore.s` is still KCORE (108 pre-link TEXT, 4 BSS words):
MEMFS uses the same allocation/free interface and backing I/O paths.
Its migration must use a shared service and preserve one authoritative
bitmap; adding a second allocator or moving it behind an optional SWAP
package would violate this contract.

During KINIT, `proc_swap_boot_init` currently allocates one process swap
record per slot, the bitmap (`ceil(backstore_blocks/36)` words), and a
two-bit-per-slot process storage-state map together. Zero backing capacity
does not imply the executable-backing records are unnecessary. Consider
splitting their lifetimes only after validating all PROC, MEMFS and VM
consumers and measuring runtime MM occupation, not merely static KCORE.

# DAIMOS BSTORE / BCACHE DESIGN AND IMPLEMENTATION PLAN

STATUS: STORAGE CONSOLIDATION IMPLEMENTED; AUXSTORE MEDIA TOOLING/LAYOUT
UPDATED 2026-10-06.  THE MIGRATION PLAN BELOW IS RETAINED AS THE DESIGN
CHECKLIST, WITH REMAINING TEST/MAINTENANCE ITEMS CALLED OUT EXPLICITLY.

## 1. GOAL

DAIMOS SHOULD HAVE ONE SMALL 128-WORD BLOCK MECHANISM FOR THE STORAGE CASES
THAT ARE ACTUALLY COMMON, WITHOUT FORCING DIFFERENT PERSISTENCE SEMANTICS INTO
ONE OVERGENERAL INTERFACE.

THE COMMON UNIT IS ONE OCTAL 0200-WORD BLOCK.  D6FS, DTFS/TSFS, DSK270,
DRM236, PROCESS SWAP, MEMFS EVICTION, AND LOGSTORE ALREADY USE THAT SIZE.
NO VARIABLE BLOCK SIZE, BYTE PACKING, OR CONVERSION LAYER IS REQUIRED.

THE TARGET HAS THREE SMALL PARTS:

```TEXT
BACKSTORE   AUTHORITATIVE ALLOCATED BLOCKS FOR PROCESS SWAP AND MEMFS
BCACHE      CLEAN/DISCARDABLE BLOCK COPIES FOR FILESYSTEM READ ACCELERATION
AUXSTORE    HARDWARE-INDEPENDENT DSK/DRM PLACEMENT FOR BACKSTORE, CACHESTORE,
            AND LOGSTORE REGIONS
```

LOGSTORE KEEPS ITS OWN RING/RECOVERY SEMANTICS.  IT USES AUXSTORE I/O BUT IS
NOT AN ALLOCATED BACKSTORE OBJECT AND IS NOT CACHED AUTOMATICALLY.

## 2. LAYERING

```TEXT
                     FILESYSTEMS

            D6FS       DTFS       TSFS
              \          |          /
               \         |         /
                +------ BCACHE ----+
                       /     \
                  RAM L1     CACHESTORE L2
                                |
                             AUXSTORE
                                |
                            DRM / DSK

                     MEMORY BACKING

            PROCESS SWAP       MEMFS
                  \             /
                   \           /
                    BACKSTORE
                        |
                     AUXSTORE
                        |
                     DRM / DSK

                       LOGGING

                     LOGSTORE
                        |
                     AUXSTORE
                        |
                     DRM / DSK
```

BCACHE IS OPT-IN.  RAW PROCESS SWAP AND LOGSTORE I/O MUST NOT PASS THROUGH THE
CACHE.  FILESYSTEMS CHOOSE TO USE BCACHE AROUND THEIR NORMAL BLOCK READ/WRITE
PATHS.

## 3. BACKSTORE

BACKSTORE REPLACES THE SWAP-SPECIFIC ALLOCATOR WITH A GENERAL RAW-BLOCK
ALLOCATOR.  IT OWNS EXACTLY ONE BITMAP, ONE BIT PER 0200-WORD BLOCK.

THE REQUIRED INTERFACE IS CONCEPTUALLY:

```TEXT
BACKSTORE_ALLOC(BLOCKS, RESERVE, &FIRST)
BACKSTORE_FREE(FIRST, BLOCKS)
BACKSTORE_READ(FIRST, BLOCKS, BUFFER)
BACKSTORE_WRITE(FIRST, BLOCKS, BUFFER)
```

`RESERVE` IS THE NUMBER OF FREE BLOCKS THAT MUST REMAIN AFTER AN ALLOCATION.
PROCESS SWAP USES ZERO.  MEMFS USES THE PROCESS-SWAP RESERVE.  THIS REPLACES
THE SPECIAL `SWAP_STORE_ALLOC_MEMFS()` POLICY WITHOUT A SECOND ALLOCATOR.

PROCESS SWAP AND MEMFS KEEP THEIR OWN MAPPING RECORDS ABOVE BACKSTORE.  BSTORE
DOES NOT KNOW WHETHER A RUN BELONGS TO A PROCESS, FILE, OR SNAPSHOT.

PERSISTENT MEMFS MAY RESERVE ITS EXISTING VERSIONED SNAPSHOT EXTENT AT BOOT.
THE GENERAL BACKSTORE ALLOCATION BITMAP DOES NOT BECOME A PERSISTENT ON-MEDIA
ALLOCATOR.

## 4. BCACHE

BCACHE IS CLEAN-ONLY AND WRITE-THROUGH.  IT NEVER OWNS THE ONLY VALID COPY OF
DATA.  THERE IS NO DIRTY LIST, WRITEBACK DAEMON, FLUSH SCHEDULER, OR CACHE
JOURNAL.

READ:

```TEXT
LOOK UP CLEAN COPY
  HIT  -> RETURN IT
  MISS -> CALLER READS AUTHORITATIVE DEVICE, THEN INSERTS COPY
```

WRITE:

```TEXT
CALLER WRITES AUTHORITATIVE DEVICE FIRST
  FAILURE -> CACHE IS NOT UPDATED
  SUCCESS -> UPDATE/REPLACE CLEAN CACHE COPY
```

THE INITIAL RAM L1 IS ONE CLEAN TWO-BLOCK CACHE SHARED BY D6FS AND DTFS/TSFS.
IT REUSES THE ALREADY-PERMANENT SERIALIZED FILESYSTEM TRANSFER WORKSPACE AS
SLOT 0 RATHER THAN ALLOCATING TWO ADDITIONAL FULL DATA BLOCKS.

CURRENT L1 SHAPE:

```TEXT
KCORE:
    1 WORD         WORKSPACE TAG
    0200 WORDS     FS_BLOCK_WORKSPACE / CACHE SLOT 0

MANAGED CORE:
    WORD 0         PRIVATE-SLOT TAG
    WORDS 1..0200  CACHE SLOT 1
```

THE RECLAIMABLE MANAGED-CORE SLAB IS THEREFORE OCTAL 0201 WORDS (129
DECIMAL), DOWN FROM THE FORMER OCTAL 0403-WORD PRIVATE SLAB.  THE ONE
ADDITIONAL FIXED TAG WORD IS MORE THAN OFFSET BY ELIMINATING ONE DYNAMIC
128-WORD DATA BLOCK.  INCLUDING THE IMPLEMENTATION/CALL-SITE CODE GROWTH,
THE CHANGE SAVES 109 PHYSICAL RAM WORDS WHILE THE CACHE IS ACTIVE.

THE TWO ENTRIES REMAIN FULLY ASSOCIATIVE.  A WORKSPACE-BACKED MISS PRESERVES
THE VALID WORKSPACE BLOCK INTO THE PRIVATE SLOT BEFORE THE CALLER REFILLS THE
WORKSPACE.  A PRIVATE-SLOT HIT REQUESTED INTO THE WORKSPACE PROMOTES THAT KEY
TO SLOT 0.  THERE IS NO VICTIM-HAND WORD, LRU LIST, HASH CHAIN, REFERENCE
COUNT, OR DIRTY STATE.

DIRECT USERS OF FS_BLOCK_WORKSPACE MUST INVALIDATE THE WORKSPACE TAG BEFORE
MODIFYING SCRATCH CONTENTS.  THE COMMON ZERO-WORKSPACE HELPER DOES THIS
AUTOMATICALLY.  D6FS INVALIDATES BEFORE WRITE-THROUGH AND RE-TAGS ONLY AFTER
THE AUTHORITATIVE WRITE SUCCEEDS, SO A FAILED WRITE CANNOT LEAVE MODIFIED
WORKSPACE DATA MARKED CLEAN.

## 5. CACHE KEYS

ONE 36-BIT WORD IDENTIFIES A CLEAN BLOCK COPY.  THE EXACT BIT PACKING MAY BE
TUNED DURING IMPLEMENTATION, BUT IT MUST REPRESENT:

```TEXT
SOURCE / MOUNT IDENTITY
MEDIA OR MOUNT GENERATION
LOGICAL BLOCK NUMBER
```

REMOVABLE MEDIA MUST CHANGE GENERATION WHEN A NEW MEDIUM IS ACCEPTED.  OLD DTC
CACHE ENTRIES THEN BECOME UNREACHABLE WITHOUT A CACHE SCAN.

D6FS USES A STABLE MOUNT/BACKING IDENTITY PLUS LOGICAL BLOCK NUMBER.  DTFS AND
TSFS USE THE DTC UNIT/MEDIA GENERATION PLUS PHYSICAL TAPE BLOCK NUMBER.

## 6. OPTIONAL CACHESTORE L2

CACHESTORE IS A FIXED AUXSTORE REGION USED ONLY WHEN THE AUTHORITATIVE SOURCE
IS MATERIALLY SLOWER THAN AUXSTORE.  THE PRIMARY V0.9 USE IS DECTAPE CACHED ON
A DRUM OR, SECONDARILY, A FASTER DISK.

NORMAL D6FS-ON-DISK MUST NOT COPY DISK BLOCKS INTO ANOTHER REGION ON THE SAME
DISK BY DEFAULT.  D6FS STILL USES RAM L1.

CACHESTORE IS DIRECT-MAPPED.  USE A POWER-OF-TWO NUMBER OF DATA SLOTS:

```TEXT
128, 256, 512, 1024, 2048, ...
```

THE SLOT IS:

```TEXT
HASH(KEY) & (SLOTS - 1)
```

ONE 128-WORD TAG BLOCK HOLDS 128 TAG WORDS.  TAGS LIVE ON AUXSTORE, NOT IN
RESIDENT RAM.  CACHESTORE REQUIRES NO ALLOCATION BITMAP.

CACHESTORE IS DISPOSABLE ACROSS BOOT.  KINIT MAY INVALIDATE IT BY CLEARING THE
TAG BLOCKS.  DO NOT ADD PERSISTENT EPOCH/JOURNAL STATE UNLESS MEASUREMENT SHOWS
THAT BOOT-TIME TAG CLEARING IS A REAL PROBLEM.

## 7. AUXSTORE REGIONS

NEW NON-FILESYSTEM AUXILIARY MEDIA SHOULD USE A SMALL DEVICE-INDEPENDENT
DESCRIPTOR WITH FIXED REGIONS:

```TEXT
AUXSTORE DESCRIPTOR
LOGSTORE
BACKSTORE
CACHESTORE TAGS
CACHESTORE DATA
```

THE DESCRIPTOR RECORDS START/COUNT FOR LOGSTORE, BACKSTORE, AND CACHESTORE.
AS OF V638, HOST `MKAUXSTORE` REQUIRES EXPLICIT DESTRUCTIVE CONFIRMATION;
DRY-RUN AND HOST SAFETY TESTS REFUSE UNKNOWN OR NONEMPTY TARGET REGIONS.
THE NORMAL PDP-6 BUILD CONVERTS ONLY AN EXACTLY VERIFIED ONE-MEMBER CLEAN
DBOOT HEADER TO AUXST1 VIA EXPLICIT `--CONVERT-DBOOT`, CHECKING THE D6FS
BASE/COUNT, SUPERBLOCK OFFSETS, AND RESERVED TAIL BEFORE WRITING. THE
NON-ROOT D6FS PARTITION IS PRESERVED BYTE FOR BYTE. THESE ARE IMAGE-BUILD
OPERATIONS, NOT AUTOMATIC FORMATTING AT DAIMOS BOOT.
THE CACHE TAG/DATA SPLIT IS DERIVED FROM CACHESTORE SIZE.  AUXSTORE PREFERS DRM
WHEN PRESENT, THEN DSK.  IT DOES NOT REQUIRE D6FS ON THE AUXILIARY DEVICE.

THE NORMAL PDP-6 BUILD FORMATS DRM0 AS A CO-RESIDENT AUXILIARY MEDIUM RATHER
THAN AS A SECOND ROOT SET.  PHYSICAL BLOCK 0 HOLDS THE AUXSTORE DESCRIPTOR.
THE DESCRIPTOR'S OPTIONAL WORDS 5 AND 6 RECORD A NON-ROOT D6FS BASE/COUNT AND
ITS TWO LOGICAL SUPERBLOCK LOCATIONS.  THE D6FS REGION IS INDEPENDENT OF THE
AUXSTORE REGIONS; IT CONTAINS NO BOOT STREAM AND OWNS NO SWAP OR LOGSTORE
RESERVATION.  THE REMAINING PHYSICAL TAIL IS PARTITIONED AS LOGSTORE,
BACKSTORE, THEN CACHESTORE.  `MKAUXSTORE` IN DAIMOS `TOOLS/HOST` FORMATS AND INSPECTS
THIS DESCRIPTOR.  THE NORMAL DEFAULT GEOMETRY IS:

```TEXT
BLOCK 0             AUXSTORE DESCRIPTOR
1..                 NON-ROOT D6FS, 011367 BLOCKS
11370..11767        LOGSTORE, 0400 BLOCKS
11770..15767        BACKSTORE, 04000 BLOCKS
15770..17777        CACHESTORE, 02010 BLOCKS
```

DRM0 THEREFORE USES THE FULL TYPE-236 020000-BLOCK CAPACITY.  DRM1-3 ARE
BLANK IN THE NORMAL BOOT PROFILE.  AN EXPLICIT DRM BOOT CONTINUES TO USE THE
SEPARATE BOOTABLE DRUM-SET PATH.

OLD D6FS/BLOCKSET SWAP/LOGSTORE RESERVATIONS MAY REMAIN READ-COMPATIBLE DURING
MIGRATION, BUT NEWLY FORMATTED AUXILIARY MEDIA SHOULD PREFER AUXSTORE.

## 8. MEMORY-PRESSURE ORDER

THE TARGET CHEAPEST-FIRST RECLAIM ORDER IS:

```TEXT
1. DISCARD THE CLEAN BCACHE RAM L1 SLAB.
2. EVICT ELIGIBLE MEMFS DATA INTO BACKSTORE.
3. COMPACT CORE IF FRAGMENTATION, NOT FREE WORD COUNT, BLOCKS ALLOCATION.
4. SWAP AN ELIGIBLE PROCESS INTO BACKSTORE.
```

THERE IS NO DIRTY GENERIC CACHE RECLAIM CASE BECAUSE BCACHE NEVER HOLDS DIRTY
DATA.

## 9. EXPLICIT NON-GOALS

V0.9 BSTORE/BCACHE DOES NOT PROVIDE:

- VARIABLE-SIZE CACHE OBJECTS;
- BYTE/CHARACTER CACHE UNITS;
- GENERAL WRITE-BACK CACHING;
- AUTOMATIC CACHING OF EVERY BLOCK-DEVICE REQUEST;
- A PERSISTENT GENERAL-PURPOSE BLOCK ALLOCATOR;
- CACHE LRU CHAINS OR HASH TABLES;
- PROCESS SWAP THROUGH BCACHE;
- LOGSTORE THROUGH BCACHE;
- DYNAMIC SHARING OF CACHESTORE BLOCKS WITH AUTHORITATIVE BACKSTORE.

THE LAST RULE IS INTENTIONAL.  SHARING THE SAME PHYSICAL SLOTS WOULD REQUIRE
EXTRA VALIDITY/GENERATION OR INVALIDATION STATE TO PREVENT AUTHORITATIVE DATA
FROM COLLIDING WITH STALE CACHE TAGS.  FIXED BACKSTORE/CACHESTORE REGIONS ARE
SMALLER AND EASIER TO PROVE CORRECT.

## 10. IMPLEMENTATION PLAN

THE MIGRATION MUST BE CUMULATIVE AND KEEP A BUILDABLE TREE AT EACH COMMITTED
STAGE.

### STAGE 1 - FREEZE THE ABI AND MEDIA LAYOUT

1. DEFINE `BSTORE_BLOCK_WORDS` AS OCTAL 0200 AND ADD BUILD-TIME ASSERTIONS FOR
   D6FS, DTFS, DSK270, AND DRM236 BLOCK SIZE EQUALITY.
2. REPLACE THE IN-PROGRESS AUXCACHE MEDIA SEMANTICS WITH AUXSTORE REGIONS NAMED
   LOGSTORE, BACKSTORE, AND CACHESTORE.
3. DEFINE THE COMPACT AUXSTORE DESCRIPTOR AND VALIDATE RANGE BOUNDS/OVERLAP IN
   KINIT.
4. KEEP LEGACY D6FS/BLOCKSET RESERVATION DISCOVERY AS A FALLBACK ONLY.

ACCEPTANCE: DSK/D6FS, DRM/D6FS, AND DTC/TSFS BOOT STILL LINK; NO NEW PERMANENT
MEMORY ABOVE THE CONFIGURED LIMIT.

### STAGE 2 - GENERALIZE SWAP_STORE INTO BACKSTORE

1. RENAME/REWORK `SWAP_STORE` AS THE GENERIC ONE-BIT-PER-BLOCK ALLOCATOR.
2. REPLACE `SWAP_STORE_ALLOC_MEMFS()` WITH THE RESERVE PARAMETER.
3. ROUTE PROCESS SWAP THROUGH BACKSTORE WITHOUT CHANGING PROCESS SWAP RECORDS.
4. ROUTE MEMFS EVICTION/FAULT-IN THROUGH THE SAME BACKSTORE API.
5. RETAIN PERSISTENT MEMFS SNAPSHOT RESERVATION USING THE SAME BITMAP.

ACCEPTANCE: PROCESS SWAP AND VOLATILE/PERSISTENT MEMFS TESTS PASS; ONE ALLOCATOR
BITMAP EXISTS; NO SECOND MEMFS BACKING BITMAP EXISTS.

### STAGE 3 - GENERIC RAM BCACHE

1. EXTRACT THE CURRENT D6FS TWO-BLOCK CLEAN CACHE INTO `BCACHE`.
2. REDUCE THE SLAB TO THE MINIMUM GENERIC TAG+DATA SHAPE.
3. REPLACE D6FS `D6FS_CACHE_FETCH/STORE/RECLAIM` WITH BCACHE GET/PUT/RECLAIM.
4. KEEP D6FS WRITE-THROUGH ORDERING: AUTHORITATIVE WRITE BEFORE CACHE UPDATE.

ACCEPTANCE: D6FS CACHE TESTS PASS; D6FS PRIVATE CACHE STATE/CODE IS GONE;
DYNAMIC RAM IS NO LARGER THAN THE OLD OCTAL-0405 SLAB.

### STAGE 4 - DTFS/TSFS USE THE SAME L1

1. GIVE DTC MEDIA A COMPACT CACHE KEY WITH MEDIA GENERATION.
2. REMOVE THE SEPARATE DTC RAM-CACHE IDENTITY/ALLOCATION PATH.
3. CALL BCACHE AROUND THE EXISTING RAW DTC READ/WRITE SERVICES.
4. VERIFY NATIVE DTFS AND TSFS USE THE SAME PATH; TENEX/ITS ARE DISABLED.

ACCEPTANCE: DTC WITHOUT AUXSTORE USES ONLY THE COMMON TWO-BLOCK RAM L1 AND
REMAINS CORRECT ACROSS MEDIA CHANGES.

### STAGE 5 - CACHESTORE L2

1. ADD DIRECT-MAPPED CACHESTORE TAG/DATA ACCESS UNDER BCACHE.
2. ENABLE L2 ONLY FOR SOURCES WHOSE POLICY REQUESTS IT; DTC ENABLES IT WHEN
   AUXSTORE CACHESTORE EXISTS.
3. CLEAR CACHESTORE TAG BLOCKS AT BOOT.
4. KEEP DTC WRITE-THROUGH: TAPE SUCCESS FIRST, THEN BEST-EFFORT L1/L2 UPDATE.

ACCEPTANCE: DTC CACHE HIT/MISS/REPLACEMENT TESTS PASS; MEMFS/BACKSTORE CANNOT
ALIAS CACHESTORE; A DTC ROOT WORKS WITH AND WITHOUT AUXSTORE.

### STAGE 6 - REMOVE LEGACY DUPLICATION

1. DELETE OBSOLETE `AUXCACHE` PIN/VALID BITMAP CODE.
2. DELETE D6FS-SPECIFIC CACHE IMPLEMENTATION.
3. REMOVE SWAP-SPECIFIC ALLOCATOR NAMES AND SPECIAL MEMFS ALLOCATION ENTRY.
4. REMOVE DTC-SPECIFIC CACHE ALLOCATION/GEOMETRY STATE THAT BCACHE REPLACES.
5. AUDIT COMMENTS, SYMBOLS, AND TODO ENTRIES FOR STALE D6FS-OWNED SWAP/CACHE
   LANGUAGE.

ACCEPTANCE: ONE BACKSTORE ALLOCATOR, ONE BCACHE IMPLEMENTATION, ONE AUXSTORE
LAYOUT, AND NO DUPLICATE CACHE/BACKING BITMAPS REMAIN.

### STAGE 7 - TOOLS AND TESTS

1. DONE: ADD THE AUXSTORE FORMAT/INSPECTION TOOL TO DAIMOS `TOOLS/HOST`.
2. ADD HOST TESTS FOR DESCRIPTOR VALIDATION, BACKSTORE FIRST-FIT/RESERVE/FREE,
   CACHE KEYING, AND DIRECT-MAPPED CACHESTORE REPLACEMENT.
3. ADD TARGET/SIMH TESTS FOR PROCESS SWAP, MEMFS EVICTION, D6FS L1, DTFS/TSFS
   L1, AND DTC+AUXSTORE L2.
4. TEST DSK ROOT + AUX DRM, DRM ROOT, DTC ROOT + DRM AUX, DTC ROOT WITHOUT AUX,
   AND NO-BACKSTORE RAM-ONLY MEMFS.
5. RUN THE FULL PERMANENT-SIZE AND LOGIN/DSH ACCEPTANCE GATES.

ACCEPTANCE: ALL TESTS PASS AND THE PERMANENT ADDRESS LIMIT IS NOT EXCEEDED.

## 11. SIZE AND PERFORMANCE RULES

THE IMPLEMENTATION IS ACCEPTABLE ONLY IF IT CONSOLIDATES CODE OR PROVIDES A
MEASURED REQUIRED CAPABILITY WITHOUT VIOLATING THE PERMANENT-MEMORY LIMIT.

PRIORITIES ARE:

1. MINIMUM RESIDENT/PINNED RAM;
2. EXECUTION SPEED;
3. SIMPLE, PROVABLE STATE;
4. ON-MEDIA SPACE.

DO NOT ADD A GENERAL FRAMEWORK FOR A FUTURE DEVICE OR POLICY UNTIL A REAL
CONSUMER REQUIRES IT.
