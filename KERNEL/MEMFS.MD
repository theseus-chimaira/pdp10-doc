## 2026-11-10 v1401-v1404: MM-backed chunk ownership and release faults

The previous MEMFS private `memfs_data_free` leaf cleared the descriptor of
an emptied physical data chunk before invoking `mm_free`. A refused MM free
therefore left the vnode referring to an untracked but still allocated RAM
extent. The corrected leaf returns success/failure and clears the descriptor
**only after** the managed-memory release succeeds. TRUNCATE propagates the
refusal while preserving the previous file size/data. Callers discarding
speculative new extents must instead enter the fatal ownership invariant if
that cleanup cannot be completed: the new extent has no published vnode.

Within a shared 02000-word chunk, the sector bitmap now checks that every
bit in a requested release range is still owned. This rejects duplicate
sector frees rather than silently touching bookkeeping while other files
own neighbouring sectors. Releasing an extent absent from all chunk
descriptors also reports failure. No new per-sector or per-vnode metadata.

On eviction, the fresh BACKSTORE allocation is provisional until its write
and vnode publication succeed. If writing fails and `backstore_free` also
refuses to release the provisional run, normal eviction cannot safely
return because the allocated blocks have no persistent owner. The error
now reaches the existing fatal storage invariant instead of leaking disk
blocks. A dedicated source-linked PDP-6 fault test confirms the invariant
marker is set before HALT, in addition to ordinary MM-free and double-free
tests (previous failures at octal 0260 and 0262).

The boot image Makefile also omitted the raw `memfs_data_pdp6.s` and
`memfs_snapshot_pdp6.s` input dependencies. As a result, a direct MRES
target could use stale generated assembler despite changes in these
sources. Both inputs are now explicit dependencies; the target was verified
to regenerate the kernel assembler and relink correctly. After this fix,
the actual MEMFS MRES package measures **1771 resident words** (1758
initialized + 13 BSS), up from 1752 at v1400. The new code is movable;
fixed KCORE has not grown. Normal/fault PDP-6 and v216 SIMH/AAP restricted
user XIP lifecycle regressions PASS.

The 32K/48K/256K booted pressure matrix, actual swapless PURE scheduler
activation, and general production XIP RUN/EXEC remain open. D6 codec
integration continues to be deferred.

## 2026-11-10 v1399/v1400: backing ownership and failure-atomic teardown

`memfs_backing_drop` no longer clears the sole vnode first,,block-count
reference before the shared allocator has released its complete owned run.
It reports 0 on success (including no backing) and -1 on refusal. The real
MEMFS WRITE path checks that result before copying user data, so a failed
release cannot leave an untracked allocation or overwrite a file while its
old backing is still authoritative. TRUNCATE-to-zero and same-sector growth
now release the stale backing owner before changing resident data/geometry.
Larger relocation growth keeps the original owner intact until release is
confirmed, discarding only the candidate destination on refusal. Tight-core
spill is handled separately: the original containing chunk was already
evicted, so a shared commit path must not attempt to free its stale base.

Unmount cannot free one evicted file and subsequently discover that another
file's BACKSTORE record is invalid. `memfs_data_destroy` now first checks
all 64 backing records using the same read-only complete-run verifier as
the swap subsystem. An invalid later record rejects teardown **before any
earlier owner is released**. A failure returns -1 through the real MRES
PREPARE_UNMOUNT operation, leaving the mount intact and retryable. Once all
ownership has passed preflight, a late unexpected MM or bitmap refusal is
treated as a fatal ownership invariant, not a recoverable partial unmount.
The existing node/backing table is reused, with no new persistent metadata.

The source-linked PDP-6 regression failed against the pre-fix production
code at WRITE (octal 0240) and at teardown (0251), then passed after the
fixes. It checks refused releases during WRITE, same-sector and relocated
growth, TRUNCATE, direct unmount preparation and the MRES dispatcher,
including an invalid *later* owner that must not destroy an earlier one.
The original SIMH and AAP v216 XIP user-image execution and VM teardown
both still PASS. Production MEMFS MRES rises from **1719 to 1752 resident
words (+33)**, unchanged 13 BSS words; fixed KCORE is unchanged. This is a
validated safety correction, not net memory savings.

Still open: propagation of failures from MEMFS's internal sector/MM free
primitive, failed eviction-write cleanup, fuller low-core spill fault
injection and booted 32K/48K/256K pressure tests. The restricted XIP
admission profile has not yet been connected to general RUN/EXEC. The
grouped D6 codec remains deferred, with production D6LZ36 unchanged.

## 2026-11-10 v1396: pinned XIP metadata is immutable during execution

The production movable MEMFS provider now checks its existing four-entry
XIP pin reference table before any operation through `memfs_chmod`. This
also covers the same dispatch entry's CHOWN and UTIME branches. A pinned
executable must retain its file mode, ownership and timestamp as well as its
code words; restoring writable or changing provenance while a process still
fetches that memory is forbidden. On final unpin, ordinary metadata edits
resume. The check adds no reference table, filesystem metadata word, or
fixed KCORE code. The rebuilt MEMFS MRES rises from **1712 to 1719 resident
words**, with no additional DATA/BSS.

