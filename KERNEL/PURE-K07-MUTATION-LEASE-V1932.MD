# PURE mutation admission after provider wait, v1932

WRITE_WORDS and TRUNCATE keep their early VFS RO and backing-owner
checks. For serialized physical providers, the existing request frame
now remains live through an additional check after lease acquisition.
The original vnode is checked against current RO policy and the PURE
backing scanner. Successful admission restores the original seven
request registers. Refusal returns through the ordinary lease unlock
and wakeup path without executing the provider operation. MEMFS
retains its nonserialized early VFS check.

DTFS and D6FS unlink now also check the exact child resolved within
the provider, rather than relying only on the earlier pathname lookup
in file_remove. The inspected rename implementations refuse existing
destinations; they do not replace an executable vnode.

Implementation 6810c892, testkit 75e3dd7. Native source-linked tests
use the actual provider gateway, RO leaf, backing scanner and D6FS
unlink. Sixteen delayed WRITE/TRUNCATE cases and two D6FS child cases
pass; four independently executed unsafe variants are rejected.
DTFS unlink native/TENEX protected-child cases and the previous lease,
contention, backing guard and rename gateway regressions pass. Physical
provider destinations and lease wait completion are controlled.

Folded KCC lowmem tape fixed KCORE low end 025241 -> 025256 octal
(+13 words), D6FS MRES 2392 -> 2398 (+6), DTFS/TSFS MRES 2985 -> 2988
(+3), PURE MRES 3879 unchanged. No new permanent or per-process data.

The fix closes delayed dispatch admission, not the complete lifetime
race. A whole-load RUN/EXEC reservation and protection against owner
publication during an already dispatched mutation are still necessary.
Direct MEMFS mutation admission, poisoned administrative retirement
and full-guest physical/PI6 acceptance also remain K07 work.
RAW PURE and mode3 remain default OFF.
