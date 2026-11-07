# v1340: booted no-partner recovery, independent owner release

Date: 2026-11-07. Scope: genuine booted 256K KCC PDP-6, DSK/D6FS,
disposable test-only MM reservations and diagnostic hooks.
Production KCORE/MM/SWAP/PI6/storage source and permanent RAM are
unchanged. **Final full-system acceptance is still OPEN.**

## What has been executed successfully

The v1338 guest allocated four typed, owner-075, 8192-word MM
extents early through production `mm_alloc_aligned_noreclaim`.
Its independent spinner process, slot 5, was pinned through the
unchanged `mm_pin`, and polled a read-only diagnostic syscall that
returns the actual production no-partner cooldown count.

At the source-linked eleventh live sample, physical MM allocation
returned genuine signed 36-bit `MM_ERR_NOMEM`, and the production
slot-0 service entered its no-eligible-partner cooldown. The
independent spinner observed cooldown=1 and called the existing
test-only typed-owner cleanup once, executing production `mm_free`
on all four real extents and production `mm_unpin` on its own process
extent. Observed release counters: query calls=273, last cooldown=1,
release calls=1, released=1, prepared=4, freed=4, count=0, owner
error=0, pinned base=0, release-pressure witness=1. Exchanges and
fatal swap-in errors were zero.

After the memory release, successful ordinary swap-ins advanced
from 13 through 16 to 18. Swap-outs advanced from 31 to 34. The
actual kernel was therefore able to resume swapping after an
independent physical MM release.

**The entire guest did not finish.** After the eighteenth child
admission and the INIT `^` milestone, progress stopped at 34 swap-
outs and 18 swap-ins despite advancing clock. The experiment was
interrupted at sample 20, with no HALT, completed TERM/WAIT, final
zero backing or MM resource proof. No production defect is asserted
from this incomplete execution. v1340 adds the linked
`proc_timer_next` timer-frontier word to the sampler to identify
the pending scheduler/timer relationship after release.

## Corrected host SIMH telemetry

The unmodified raw v1338 PTY transcript contains the real guest
marker `@%&` immediately before the SIMH `Simulation stopped`
acknowledgment. The filtered guest milestone stream silently
lost these bytes because the original sampler treated **all** PTY
data received between sending Ctrl-E and receiving debugger stop
as diagnostic. This is a host testkit defect, not a guest TTY or
MM release failure.

Testkit v1340 fixes the filter so bytes preceding the exact stop
marker are fed through the same split-prompt-safe guest output
parser as normally running output. The sampler excludes the actual
SIMH debugger response. Its self-test reproduces this boundary by
emitting `@%&` upon Ctrl-E before the stop marker, then checks exact
presence in the canonical and raw streams. Normal/early HALT,
source-linked process-table inspection and truncated-postflight
negative tests all PASS. The new test remains confined to testkit
and does not modify production kernel or simulator code.

Testkit commit `e9dc25c`: sampler, regression and new versioned
full-guest timer-frontier wrapper. Previous v1338 query telemetry
was committed in `e5dcb3b`.

Retained source-linked v1338 guest traces and SHA256 manifest:
`$TMPDIR/daimos-swap-independent-release-v1340-evidence-r1.tar.gz`.
Archive SHA256:
`cb5ebee862a0129e9955ef3beaba4635bca8f796aeda19166855b5241f63573e`.

## Remaining acceptance

The corrected v1340 256K guest must still complete and demonstrate
all RUN/TERM/WAIT reaps, provider/witness/OOM milestones, source-
linked actual swap restore, all typed owner frees, correct pin
accounting, zero remaining physical test extents and zero outstanding
disk backing. A sampler PASS or partial read-only trace does not
substitute for the unchanged strict final postflight checks.
The proposed 16-word resident v1208/v1211 fallback remains excluded.
