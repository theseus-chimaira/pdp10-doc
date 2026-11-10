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
