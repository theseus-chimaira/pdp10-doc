# DAIMOS KCORE feature state, v1509

Date: 2026-11-12. Incremental acceptance over v1508, feature
completion only. Production KCORE v1507 has **no new process state,
resident RAM allocation, code modification, or interface change**
for the v1509 physical diagnostic.

## Physical resident-bystander progress: ACCEPTED

The v1508 booted 256K KCC PDP-6 permanently refused checked
BACKSTORE release for a selected real DISK owner, but its F07
parent had already TERM/WAITed the independent PID 5 spinner.
Thus the three prior fault-on samples established bounded retries
and ownership, not actual runnable-user fairness.

Testkit v1509 changes only a disposable copy of F07 INIT: defer the
original PID 5 spinner TERM/WAIT until after the extra-child
cleanup. Under permanent selected BACKSTORE refusal the parent
is blocked in WAIT, and real PID 5 continues to call SLEEP(2).
The original v1150 source-linked kernel witness increments only
when actual PID 5 enters this SLEEP syscall after the held-provider
and gateway conditions. The v1509 observer samples this counter,
the real process table and PI6 clock; it does not fabricate turns.

Full guest selected slot `010` octal, authoritative original DISK
run `02350`/`0330`, with record `002350000330`. Fault-on physical
samples **24-29** show checked release refusals **3, 5, 7, 9,
11, 14** while real clock advances **37630 to 45534** decimal.
All six slot-05 process-table snapshots show `SLEEP/RESIDENT`.
The spinner's actual SLEEP entries climb **98, 174, 240, 306,
372, 456**: 358 further real syscalls during persistent failure.
The failed owner retains its disk record, bitmap accounting,
stable u-area, retry marker and pending TERM. Repairs, false
READY_ONCE/RESIDENT, fatal close, finish and zombie remain absent.

The strict v1509 verifier PASSED on finalized read-only samples;
four guest-source and nine progress/ownership negative mutations
were rejected. The separate v1508 invariant verifier PASSed on
all six samples. The first v1509 test runner was checked against
both the locally extended v1508 sampler and the **clean committed**
v1508 sampler; the latter preparation passed without requiring
the unrelated v1508 working-tree edits. Testkit revisions:
`decee70` (test), `b035da1` (physical evidence),
`cb0f9b1` (clean-baseline reproducibility). DAIMOS evidence:
`design/kcore-backstore-bystander-physical-v1509.md`, including
five exact SHA256 fingerprints.

This is a purposely *sample-stopped* physical diagnostic. It is
not a normal HALT/EXIT result, a repaired resource owner, or a
claim that an independently healthy swapped job completed during
the same fault. Such a swapped contender was not observed;
only the live **resident** PID 5 is physically accepted.

## Remaining prioritized KCORE feature gates

1. Physically qualify a distinct healthy **swapped** process's
   restore and execution while another owner's BACKSTORE run is
   persistently refused; continue preserving its exact metadata.
2. Characterize true DEVICE/media loss and safe long-lived backing
   ownership or verified repair/reconstruction without silent
   forced free, false ZOMB or premature parent WAIT completion.
3. Mixed DEVICE/VOLUME, DTFS tape-root, BCACHE, LOGSTORE/MEMFS
   and provider error/recovery integrations.
4. F1 uncompressed swapless PURE-text reclamation, immutable
   backing, mutable suffix, MM and full APR restore/rollback.
5. F3 dynamic FSINFO enumeration to 26 mounts, including the
   separate local v1492 work; physical 1/4/8/26 profiles.
6. Optional compression after verified PURE RAM benefit; remaining
   KINIT/clock/device profiles and final KCORE size optimization.

Earlier v1506 repaired physical owning TERM and v1507 native
normal/realtime bounded retries remain accepted. 32K HOLD,
VERSION 0.5. MRES runtime unload/reload POST-1.0, unified TSFS
and online D6FS packer outside this KCORE completion scope.
