# PDP-6 RAW PURE deferred ownership recovery (v1919)

Existing u-area word 067 stores the original sector-aligned RAW suffix
and transaction kind (bits 29/30: DROP/PREFIX; neither: FULL). In-place
transactions require zero scratch target. The old idle retry armer and
slot-zero tagged service ABI are unchanged. Arming can occur on the
noncurrent victim's reclaiming caller; reconciliation runs on slot zero.

With PI excluded, the service checks the original escrow, process state,
backing, storage mode, alignment, descriptor capacity and nonoverlap.
DROP can resume an exact unpinned original resident or reconcile the
exact committed PACKED suffix. PREFIX with an exact unpinned full-span
PACKED or unpublished PROCESS rolls back to RAW using a newly acquired
pin. Lost successful pin/release/unpin replies require independent exact
owner proof. The service never assumes ownership of a pre-existing pin
or frees the authoritative suffix. Uncertain owners remain held, and
READY_ONCE is issued only by a later ordinary successful restoration.
FULL continues its original exact scratch rollback contract.

Native v1919 (daimos-testkit 1214632) passes three DROP, four PREFIX and
six invalid marker/shape cases. Five unsafe mutations fail at 0101,
0403, 0412, 01004 and 0607 octal. Native v1918, v1612, v1902 and v1906
also pass. The real MM rebase/claim/release/commit assembly is linked;
pin/unpin, allocation, PI, backing loader and retry armer are fixtures.
Existing injected pins must be retired lawfully before reconciliation.

Folded KCC lowmem tape PURE MRES grows from 3369 to 3556 resident
36-bit words (+187), BSS zero, fixed KCORE low end 025133 unchanged.
No additional permanent or per-process storage. This does not claim
physical PI6 or production acceptance. Terminal cleanup and final MM
geometry review remain K06 work. RAW PURE and mode3 stay default OFF.
