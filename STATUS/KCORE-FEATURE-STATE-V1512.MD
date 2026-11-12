# DAIMOS KCORE feature state v1512: lost Type-236 completion / PI6 timeout

Date: 2026-11-12. Incremental physical acceptance over v1511.
Production KCORE remains the accepted v1507; there are no new
process states, permanent resident words, runtime API changes,
unsafe forced backing releases or code-size optimization claims.

## Physically accepted lost-PI2 / real PI6-watchdog boundary

The full disposable 256K KCC PDP-6 F07 testkit v1512 profile
first selects an actual BACKSTORE owner after authentic hardware
swap payload transfer. For its next two selected real Type-236
DR_DONE completions, test-only PI2 instrumentation acknowledges
the controller without publishing the completion of the original
active private stack descriptor. The descriptor is left owned
for the **unchanged production 300-tick PI6 watchdog** to expire.
That watchdog then invokes the real DRM descriptor completion,
counts a storage error, wakes the original slot-0 caller and
returns negative BACKSTORE read for normal MM unpin/free rollback.
Subsequent reads return to the original checked-free refusal.
Other independent processes' Type-236 requests are unaffected.

Real guest: 39 source-linked SIMH samples, including 16 fault-on
samples 24..39. Selected bad process slot `010` octal, original
BACKSTORE run `02350`/`0330`, exact allocated DISK record
`002350000330`. Its nonresident VM, stable u-area, bitmap and
used-block accounting, v1507 retry stamp and pending TERM remain
authoritative, with zero false READY_ONCE, repair, close, finish
or zombie publication. The first fault-on sample shows
`lost_pi2=1`, `watchdog_errors=0` and a real still-active
Type-236 descriptor (not an immediately fabricated error).

At sample 25 the counters show `lost_pi2=2`, real PI6
`watchdog_errors=2`, actual DRM storage errors=2, and direct
PI2-error injection **zero**. By sample 39, the real failed
slot-0 service count is exactly **32 = 2 PI6 timeout failures
+ 30 subsequent checked free refusals**. PI6 clock progressed
from 36219 to 59022 decimal, a genuine **22,803 ticks**.

Independent resident PID 5 progressed from 42 to 1034 genuine
user SLEEP calls (+992). Independent swapped PID `011` was
physically SLEEP/DISK at sample 30; one real checked restore
committed by sample 31, after which it executed **562** user
SLEEP syscalls. The source-linked exchange count remained zero.

Strict v1512 physical verifier PASSED on finalized guest/console
logs. Its self-tests reject twelve unsafe timeout/ownership/
progress mutations plus missing digit-bearing `lost_pi2` source
evidence; source-injection tests reject four malformed watchdog/
descriptor sites. Native v1079 watchdog, v1080 FIFO and v1247
PI-register tests also PASS. Source testkit versions `25e0287`
(preparation), `27814c7` (fail-closed digit-key parser),
`3cd5e6f` (physical acceptance). DAIMOS report including
the sixteen-sample table and SHA256 evidence:
`design/kcore-backstore-watchdog-physical-v1512.md`.

The guest was deliberately stopped at sample 39, **not a full
F07 HALT**, because the original permanently failing DISK owner
still holds its backing and cannot consume pending private TERM.
This accepted experiment suppresses PI2 completion after a real
Type-236 transfer; it does **not** demonstrate actual physically
damaged magnetic media, permanently detached hardware, complete
lost-media recovery, or terminal reclaimed owner resources.

## Remaining KCORE feature gates

1. Genuine DEVICE/media inaccessibility, recovery/reconstruction
   and safe long-lived owner policy; no phantom free, false ZOMB
   or invalid parent WAIT success.
2. Mixed DEVICE/VOLUME, DTFS tape-root, BCACHE, LOGSTORE/MEMFS
   provider fault/recovery and fail-closed routing integration.
3. F1 immutable swapless PURE text reclamation, mutable suffix,
   full APR/MM restoration and rollback with RAM benefit.
4. F3 dynamic FSINFO 1/4/8/26 mount enumeration; do not overwrite
   separate in-progress v1492 work.
5. Optional compression and final KCORE optimization only after
   functional acceptance and net measured RAM savings.

Production v1506 repaired-owner TERM, v1507 retry, and physical
v1510-v1512 independent fairness/error rollback remain accepted.
VERSION 0.5, strict 32K HOLD. MRES unload/reload POST-1.0;
unified TSFS and D6FS packer separate scope.
