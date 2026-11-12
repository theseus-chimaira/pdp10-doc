# DAIMOS KCORE feature state, v1508

Date: 2026-11-12. Incremental update to
`STATUS/KCORE-FEATURE-STATE-V1507.MD`; production KCORE remains
v1507 and has no further process state, resident data or ABI growth.

## Physical persistent-refusal boundary: scoped PASS

Testkit `25a7993` v1508 ran a disposable 256K KCC PDP-6 with
source-linked instruments and read-only physical SIMH snapshots.
After an actual successful swap-in payload read, one selected
ordinary DISK owner (slot `010` octal) had checked BACKSTORE free
continuously refused. The real bitmap was left intact.

At physical fault-on samples 19, 20 and 21, checked refusals were
1, 4 and 7 with clock values 36599, 38542 and 40484 decimal.
The exact original disk run `02350`/`0330` remained authoritative;
u-area was stable, retry stamp and pending TERM persisted. Owner
deferred TERM completion, private close, finalization, zombie and
repair counters remained zero, as expected for a permanently
unavailable backing owner.

The strict v1508 physical verifier PASSed. Its separate self-test
rejected ten unsafe owner/bitmap/scheduling/finalization witness
mutations. The earlier v1507 native two-owner, normal/realtime,
healthy bystander, exact retry deadline and repaired-backstore
tests all remain PASS. Source/test evidence including SHA256 is
listed in DAIMOS
`design/kcore-backstore-persistent-physical-v1508.md`.

The captured guest **intentionally stopped after diagnostic
sampling**: do not claim a normal HALT, all-child WAIT acceptance,
true physical DEVICE fault, recovery of lost media, or eventual
resource reclamation. No independently healthy swapped job was
shown completing between the three fault-on samples, so physical
bystander fairness remains open even though the native v1507
scheduler gate passed.

## Remaining KCORE work

1. Physical bystander/swap fairness under a persistent fault, and
   verified behavior for genuinely irreparable BACKSTORE media or
   DEVICE loss. Retain an unrecoverable owner safely rather than
   dropping its backing, u-area or private executive continuation.
2. Mixed DEVICE/VOLUME, BCACHE/DTFS tape-root, LOGSTORE/MEMFS and
   provider failure/recovery integration.
3. F1 uncompressed swapless PURE-text drop/reclaim, immutable
   executable backing, mutable suffix, MM and complete APR restore.
4. F3 dynamic FSINFO enumeration and physical 1/4/8/26 mount
   acceptance; v1492 is a separate local in-progress change.
5. Optional compression after measured positive PURE RAM results;
   remaining KINIT/clock/device gates and then final KCORE sizing.

The previous repaired physical v1506 owner-return/TERM completion
remains accepted. VERSION 0.5, 32K HOLD, no MRES unload/reload
claim and no 0.9 release declaration.
