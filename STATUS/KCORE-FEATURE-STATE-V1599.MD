# KCORE P1-C v1599: two real PI6 owners at fatal physical swap READ

Date: 2026-11-15. Feature-completion acceptance using original
DAIMOS VERSION 0.5 and the full 128K PDP-6 KCC guest. The selected
hardware-simulator error and linked diagnostic words are **test-only
in a disposable KCORE copy**; the actual production scheduler,
swap/MM/BCACHE/BACKSTORE/driver and critical HALT branches are
unaltered. No permanent RAM growth, format, or ABI change.

The guest cold-boots four real Type-236 DR0..DR3 D6FSR2 members
as an interleaved D6FS root VOLUME with mapped BACKSTORE. It
executes genuine LOGSTORE readback and begins MEMFS snapshot
shutdown while four independent memory-heavy children own disk
swap records and sleep. The v1592 test-only timer observer retimes
two distinct actual SLEEP/DISK slots, **6** and **7**, to the next
original clock tick. Both are woken ONCE by the original PI6
timer service. No test queues RUN or invokes a swap transaction.

The unchanged slot-0 scheduler issues a physical mapped swap
READ for slot 7. On its first genuine Type-236 DR_DONE PI2,
a selected test-only error takes the original `drm236_pi_error`
path. The original BACKSTORE critical-failure gate reaches the
no-return reason-2 HALT, **PC octal `004011`**. Exactly one
selected READ and PI2, original DRM error 1, **zero** RESIDENT
publications. Original process swap ownership `01650` octal
and BACKSTORE allocation `01734` octal remain; pending snapshot
header is uncommitted. The selected original Type-236 DMA group
`0375060` maps DR1 physical block `017506` to original logical
BACKSTORE sector `000431`, with nonzero real backing payload.
All four D6FSR2 member descriptors retain their complete original
physical format; 268 other backing sectors hold data.

The independent original-symbol and raw-media verifier PASSed
the full guest and **23** false witnesses (eight new double PI6
owner/identity cases, fifteen inherited real DRM/PI2/media/fatal
cases). The original v1586 single-owner physical verifier and
all 15 mutations also still PASS.

This v1599 result means **two independently woken owners were
present at the real BACKSTORE fatal boundary**. It does not
mean there were two simultaneous DMA swap READs: one was selected
and the original mandated HALT terminated all execution.
The PI2 itself was genuine hardware-simulator completion, but
its error path was deliberately selected in disposable test
source; do not claim spontaneous magnetic failure.

Remaining P1-C: direct DEVICE-backed DRM AUXSTORE with an
independent DSK270 root and loaded swap, separate provider
write/dirty-fault isolation, repeated timing and final
current-source full system matrix. P1-D final source-matched
inventory and P1-E KA10 VM design still open. Strict 32K
optimization HOLD; fixed KCORE low end `024364` octal.

Accepted source/tests/reproduction details:
`DAIMOS/design/kcore-p1c-two-owner-fatal-v1599.md`,
`daimos-testkit/tests/validation/daimos-p1c-two-owner-fatal-physical-v1599.md`.
