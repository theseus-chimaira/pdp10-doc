# Transient executable backing reservation, v1933

RUN and EXEC reserve the looked-up vnode before STAT, execute permission,
header reads, VM allocation, payload loading and final attachment. The
private provider-gateway operation 032 (octal) drains an earlier physical
operation before publishing the dependency under the existing lease.
MEMFS uses its non-sleeping direct path. The reservation validates the
published provider/public mount and the existing DTFS/D6FS quiescence
admission guard. No extra provider operation is dispatched.

A two-word stack-owned record (next, vnode) and one resident list head
represent active loads. Both file mutation and mount-owner scans include
these records before scanning permanent process backing. Release removes
exactly the caller's record in arbitrary list order; missing published
records are invariant failures. The common loader return preserves its
result and saved ACs while releasing the reservation. No new process or
mount table words. The v1932 post-lease mutation checks recognize active
load ownership without a separate lock.

Implementation 04aefcb1, testkit d595c78. Native source-linked reservation,
lease, quiescence, mutation, scanner and plain DXR1 loader gates pass.
Eight loader cases and six rejected unsafe controls cover lease drain,
file/mount protection, quiescence refusal, non-LIFO removal and release.
Earlier v961/v962/v966/v1563, v327/v917/v1078/v1088, v466, v1932 and
v1649 regressions pass. Some other historical extraction runners were
adapted but not rerun. Native I/O, allocation and attachment destinations
are controlled rather than complete physical guest operations.

Folded KCC lowmem tape KCORE low end 025256 -> 025411 octal (+91 words),
BSS +1 word. D6FS 2398, DTFS/TSFS 2988 and PURE 3879 resident words
unchanged. The loader frame grows by two words, with additional temporary
call stack usage. No process or mount slab enlargement.

The reservation spans the loader, not the entire staged EXEC transaction.
Old/new backing identity must remain visible through startup, record
replacement, previous VM destruction and rollback. Integrated namespace
lookup-to-reservation, fatal/private-stack lifetime, compressed/raw-XIP
and physical PI6 acceptance remain open. RAW PURE and mode3 default OFF.
