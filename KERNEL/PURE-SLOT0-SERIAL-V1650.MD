# P1-E v1650: original PDP-6 PI6 serialization during FULL

2026-11-17. Native source-linked instruction-level PASS, correlated
original v1643 accepted 48K physical trace PASS. No production change.

The original swapless PURE restoration runs from the permanent
slot-0 `proc_swap_service_one` service. The PDP-6 PI6 clock
path switches from slot 0 only when the recorded interrupt
PC lies within `[proc_idle_loop, proc_idle_loop_end)`. A pending
swap request in `proc_sched_cursor & 0400` also excludes a
normal idle-loop dispatch. The software PI6 reschedule path
uses the same saved-PC interval and swap ownership condition.

The original linked 48K bounded FULL MRES loader is at
**035252 octal**. The switchable idle interval is
**017216..017264 octal**: FULL is not in it. At the original
physical FULL_PINNED checkpoint `proc_current_slot` is 0.
Slot 3 owns both the original pin1 PACKED writable suffix
`111200..120000` and a separate pin1 full user APR
`120000..130000` (all octal).

The new testkit fixture executes the original extracted
scheduler instructions on native PDP-6 in eleven cases:
permitted idle scheduling, forbidden MRES-PC scheduling,
due/non-due timer, lower/upper PC boundaries and swap
ownership. Three deliberately broken gate variants
execute and are rejected. A separate auditor requires the
original v1643 source-linked physical trace/map identities
and refuses two manipulated traces (false current slot;
missing RAW source pin). The full v1643 read failure,
rollback, later successful restore and WAIT audit also
passes again. The physical trace is reused; v1650 is NOT
a new physical simulation.

**Conclusion:** an independent user-space DTFS UNMOUNT
cannot execute concurrently with the slot-0 FULL MRES
continuation using the original PDP-6 scheduler. Forcing
an unmount at that point by bypassing PI6 would fabricate
an invalid interleaving. The original mount-owner guard
and V2 quiescence remain separately verified by v1649
and v1647.

DAIMOS report:
`design/kcore-p1e-slot0-serialization-v1650.md`.
Testkit:
`tests/validation/pure-slot0-serial-v1650/`,
branch `test-kcore-p1e-slot0-serial-v1650-20261117`,
commit `39febe6`.

P1-E remains OPEN for real device/media failure,
uncertain MM pin ownership, complete transient peak
RAM/BCACHE in 32K/48K/64K and distinct KA10 VM.
PURE remains opt-in/OFF; strict 32K optimization HOLD.
