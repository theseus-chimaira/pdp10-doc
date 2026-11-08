# v1342: distinguish post-release guest wait dependency from MM failure

Date: 2026-11-08. Status: **diagnostic only**, no full-system
acceptance and no production kernel source change.

The v1340 booted 256K PDP-6 guest exercised genuine typed MM
allocation shortage, no-partner cooldown, independent owner release,
and subsequent successful swap-ins (13 to 18). The four real 8K
owner extents were freed through production `mm_free`, and the
independent spinner's VM pin was removed with production `mm_unpin`.
The original `@%&*` user milestones were preserved by the corrected
Ctrl-E host sampler. However, 24 periodic samples ended without
the guest's final HALT, TERM/WAIT sequence or zero-backstore proof.

The new v1341 testkit decoder interprets **the actual linked**
`proc_table`, `proc_high_slot`, `proc_timer_clock` and all process
descriptors from the existing raw SIMH transcript. At sample 24:

| Process | Real state | Wait condition |
| --- | --- | --- |
| INIT (slot 1) | SLEEP, swapped | CHILD wait |
| Witness (slot 2) | SLEEP, swapped | interruptible event wait |
| OOM requester (slot 3) | SLEEP, resident | timer deadline, 334 ticks ahead |
| Gateway (slot 4) | SLEEP, resident | `fs_provider_wait_event_v917` |
| Spinner (slot 5) | SLEEP, resident | timer deadline, 2 ticks ahead |
| Holder (slot 6) | SLEEP, resident | `daimos_test_hold_event_v1150` |

The linked PI6 clock advanced to octal `0177102`; the nearest
timer frontier continued advancing. Successful swap counts stayed
at 34 out / 18 in after sample 13, with no new swap-in shortage.
This rules out a stopped clock, but **does not** prove the
reason the requester did not finish.

The original v1150 **test-only** provider lease fixture releases
the holder from its private event wait only when INIT calls
`proc_sleep_ticks` after a genuine OOM marker and at least two
observed fatal provider-busy failures. At the late sample INIT
was instead waiting for its child. This is a possible test-workload
dependency cycle: holder remains gated while the requester cannot
complete; INIT cannot enter the sleep hook while waiting for that
requester. A production provider or scheduler defect must not be
inferred until the exact original release preconditions are measured.

The v1342 versioned disposable guest extends the read-only sampler
with the original source-linked `armed`, `real_oom`, `fatal_busy`,
`release`, `holder_return`, `holder_enter`, `predispatch`,
`ordinary_wait`, and `fs_provider_ready` values. This directly
distinguishes a missing OOM event from a missed test-only release
hook or a genuine kernel wait failure. The sampler's Ctrl-E,
process-table, early-HALT and strict-postflight selftests pass.

Testkit: `daimos-swap-stall-waits-v1341.py`,
`daimos-swap-independent-trace-v1342.sh`, commit `24dd9bd`.
No speculative fallback, production MM/PI6/FS changes or resident
KCORE/MRES growth are justified by the current evidence.
