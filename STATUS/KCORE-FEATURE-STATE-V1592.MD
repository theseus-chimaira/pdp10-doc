# KCORE P1-C v1592: simultaneous real PI6 wake with isolated swap rollback

Date: 2026-11-15. Two full 128K KCC PDP-6 cold boots on original
Type-236 four-member DR0..DR3 D6FSR2 root VOLUME, mapped
BACKSTORE, LOGSTORE and persistent MEMFS v2/XIP. This v1592
source-linked testkit-only exercise advances the v1585 physical
one-owner timer test without adding permanent scheduler logic.

Two DIFFERENT original SLEEP, timer-tagged, VM-on-DISK process
descriptors with real backing records are retimed to the next
original PI6 tick inside the snapshot. Original timer service
physically wakes slots **6 and 7** once each, without calling a
test-only swap service or faking controller interrupt completion.
Original slot-0 executes five independent physical swap READs and
obtains 1,120 matching Type-236 PI2 DR_DONE events per boot.

One test-controlled checked BACKSTORE release refusal on slot 6
occurs after real I/O and MM unpin. The original cleanup preserves
DISK backing, original owner records, all bitmap words, BCACHE
identity and transition semantics; one TERM event occurs.
Four other original swap-ins publish RESIDENT successfully.
Both original HALTs normal, second cold boot restores the first
boot's persistent MEMFS/XIP and LOGSTORE. Raw media physical
snapshot header valid on DR3 `017777`, 3,648-word pool, 277
other nonzero backing sectors and original D6FSR2 geometry.
The independent strict verifier rejects 16 false v1592 owner/
PI6/MM/bitmap/TERM witnesses plus 13 inherited v1584 physical/
PI/mapper/snapshot mutations, for **29** rejected unsafe cases.

**Scope:** v1592 proves genuine timer wakeups of TWO distinct owners
and one owner-specific recoverable rollback with actual aggregate
other successful swap completions. It does **not** independently
observe that slot 7 itself published RESIDENT; its four successes
are a total, not per-slot identities. That per-slot publication,
mixed DEVICE/VOLUME/DTFS physical fault timing and isolated
loaded-simulator timeout stability remain open. Controlled refusal
must never be equated with recoverable owned-media controller I/O,
which retains the critical fail-stop policy.

Detailed source-linked report:
`DAIMOS/design/kcore-p1c-two-owner-physical-v1592.md` and
`daimos-testkit/tests/validation/daimos-p1c-two-owner-physical-v1592.md`.

P1-D v1591 above-seven-slot TSFS inventory already accepted;
P1-E implementation is ON HOLD pending KA10-aware architecture.
No production RAM, KCORE text, ABI or media-format changes;
VERSION 0.5, low end `024364` octal, 32K/optimization HOLD.
