# Physical PDP-6 K07 executable reservation / retirement acceptance, v1944

2026-11-26. Scoped physical PASS for both owner-admission orderings
with current production v1939 source and two disposable test-only
scheduler sleeps. No production source or permanent RAM growth.

A genuine 48K TSFS guest mounts independent writable stamped
DTVOL2 on DTC3 and copies the compressed PURE executable through
ordinary filesystem syscalls: eight writes, descriptor closure,
chmod, CLEAN unmount and remount. The actual seed reel is verified
independently: unchanged 72-bit identity and valid CLEAN seal,
native directory and block owner chain, exact 999-word executable.
The first replay stopped at E because SIMH's flush diagnostic
intervened between genuine guest marker and monitor label. The
corrected monitor accepts only that specific diagnostic, not
interrupted or missing guest markers. The second guest reuses the
actual verified seed reel, mounts/stats the same file and retains
the original compressed child. Disposable kernel images identical.

## Reservation first

The original executable loader acquires its transient reservation,
releases the physical provider lease, checks namespace revision,
then the disposable test sleeps before STAT. A real independently
root-backed helper calls UNMOUNT. Physical snapshot at its refusal:
current slot2, parent slot1 SLEEP, parent u-area050251, reservation
050364 with next0 and vnode050202000000 (octal), wholly within
parent executive stack. No permanent secondary backing owner.
Lease777777777777 is free (RH nonzero). Original root and media
0100003, namespace epoch2 and MM12 unchanged; PI6 delta1476
decimal ticks. The original transient mount-owner scanner protects
the backing even before attachment. Parent RUN later succeeds,
publishing slot3 secondary backing and releasing the reservation.
Both exact WAITs succeed, MM8, no child VM/u-area or reservation.

## Retirement first

A real unmount helper obtains the original V2 owner-admission
barrier and sleeps before SYNC. Parent observes FSINFO refusal,
attempts genuine compressed PURE RUN, and again observes FSINFO
refusal. Physical Q/R snapshots retain original published root,
media0120003 (V2 + quiescing, unit3), epoch2, exact backing records
and MM12. Helper sleeps in slot2. No new permanent owner or
reservation. Thus refusal is on a still-live protected mount, not
merely a mount already absent from the namespace.

Actual helper UNMOUNT subsequently retires root (epoch3),
quiescence cleared, PI6 delta2504 decimal ticks. Ordinary DTFS
retirement leaves the unused unarmed media tag0100003, unlike
D6FS's explicit zeroing. Root is the publication authority, and a
fresh mount authenticates RBN0 and replaces the media tag.
Same-medium mount, actual compressed RUN, data sentinel and exact
WAIT pass. Final unmount and parent exit: root0, MM8, no child
VM/u-area or reservation, overall PI6 delta20583 decimal ticks.
Exactly one normal HALT PC000000, simulator exit0. Final DTC3
byte-identical to original genuinely seeded CLEAN input.

Testkit 3b57c03, 8012789, 71f29f4. Actual 12-stage replay passes.
Ten altered physical transcript controls and six synthetic native
media corruption controls rejected. These validate the verifiers,
not executed guest faults. Lease predicate and retired-media
predicate corrected according to original production implementation.
The test uses real scheduler/physical I/O with deliberately widened
race windows; it is not a measurement of natural race probability.

Accepted trace SHA256:
6b5691b66f10a81b058a5c6b3b662f27e529f4f8a0bda82bbee0a877833538fd
Physical CLEAN reel SHA256:
44e616a50abae9914359552e9cf55152f6250c9ae50dc526d96091d8e260593e
Testkit documentation:
tests/validation/pure-k07-race-v1943/README-v1944.md.

K07 physical pressure/FULL/WAIT baseline v1942 and physical
admission ordering v1944 pass. Integrated source loss/replacement
and recovery are still open. No inference of fault recovery from
an intact-media race. RAW PURE and mode3 remain default OFF.
