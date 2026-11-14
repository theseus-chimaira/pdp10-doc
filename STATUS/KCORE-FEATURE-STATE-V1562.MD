# KCORE v1562: V2 DTFS transient unmount quiescence prerequisite

Date: 2026-11-14. Native source-linked acceptance PASS.
**P0-B durable writable DTFS remains OPEN.** DTVOL2 is READ-ONLY.

The existing VFS `vfs_unmount` performed an initial all-process owner
scan, then a possibly sleeping filesystem SYNC and optional PREPARE,
followed by a second owner scan and immediate retirement. A future
physical DTVOL2 CLEAN seal must occur after final ownership checks,
and its Type-551 WRITE/readback can sleep. Without an exclusive owner
admission barrier, new FILE/cwd and other owners could appear while
that final seal was being written.

v1562 adds the *transient, in-memory* per-mount quiescence bit
`0020000` to spare right-half bits of the existing MM-backed DTFS
media record. VFS acquires it after the initial checked owner scan
and before sleeping SYNC. The owner-specific release authenticates
the original published mount, clears only a barrier acquired by the
same unmount, and proceeds to retirement without further sleeping
calls. Failed SYNC and failed owner checks retain the mount and
release the transient flag without claiming a physical CLEAN write.

Fixed-VFS `vfs_dtfs_owner_admit_v1562` refuses to publish a new FILE
descriptor or cwd on a quiescing DTFS V2 mount; it also detects stale
DTFS vnodes whose public slot has been retired or reallocated. The
check is integrated immediately before the no-sleep stores in the
actual `file_new_fd` and `file_chdir`. Normal DTFS MRES dispatch
also checks admission, leaving its internal SYNC and unmounted
physical MOUNT_UNIT paths available as required. The linked v1561
physical intent gate independently rejects quiescing media before
any future DIRTY/WRITE. Other providers and legacy V1 semantics
remain unchanged.

## Verified native results

- New v1562 source-linked owner/barrier fixture: **41 native PDP-6
  scenarios PASS, seven unsafe source mutations rejected**.
- New v1562 full actual VFS unmount fixture: **12 source-linked
  PDP-6 scenarios PASS, three unsafe mutations rejected**. A modeled
  concurrent FILE publication attempts to enter during sleeping
  SYNC and is refused until the barrier is released. Physical I/O
  is simulated in this fixture.
- Historical v1514 nonroot failed-media unmount, v1523 remote
  FILE/cwd owner, v1560 retained-MM lifecycle, v1559 write path,
  v1555 read-only mount and write refusal, v1556 physical identity
  recovery, v1554 armed-gate recheck and v1561 mounted intent
  bridge native suites PASS after compatibility fixture updates.
- Full 256K KCC KCORE and optional DTFS MRES assembly/fold/link
  and image generation PASS. Fixed KCORE TEXT placement grows by
  **152 words** compared with v1561; no extra DATA/BSS or persistent
  mount table. Strict 32K size acceptance remains on HOLD.

This does **not** prove exclusivity against every possible PIPE,
swapped-process, inherited descriptor and in-flight provider admission.
It does **not** implement the final physical DIRTY-to-CLEAN seal,
independent readback, physical media generation/exchange checks or
actual Type-551 partial-write/reinsertion/cold-boot acceptance.
The v1555 unconditional V2 READ-ONLY guard remains in place. No
new writable V2 data/directory or RBN0 marker operation is enabled.

Next: complete cross-class owner admission and real scheduler race
tests; design and execute physically checked owner-safe CLEAN only
after its last owner scan and without prematurely releasing the
barrier; then establish removable tape generation safety and full
physical guest fault acceptance before permitting V2 RW.

P0-A CLOSED; **P0-B OPEN**; P1-C/D/E unchanged. VERSION 0.5.
No RAM-size optimization performed.

Source: `DAIMOS/design/kcore-dtfs-quiesce-v1562.md`.
Tests: `daimos-testkit/tests/validation/daimos-dtfs-quiesce-v1562.sh`
and `daimos-dtfs-quiesce-unmount-v1562.sh`.
