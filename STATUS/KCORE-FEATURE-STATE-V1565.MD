# KCORE v1565: actual PDP-6 scheduler overlap against DTFS V2 seal

Date: 2026-11-14. **Full physical guest PASS; production code unchanged.**

## Tested boundary

v1562 protects final DTFS V2 unmount with a per-mount admission bit,
acquired between the first owner scan and sleeping provider SYNC.
v1563 closes staged RUN/EXEC PURE executable owner publication.
v1564 verified physical DTVOL2/CLEAN read-only mount/unmount and
physical WRITE_INTENT rejection on two real Type-551 PDP-6 boots.

v1565 tests the missing independently scheduled owner crossover.
The testkit copies a complete source-matched KCC 256K physical
TSFS-root image to an isolated directory, then inserts a *test-only*
768-tick scheduler sleep into the copied VFS assembly immediately
after production `vfs_dtfs_quiesce_begin_v1562` acquires its seal
and before original `vfs_sync`. The normal KCORE build/link was
rerun and the versioned injection label verified in the linked map.
No production VFS, FILE, PIPE, DTFS, scheduler or physical-media
source in DAIMOS was edited.

## Actual two-process witness

With a native stamped DTVOL2/CLEAN Type-551 DTC3 mounted RO,
PID1 starts PID2 using real RUN and two anonymous pipes.
PID1 then enters UNMOUNT. PID2 witnesses the STILL PUBLISHED
but quiescent slot: actual `dsys_fsinfo(1)` returns -1 because
the provider SPACE operation is refused by the V2 barrier.
The pre-seal healthy mount returns +1; after retirement the slot
returns 0. A missed barrier cannot pass the guest oracle.

PID2 attempts real OPEN and CHDIR of `/MOUNT/DT0`; both refuse,
and GETCWD remains `/`. The child reports the results on its
pipe while PID1 completes ordinary SYNC, owner/MM checks and
retirement. After retirement the original TSFS directory at
`/MOUNT/DT0` becomes accessible again, and the DTFS slot remains
empty; this is correct namespace behaviour, not a stale DTFS FD.
PID1 waits for clean child exit. The independent whole-tape
SHA-256 remains unchanged.

Post-v1565 regression reruns PASS for the real native production
guards: v1562 quiescence 41 cases/seven unsafe mutations rejected;
v1562 complete unmount 12/three; v1563 PURE owner scans 26/six;
v1563 linked KCC attach and VFS gate 11/two.

Full CTY acceptance includes `DTFSRACE V1565 SEAL_OBSERVED`,
`OWNERS_REFUSED`, `RETIRED`, `RACE_PROVEN`, `RETIRED_TSFS`, `PASS`.

The initial test oracle incorrectly demanded failure to access
the underlying TSFS directory after unmount. It failed only
that assertion. The corrected, source-grounded oracle passed
the real PDP-6 test, with no production code workaround.

## Remaining P0-B

The forced scheduler pause happens before physical SYNC, not
inside an actual overlapping Type-551 I/O transaction. This
stage does not prove media generation/exchange, exclusive
physical DIRTY-to-CLEAN write/readback, partial physical DATA
failure, reinsertion or post-RW cold-boot recovery. The v1555
DTVOL2 unconditional read-only guard stays in place.

**P0-A COMPLETE, P0-B OPEN, P1-C/D/E unchanged.**
No fixed production KCORE/RAM size change. VERSION 0.5;
strict 32K optimization HOLD.

Source: `DAIMOS/design/kcore-dtfs-owner-race-v1565.md`.
Tests: `daimos-testkit/tests/validation/daimos-dtfs-v2-race-v1565.md`.
