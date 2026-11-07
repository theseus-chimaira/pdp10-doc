# DAIMOS v1330: late physical owner reservation is not a valid pressure setup

Date: 2026-11-07. The v1323 single-1K-owner F07 experiment has
completed as a **negative setup result**, not a swap liveness test.
DAIMOS production MM, swap, scheduler, KCORE and MRES code are
unchanged; the rejected v1208/v1211 resident swap fallback is absent.

## Booted 256K guest result

Testkit `c43fe7b` derived a disposable 256K KCC F07 PDP-6 guest
with 18 additional 24K-word-BSS children and real DSK/D6FS swapping.
At the original post-admission pressure stage, INIT attempted one
APR-`02000`-aligned, `02000`-octal-word (1024-word)
`MM_TYPE_KERNEL_DYNAMIC` extent through the actual production
nonreclaiming allocator, owner `075`. This MM allocation failed.
Original test INIT emitted `!t`, then HALTed, before the normal
provider/OOM/reap stages.

The authoritative post-HALT counters show exactly:

* 51 normal swap-outs and 35 normal swap-ins;
* test owner count=0, prepared=0, freed=0, error=-1;
* actual swap-in MM allocation failures=0, no-partner cooldown=0;
* process exchange operations=0;
* swap/backstore blocks outstanding=3440 decimal at the early HALT.

Outstanding backing at an **aborted test** is not the normal final
cleanup assertion. No successful full-guest completion, rollback
acceptance or no-partner liveness is inferred. The earlier v1322
single-8K owner experiment failed at exactly the same late stage
(`!t`) with zero swap-in MM failures/cooldowns. Together, the two
results show that **late allocation of an extra test owner** is an
unsuitable way to force swap-in pressure in this real process load;
the test allocation itself cannot be admitted.

Original v1323 console, SIMH sampler, address manifest, kernel map,
boot configuration and versioned checksum manifest are retained:

`$TMPDIR/daimos-swap-physical-owner-v1323-setup-failure-r1.tar.gz`

SHA256: `04944fa347f6b259ca779ea96126ab440fa1eeeb9081794b56f0ed4d1bf3cb25`.

## Test direction and diagnostic provenance

The separately committed testkit `1a0693e` v1326 alternative
temporarily pins *existing real resident process extents* through
production `mm_pin`, excluding the real OOM requester, and releases
them through `mm_unpin`. This constrains legitimate reclamation and
equal-size exchange eligibility without fabricating an MM return
value or trying to allocate a new physical owner at a fully occupied
stage. The v1329 wrapper (`5a0adb8`) uses that fixture and the
corrected host-only v1326 SIMH sampler. Preflight PASS does not
constitute a full-guest result; the 256K execution remained OPEN at
this documentation checkpoint.

Host sampler `e436459` v1326 preserves an unmodified raw SIMH trace
and filters only exact `sim> ` debugger prompts from the separate
guest milestone transcript during both GO execution and post-HALT
examination, including split PTY reads. Its strict self-test and
earlier v1306/v1324 tests PASS. This corrects a diagnostic format
problem, not production PDP-6 software.

Full physical no-partner recovery requires a finished booted guest,
typed actual swap-in MM error, a real no-eligible-partner cooldown,
independently runnable progress, later restore and exact zero backing
leaks. These have **not** yet been accepted.
