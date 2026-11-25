# Poisoned D6FS administrative retirement, v1939

The existing root-only recovery extension (054 octal) retains its
action numbers: abandon FD (1), discard zero-based mount slot (2).
It now accepts explicitly poisoned noncritical D6FS as well as native
DTFS. Descriptor abandonment compares the descriptor provider to the
actual published mount provider before clearing both FILE words.
Healthy, read-suspect, unknown-state and unrelated provider volumes
are not administrative discard candidates. Neither the selected
namespace root nor an active SWAP/LOGSTORE owner can be discarded.

D6FS discard closes owner admission with its existing temporary
quiescence bit before acquiring the serialized provider lease.
An already held barrier cannot be borrowed. Private operation 033
(octal) verifies held barrier, sticky write uncertainty, actual
published root, provider and reader mount ID under the lease.
The normal D6FS release tail invalidates clean cache identities,
checks all owners again and frees the exact reader extent without
physical I/O. There is no selected-header proof, CLEAN publication,
media write or filesystem repair. A failed cache slab free retains
its MM owner while invalidating both tags; a failed reader free or
late owner check retains the mounted reader and identity. Failure
clears only the temporary barrier, preserving poison. Successful
reader release retains the barrier until VFS unpublishes the root
and target and clears the media state. The DTFS drain path remains
separate and verifies the provider again after waiting.

The administrator must quarantine abandoned media. A subsequent
independent MOUNT validates the medium; discard is not evidence
that the medium is repaired or safe to remount. All FILE/cwd/PIPE/
PURE owners must be gone. No new persistent data or syscall number.

Implementation 1c6384c5; testkit da09056. Native source-linked
v1927 gate passes administrative retirement and refusal/retry with
zero physical callbacks and real BCACHE reclaim. Seven new unsafe
controls and 17 earlier proof/unmount controls are rejected.
v1531 native FD/VFS oracle passes D6FS abandonment and stale-provider
refusal, with all four earlier DTFS controls rejected. D6FS context
selector, multimount, reader slab, remount and v1562/v1575 regressions
pass. MM and physical callbacks and lease-wait owner injection are
controlled, not full-guest PI6/physical acceptance. Other historical
runners not all rerun.

Folded lowmem KCC tape KCORE low end 025554 -> 025632 octal
(+46 words); DXR image 025133, BSS 000417 unchanged. D6FS MRES
2398 -> 2419 (+21), DTFS/TSFS 2988 and PURE 3879 unchanged.

K07 remains open for integrated physical/lifetime acceptance.
RAW PURE and mode3 remain disabled by default.