Native PDP-6 tests execute the real provider, proving that CHMOD/CHOWN/
UTIME are rejected during a pin and accepted after release, with unchanged
metadata during refusal. Existing v216 SIMH and AAP user-mode XIP fetch,
monitor UUO and production VM teardown PASS. The test harness's former
`proc_swap_detach` helper returned an invalid nonzero success status and
lacked `wall_time36`; both test-only defects have been corrected.

**Not yet complete:** the strict v1394 no-write instruction profile is
staged but not connected to a trusted source-vnode admission transaction or
general production RUN/EXEC, scheduler, BRK and swap exclusions. The PDP-6
does not provide hardware text write protection. A mutable MEMFS file with
a PURE header is not sufficient authority to execute arbitrary instructions
from shared physical memory.

## v1094: XIP release failure must retain its VM owner

VM teardown distinguishes an unloaded MEMFS MRES entry from a loaded
provider that *owns* the physical XIP extent and refuses to unpin it.
Previously both conditions could return -1, allowing the VM destroy
caller to fall through to a process-type MM free even for an owned
MEMFS extent. The owned XIP pin-release error now returns -4; status 1
still means a successful XIP release, status 0 a non-XIP extent, and
-1 is reserved for the missing-provider trampoline. Only 0 and -1
permit an MM_TYPE_PROCESS fallback.

An owned negative XIP result prevents MM_TYPE_PROCESS free, swap-bitmap
detach and `VM_SPACE_NONE` publication. The original physical XIP ref,
stable process/u-area and PREVM retry owner remain intact. There is no
new MRES vector or permanent reference table. Native PDP-6 v1094 tests
exercise failed owned unpin and repair, normal process fallback,
absent-provider fallback, MM -4/-5, failed swap detach and retry, plus
four unsafe mutations. v1086 slot-0 PI6, v880 swap-detach and v1081
finalizer tests PASS. The matched KCC full-image endpoint decreases
five resident words across the combined v1093/v1094 release changes.
P0-D remains open for integrated multi-owner fault stages, and the
production OOM requester-kill option stays OFF.

# MEMFS FILE WRITE AND O_APPEND SEMANTICS

MEMFS IS THE VOLATILE FILESYSTEM USED BY `/TEMP` IN THE STANDARD DAIMOS
CONFIGURATION. DATA WORDS ARE HELD IN DEMAND-ALLOCATED KERNEL MEMORY;
A MEMFS FILE MAY GROW OR RELOCATE ITS STORAGE DURING A WRITE.

## PER-DESCRIPTOR O_APPEND

A REGULAR-FILE DESCRIPTOR OPENED WITH `O_APPEND` RETAINS APPEND INTENT IN
ITS PACKED TWO-WORD DESCRIPTOR. THE EXISTING DIRECTORY METADATA BIT ALSO
MEANS APPEND FOR A REGULAR FILE (WHICH HAS NONZERO REGULAR/LOCK STATE).
DIRECTORY TESTS MUST DISTINGUISH THE TWO DISJOINT CASES.

`WRITE_WORDS` FOR A MEMFS APPEND DESCRIPTOR PASSES OFFSET `-1` TO THE
MEMFS PROVIDER. AFTER VALIDATING THE FILE SLOT AND ITS WRITABLE REGULAR
FILE TYPE, `memfs_write_words` RESOLVES THE SENTINEL TO THE CURRENT SLOT
WORD COUNT. THE NORMAL GROWTH AND WRITE PATH THEN USES THIS OFFSET, NOT
THE FILE SIZE CAPTURED WHEN THE DESCRIPTOR WAS OPENED. THE DESCRIPTOR'S
SEEK CURSOR IS REFRESHED FROM FILE STAT AFTER A SUCCESSFUL APPEND.

THIS DOES NOT ADD FILE-DESCRIPTOR WORDS OR A NEW PER-FILE LOCK TABLE.
MEMFS PROVIDER CALLS FOLLOW THE EXISTING RESIDENT KERNEL DISPATCH PATH.
THE D6FS PROVIDER USES ITS SEPARATE SERIALIZED DISPATCH PATH.

THE DAIMOS-TESTKIT NATIVE PDP-6 TEST OPENS TWO MEMFS APPEND DESCRIPTORS
BEFORE EITHER WRITES, SEEKS ONE TO ZERO, APPENDS THROUGH BOTH AND A
DUPLICATE, THEN CHECKS FOUR RAW WORDS AND ALL CURSOR POSITIONS. THE
CORRECTED KERNEL PASSES; THE ORIGINAL OPEN-TIME OFFSET IMPLEMENTATION
FAILS AT THE SECOND CURSOR. THE D6FS TWO-OPEN REGRESSION ALSO PASSES.

THIS TEST DOES NOT COVER SIMULTANEOUSLY SCHEDULED WRITERS, DEVICE FAILURE,
OR PARTIAL S6REC RECORD WRITES. THE HISTORY FILE IS PERSISTED ON D6FS
WHEN ITS PATH POINTS TO THAT FILESYSTEM; MEMFS ITSELF IS VOLATILE.
