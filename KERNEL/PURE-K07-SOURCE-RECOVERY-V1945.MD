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
