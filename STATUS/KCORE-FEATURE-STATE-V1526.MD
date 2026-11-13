# KCORE features v1526: physical cross-process DTFS mount lifetime

2026-11-13. Built and tested against the unchanged
full 256K v1525 PDP-6 KCC tape-root KCORE and
installed DTFS/TSFS optional MRES. This is
**feature acceptance**, with no optimization,
module policy or new permanent RAM allocation.

## Newly accepted: another process's real cwd and FILE

The new v1526 daimos-testkit gate compiles and
launches a genuine two-process PDP-6 workload.
The PID1 test user process mounts stamped native
DTC3 at `/MOUNT/DT0`, validates FSINFO provider
5 and RUNs a distinct child process from the
same actual physical TSFS root. Two kernel
pipes carry readiness/control tokens; all
important ownership phases are synchronized,
not inferred from sleep or emulator timings.

- Child alone in DTFS cwd: parent's UNMOUNT fails;
  original DTFS remains published.
- Child has left cwd and owns only an open DTFS
  directory FILE: parent's UNMOUNT still fails;
  original DTFS remains published.
- Child has closed FILE, sent a release ACK
  and is **still alive/non-zombie** per actual
  PROCINFO: parent's UNMOUNT succeeds and
  FSINFO now reports the slot empty.
- Parent releases blocked child; real WAIT
  verifies normal child exit status zero.

Both real guest runs PASS, including the final
strict PID and PROCINFO ownership checks.
The parent never holds its own DTFS cwd or
FILE during the critical refusal assertions.

Detailed implementation and test evidence:
`DAIMOS/design/kcore-dtfs-cross-process-v1526.md`
and `daimos-testkit/tests/validation/
daimos-dtfs-cross-owner-v1526.md`.

The fixed KCORE and DTFS/TSFS optional MRES
are unchanged. The v1525 controlled optional
MRES size remains octal `003746` text,
`000002` BSS. VERSION is still 0.5; strict
32K placement gate HOLD.

## Remaining feature-only KCORE gates

**P0-A:** Prove authoritative physical
permanent loss of actively selected root
TSFS units or BACKSTORE and route only
that positive evidence to v1515 fatal HALT.
Ordinary select, timeout and PI errors do
not establish permanent physical loss.

**P0-B:** Actual noncritical DTC3 read
errors while a mounted FILE/vnode is
live; matching 72-bit volume return and
changed high/low identity rejection
after physical reattachment, preserving
safe ownership and refusing stale
cache/FSINFO/SYNC. Actual failed/partial
physical write, sticky WRITE_UNCERTAIN
and explicit administrative recovery.
Cross-process cwd/FILE retirement no longer
blocks this work.

**P1:** Mixed-provider faults, dynamic
FSINFO inventory 1/4/8/26 including
errors/holes, swapless uncompressed PURE
with MM/APR lifecycle, final feature
freeze. No optimization yet; strict
32K gate HOLD.
