# F07 v1264: 24-process PDP-6 swap/OOM/cleanup regression

## Verified result (2026-11-06)

The fresh 256K KCC DSK/D6FS PDP-6 full-profile test completed
successfully with 24 live user process slots, 18 additional admitted
children, genuine drum swapping, a foreign filesystem provider lease,
and disposable requester-only physical-OOM enabled. Production OOM
remains disabled. The normal 900-second guest timeout was retained.

Final guest markers:

```
<++++++++++++++++++^@%&*?{}|~~~~~~~~~~~~~~~~~~KWP
HALT instruction, PC: 021377 (HALT 21377)
```

Every extra RUN acknowledged. The requester entered physical BRK,
only PID 3 was fatally selected, INIT completed its sleeps and WAIT,
the provider owner and gateway waiter completed, the spinner was
terminated, and all 18 extra children were terminated and reaped.
Final process/FILE/MM accounting and witness survival passed.

The independent v1203, v1205 and v1207 verifiers passed real physical
MM/BRK extent accounting, 24 process slots, 12 resident process extents,
12 nonresident/swap processes, 1,792 swap blocks at OOM, and private/
idle kernel stack high-water checks. The v1150 provider-owner and
v1159 protected-process checks also passed with zero probe errors.

## Diagnostic corrections

The original v1159 protected-process probe insisted INIT and the
gateway waiter were asleep at the instant of physical OOM. Under the
long-running 24-process F07 load both can legitimately be runnable.
Testkit v1263 accepts RUN or SLEEP for those two slots only in F07
mode. The witness and original provider lease holder must remain
sleeping; fatal selection must still identify requester PID 3.
Standalone v1159 keeps its stricter sleeping-only expectations.

The older v1150 provider probe also required gateway PID 4 to be
asleep on the provider-lock event at the physical OOM instant. Under
F07 it had already entered that wait (the real pre-dispatch counter
was nonzero) but was runnable at the snapshot. Testkit v1264 accepts
that state; if PID 4 is sleeping, its event must still be the exact
provider-lock event. Lease ownership, fatal-close retries, original
holder resumption and all independent accounting remain mandatory.

The v1262 batch-termination test mode queues the same 18 real TERM
events before performing 18 individual WAIT/reap/status checks. This
removes costly sequential swap/scheduling cycles without skipping
teardown or resource validation.

Native PDP-6 assembler checks passed for both strict and F07 probes.
The v1263 and v1264 source/regression tests each reject six negative
mutations. The rebuilt guest then passed without increasing its
900-second timeout.

## Reproducibility and acceptance boundary

- DAIMOS source: `b6db5372` plus uncommitted experimental v1208/v1211
  swap changes; this is not a clean-tree production qualification.
- Testkit source: v1263 `4eaa329`, v1264 `bb2d35d`. The v1207/v1262
  runner was committed after the full guest as v1265 `5881961`.
- Evidence archive: `$TMPDIR/daimos-f07-v1264-acceptance-r1.tar.gz`.
- Archive SHA256:
  `0fc008be42997a81138f4513d603bb828a0a978903ef60d1c761dd08e2068fea`.

This is a successful 24-process stress run, not a release gate
closure. The complete F07 gate still requires its configured 64-slot
and combined high-descriptor-pressure acceptance coverage on an
accepted production-source candidate. F07/E13/F11 remain
OPEN, the OOM gate count remains 59/64, and production requester OOM
must remain OFF. Do not substitute this partial result for the full
release matrix.
