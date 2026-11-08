# v1343: independent completion of the disposable held-provider fixture

Date: 2026-11-08. Status: **test fixture repaired and selftested;
full booted end-to-end acceptance still pending**.

## Source-linked proof of the original fixture dependency

The genuine 256K KCC PDP-6 v1342 guest reproduced the original
real-MM no-partner pressure and independent owner release, then
reached the following source-linked state at sample 12:

| Test/kernel observation | Value |
| --- | ---: |
| Genuine requester OOM marker | 1 |
| Actual fatal provider-busy attempts | 12 |
| Holder entered serialized FS lease | 1 |
| Test-only holder release signaled | 0 |
| Holder returned through unlock | 0 |
| Owner MM extents remaining | 0 |
| Spinner process pin remaining | 0 |
| Successful swap-outs | 34 |
| Successful swap-ins | 18 |

The unchanged process descriptors showed INIT in CHILD wait,
requester in timed SLEEP, gateway waiting for the real FS provider,
and holder waiting for the **disposable**
`daimos_test_hold_event_v1150`. The PI6 timer advanced and the
independent spinner remained runnable between sleeps.

The v1150 test-only lease completion routine was gated by actual
OOM and at least two fatal provider-busy failures, but could only
be invoked on INIT's next call to `proc_sleep_ticks`. INIT was
already blocked in `proc_wait_child`, so that test-only release
could never occur. This is an established **fixture scheduling
dependency**, not a demonstrated production MM or PI6 defect.

The v1342 diagnostic was intentionally interrupted once these
conditions were observed; it did not complete the full guest.
Its retained original source/map/addresses/raw SIMH and verified
SHA256 manifest are in
`$TMPDIR/daimos-swap-provider-dependency-v1342-evidence-r1.tar.gz`,
SHA256 `9a286c2a48e635d19411fab3fa2cbc453c99763561608200fcf704130bbb875b`.

## v1343 test-only correction

`daimos-f07-independent-provider-release-v1343.py` modifies
**only the disposable guest's already test-injected**
`proc_sleep_ticks` spinner branch. After the original independent
spinner progress counter, it checks the real OOM marker and whether
the test release has already been signaled. Only then does it
branch to the **original** `daimos_test_maybe_release_v1150`.
That original routine still requires at least two real fatal
provider-busy attempts, sets the original test event, and invokes
the unchanged production `proc_wakeup_event`. The holder resumes
on its own private executive stack and calls the original
`fs_provider_unlock`; no other process steals the lease.

The injector adds no new permanent state, no synthetic allocator
status, no forced process transition and no direct provider unlock.
Three unsafe fixture mutants (missing spinner, weakened busy gate,
missing original release) are rejected by the testkit selftest.
The v1343 versioned full-guest source preflight passes. Testkit
commit `28bd504`.

## Still required

A fresh 256K booted v1343 guest must show the real OOM/busy
prerequisites, exactly one provider release, holder's genuine
unlock, requester completion, all 18 additional TERM/WAIT reaps,
original witness/provider/OOM milestones, final HALT and zero
MM owner, pin and disk-backing leaks. No full-system acceptance
may be claimed from the injection selftest or intermediate traces.

Production KCORE, MM, FS, PI6, swap service and MRES remain unchanged.
The speculative 16-word resident fallback remains excluded.
