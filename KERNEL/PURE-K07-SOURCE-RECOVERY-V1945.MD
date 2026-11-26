## 2026-11-26 v1947: integrated K07/K08 physical acceptance

K07 executable backing and K08 scheduler implementation are closed.
The full uninstrumented 48K PDP-6 TSFS/PURE replay completed in
284.728 seconds, with the same production images as v1945. At the
actual FULL entry, slot3 PACKED113600..120000 and PROCESS122000..130000
each have pin1. Real PCLK PI6 interrupts TAPE PC032060 without
changing either MM owner or any of the 2176 retained RAW words.
Actual DTC3 detach during tape search reaches the original 7200-tick
timeout, and FULL returns -4 with source pin0, no destination,
original backing and volume ID intact. The subsequent real PI6 and
slot4 execution while DTC3 is absent establish independent progress.
The foreign CLEAN DTVOL2 medium fails a new PREFIX restore at
PC034446/-4, 302 ticks after the missing-source FULL failure.
The original medium then permits READY_ONCE publication for slot5
and slot3. Slot3's actual service return publishes cursor01003,
sole unpinned PROCESS122000..130000 and stable u-area051274;
the original scheduler consumes the hint, selecting AC1=3/cursor3
without changing MM13. Exact five child data/EXIT/WAIT witnesses,
parent data, final UNMOUNT, MM8, high slot2, mode word0, healthy
retired medium0100003, root0, and normal HALT PC0 all pass.
Both reels are unchanged. Independent full and supplementary
oracles reject 14 altered observed traces (not additional guest
fault injections). The v1945 same-PC repeated-stop witness remains
withdrawn; the complete later v1947 replay replaces it. No code,
image, ABI or resident RAM change. PI6 observed inside FULL and
after rollback, not inside PI-disabled sections. Wider physical
root/controller matrices and KCC production acceptance remain K09,
KA10 remains K17. RAW PURE/mode3 remain OFF by default.

## 2026-11-26 v1946 acceptance correction

K07 code complete, integrated foreign-to-original recovery/WAIT
replay pending. The earlier v1945 same-PC foreign checkpoint
was invalid: actual SIMH repeats a cleared/rearmed breakpoint
without executing the stopped instruction. Genuine missing-medium
rollback and original-medium recovery/WAIT remain valid.
Independent v1946 physical verification establishes real pinned
FULL PI6 at TAPE PC032067, actual 7200-tick device-search timeout
and FULL -4, another child's execution with DTC3 absent, and
a new foreign PREFIX -4 299 ticks after the missing refusal.
All 2176 retained RAW words and original identity remain intact.
The complete v1946 run exceeded its 260-second diagnostic deadline
during recovery; no slot3 READY_ONCE dispatch or final WAIT claimed.
No production code or RAM change. The v1945 closure and foreign
witness statements below are historical and superseded by this note.

# PDP-6 RAW PURE executable backing lifetime (K07), v1945

2026-11-26. K07 implementation closed. This is the scoped acceptance
of immutable executable ownership, physical source verification,
provider admission/retirement and retained RAW recovery, not
production RAW PURE enablement.

## Accepted ownership and identity contracts

The original mount/provider and immutable executable identity are
retained across RUN/EXEC attachment, reclaim and PREFIX/FULL restore.
Transient stack reservations prevent mutation or retirement before
permanent attachment; both the published and transient owners are
checked across the physical provider lease. Namespace revision and
private stack lifetime guards prevent stale publication. A failed
retirement keeps the original owner, and poisoned noncritical D6FS
administrative discard retires only under the final owner barrier.
D6FS and TSFS keep their existing typed physical identities; stamped
DTFS verifies both volume-ID halves and valid state before and after
reconstruction. READ_SUSPECT blocks new DROP while allowing exact
source recovery; ambiguous write state does not confer admission.
Legacy unstamped DTFS remains subject to operator trust. No hardware
reel lock, same-ID clone detection, or invisible swap-and-return
between proof reads is claimed.

## Physical source recovery

The v1945 original 48K PDP-6 guest uses byte-identical production
v1942 GCC KCORE/KINIT/PURE images and the genuine v1944 guest-seeded
sealed DTVOL2 reel. Four plain root children and one compressed
secondary child are admitted; the sixth RUN refuses correctly.
No-growth BRK leaves MM18 unchanged. Slot 3's actual FULL entry
PC040756 (opcode 200000000004) observes independently pinned
PACKED 113600..120000 and PROCESS 122000..130000, stable u-area
051274, backing 037141000000 and destination capacity06000.

The actual DTC3 unit is detached before media proof. The compiled
FULL call returns MM_ERR_INVAL (-4) at PC034453. The independently
formatted different-ID CLEAN native reel is attached and actual
FULL returns -4 again. Both transactions leave MM17, the exact
original PACKED extent with pin0, no target, original backing and
both saved ID words, cleared transaction escrow, and READ_SUSPECT
media0300003. The entire 2176-word suffix is identical at all three
snapshots. Neither failure publishes a runnable replacement.

Reattachment of the original reel permits restoration, all child
mutable-data sentinels, five exact PID/EXITED/status WAITs and final
unmount. Z has MM8, no child VM, u-area or reservation, root0 and
ordinary healthy retired media tag0100003. Both reels are byte-
identical to the inputs. Normal HALT PC000000, simulator exit0.
Clock B5659, FULL/faults6300, Z8548 decimal; the failure checkpoints
share a tick and do not establish interruption-during-rollback.
Independent verifier PASS; 12 altered actual transcripts rejected
(verifier tests, not executed guest fault cases).

## Acceptance boundary

The 13 distinct v1940 native regression groups are retained: source
identity, provider substitution, DTFS/D6FS/TSFS media, DROP admission,
mutation lease, transient reservation, EXEC handover and lowmem,
namespace epoch, stack lifetime, MEMFS mutation and D6FS discard.
No production kernel or boot source changed between v1939 and v1945.
The v1942 ordinary physical pressure/FULL/WAIT and v1944 both real PI6
reservation/retirement orderings, together with v1945 physical DTFS
loss/foreign/return, close K07 implementation. Native D6FS/TSFS
identity is not whole-guest media exchange acceptance. K08 scheduler
integration, K09 production root/controller/physical and KCC matrices,
and K17 native KA10 remain open. RAW PURE and mode3 default OFF.

No production source, image, ABI, per-owner storage or RAM growth.
DAIMOS closure 38094b56; testkit 0026580 and 13bab63.
Testkit: `tests/validation/pure-k07-loss-v1945/README-v1945.md`.
Accepted trace SHA256:
c2c9d9d516fca6b0089279b3e50ce2e38df99269b8692292cb5716da9c8b57ac.
