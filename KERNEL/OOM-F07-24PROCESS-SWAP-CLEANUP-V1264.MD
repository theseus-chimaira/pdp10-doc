## v1270 final disposition: experimental fallback dropped from production

The clean-source F07 test and the experimental v1208/v1211 test both
PASS. The experiment exercises additional pressure and fallback paths,
but no independent clean-source failure without an equal-size swap
partner has been demonstrated. It adds 16 words to SWAP MRES and has a
separate reclaim/restore fault graph requiring proof. Therefore the
experimental runtime changes are **not merged**. DAIMOS `226e7a5f`
restores committed production swap code and preserves the exact
reversible source patch at
`design/f07-swap-fallback-experiment-v1208-v1211-r1.patch.gz`.
It is an archive only, not linked into the kernel. Its source can be
restored with `gzip -dc FILE | git apply -` against the frozen baseline.
The 27 historically untracked F07 diagnostic fixtures were added to
daimos-testkit in `77f156a`; they remain opt-in diagnostics, not an
assertion of present-day standalone acceptance for every old probe.

This decision leaves F07 OPEN for 64-slot/high-descriptor combined
pressure and full exact-candidate integration. Production requester
OOM remains disabled and KCORE/MRES footprint limits are separate.

# F07 v1264: 24-process PDP-6 swap/OOM/cleanup regression

## v1269: same workload PASS on clean committed production source

On 2026-11-07, the same fresh 256K KCC DSK/D6FS full F07 workload
completed on committed DAIMOS `9a841363`, WITHOUT the experimental
v1208/v1211 swap modifications. This was an isolated, clean checkout,
not a copy of the locally modified assembler. The same opt-in v1262
batch-TERM fixture, 18 additional RUN children, foreign provider,
physical requester BRK OOM and 900-second timeout were used. The
normal OOM production policy remains disabled: OOM was enabled only
in the disposable guest.

The complete output was:

```
<++++++++++++++++++^@%&*?{}|~~~~~~~~~~~~~~~~~~KWP
HALT instruction, PC: 021377 (HALT 21377)
```

The v1150/v1159 provider/protected-process probes, v1203 physical MM,
v1205 24-process APR/BRK accounting, v1207 real swap/stack accounting,
all child WAIT/reap statuses, and every evidence SHA256 check PASS.
The clean guest produced 41 successful swap-outs and 41 swap-ins,
zero service-level allocation pressure, no exchanges and no fallback.
There were zero swap failures or cooldown entries.

The earlier v1264 experimental guest produced 33 successful swap-out/
swap-in transfers, eight exchanges, 18 service-level pressure events,
ten actual v1208 fallback invocations and zero swap failures. v1268
testkit evidence verification accepts these exact observed paths and
rejects five invalid counter mutations. The same instrumented link
places SWAP MRES symbols 020 octal (16 decimal) words later with
v1208/v1211, while the KCORE low endpoint is unchanged. This is not
a production-profile residency optimization result.

**Conclusion:** v1208/v1211 are unnecessary for this successful F07
workload. Keep the uncommitted experiment separate; accept it only
if an independent no-eligible-exchange failure test and rollback/
memory/performance evaluation establish a benefit. The 24-process
F07 milestone is now backed by clean committed production source,
but the complete 64-slot and high-descriptor combined stress matrix
is still missing. F07/E13/F11 OPEN; 59/64 OOM gates, production OOM OFF.

Clean evidence archive:
`$TMPDIR/daimos-f07-clean-v1269-acceptance-r1.tar.gz`

SHA256:
`b0883d79d0a7baacb007690f62be6234a458c927afabf08ad1aac2d57510dde2`

The clean-run harness was not modified between its launch and the
creation of evidence. Comparison tooling and its negative controls
are in daimos-testkit `218bd7e`, `e8c9667`, and `11e6cc9`.

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
