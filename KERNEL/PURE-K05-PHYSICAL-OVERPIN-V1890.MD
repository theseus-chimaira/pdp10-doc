# PDP-6 K05: actual physical overpinned FULL refusal v1890

Date: 2026-11-23. **Scoped 48K original PDP-6 physical PASS**.
K05-K09 OPEN. RAW PURE and mode3 disabled by default.

## Original source-matched hardware sequence

The source-built 48K PDP-6 KCORE and optional v1855 PURE MRES are
unchanged from the accepted full five-child normal-run image. A
newly run KCORE reclaim-JRST probe observes the installed removable
PURE base `032417` octal, source-linked bounded FULL entry `036202`
and service return `033067`. There is no hard-coded stale MRES PC.

At the original FULL entry, after the native RUN/PIPE pressure guest
has admitted five child processes and before the compressed DTC3
image read, slot3 owns two separate physical MM reservations:

* Authoritative PACKED RAW suffix: `111200..120000` octal, pin **1**.
* Temporary complete PROCESS image: `120000..130000` octal, pin **1**.

Only testkit SIMH control alters a runtime word: inspect the actual
MM descriptor metadata at physical KCORE address `024705`, observe
`000011000003` (target pin1), deposit `000021000003` (target pin2),
and re-read that real physical word. The production kernel and
optional MRES are not changed or instrumented. The executable
source remains on attached DTC3 throughout the test; no media loss
or replacement is confused with the MM fault.

After the original loader completes, the actual source-linked FULL
function returns `777777777773` octal, namely `MM_ERR_BUSY=-5`,
at service PC `033067`. Both original physical MM reservations
remain: source pin1, target pin2. The full MM table is identical to
its pre-injection state except for that one newly added target pin.
No other MM owner, selected TSFS ROOT, descriptor count or current
slot changes. The original code rejects the unexpected pin count
without releasing the sole authoritative PACKED RAW suffix or
publishing successful RESIDENT restoration.

A subsequent *real* PCLK PI6 reaches original fixed KCORE handler
`proc_sched_pi_tick=017205`, with clock/cooldown/cursor
`021676/0100/3` octal. Another whole MM table snapshot matches
the first failed-return state: both original extents, including
their pin counts, and the selected ROOT remain unchanged.

## Verification and provenance

The testkit versioned harness is
`tests/validation/pure-k05-physical-overpin-v1890/`, with independent
physical source/map/owner verifier and saved original-trace audit.
The complete native v1855 regression and healthy 48K five-child
WAIT/HALT acceptance remain separate tests, not substituted for
this physical pin fault.

Four **executed** negative physical trace mutations are rejected:
an invalid original predeposit pin, an absent deposited extra pin,
a missing authoritative source pin at the service return, and a
missing retained target pin at the later clock. Each is rejected
at its precise physical MM witness. The unaffected original trace
was restored after every negative attempt.

Source image SHA256:

* Fixed KCORE:
  `33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`
* Optional v1855 PURE MRES:
  `a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`
* New original physical v1890 transcript:
  `fa522cdbd0c1dfa8c429ba551d12db84d4671784928204f729c73eb8a0bb8b1e`

The source-built five-child fixture, MM field geometry and
actual physical MM count are validated; no additional permanent
KCORE word, MRES code word, process field, native MM table,
kernel data or feature flag was introduced by v1890.

## What remains open

The simulator intentionally exits at the postfault physical clock
checkpoint, **not** at native child WAIT or machine HALT. v1890
does not physically read slot3's process TRANSITION bit; the
**separately accepted v1891 physical run now does**. See
`KERNEL/PURE-K05-PROCESS-QUARANTINE-V1891.MD` for verified original
slot3 TRANSITION, VM base zero, unaffected peer process descriptors
and four physical negative mutants. Independent user child EXIT/WAIT
after the fault and recovery of a permanently corrupt pin remain
further acceptance gates, as do free/allocator uncertainty,
other PI6 timings and
K05-K09 production integration. The fix itself is v1855;
v1890 is its source-linked full-system physical fault witness.

DAIMOS design report:
`design/kcore-k05-physical-overpin-v1890.md`.
Related accepted normal-run result:
`KERNEL/PURE-K05-EXACT-PINNED-FULL-V1855.MD`.
