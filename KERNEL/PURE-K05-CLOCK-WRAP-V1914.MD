# PDP-6 K05 v1914: true 17-bit PCLK retry-clock rollover

Date: 2026-11-24. **Original 48K PDP-6 scoped physical PASS.**

## The K05 terminal clock hazard

The original quarantined last owner can retain its immutable
PACKED RAW source, foreign-claimed provisional PROCESS target,
existing escrow, and private executive context indefinitely
until ownership becomes trustworthy. The permanent slot0
reaper must keep retrying at its capped `01000` octal clock
interval and must never mistake *no runnable user* for
*no live user*. Clock wrap is a particularly important
boundary: a wrong subtraction could initiate an early free,
starve a forever-pending owner or spuriously HALT.

Original KCORE uses `PROC_TIMER_CLOCK_MASK=0377777` octal,
a 17-bit epoch. Original `proc_idle_loop` subtracts the saved
retry epoch from the current masked `proc_timer_clock`, masks
the difference again, and checks the original shared retry
interval. The original PCLK PI6 handler at permanent address
`017225` executes `AOS 1,proc_timer_clock`. The low 17-bit
field can wrap while the full physical clock keeps increasing.

## Unchanged original guest and synchronized test leap

The physical test reuses the byte-identical source-built
v1910 48K five-child compressed TSFS guest, fixed KCORE and
optional PURE, plus original v1913 last-unrepaired-orphan
controller. The entire real original v1911-v1913 physical
prefix runs again: PACKED source `mm_free=-5`, temporary
PROCESS target test-claimed by foreign owner2, refused parent
TERM, real parent and healthy child EXIT, only quarantined
slot3 live, fifteen capped native retries and four genuine
PCLK PI6. The pre-rebase physical transcript passes the
independent accepted v1913 AND v1911 positive verifiers.

The controller then stops at the original last v1913 PCLK PI6
entry. In order to test the actual wrap without running tens
of thousands of unrelated clock ticks, it makes exactly two
**test-only synchronized time-coordinate deposits**:

| Existing permanent KCORE location | Physical value before | Deposited octal value |
| --- | --- | --- |
| `proc_timer_clock` (`024530`) | `000000043736` | `000000377770` |
| `proc_finish_last_tick_v881` (`024532`) | `401000043736` | `401000377770` |

The pending tag, capped `01000`-tick delay, and relative
elapsed time are preserved. Original `proc_timer_next` at
`024531` is zero and stays unchanged. No MM extent, saved
executive state, code, AC1 return, or process descriptor is
deposited, and no artificial timer-interrupt success is used.
**This is a controlled near-wrap clock/epoch leap, not a
natural full-uptime timing test.**

## Genuine hardware PI6 observations

The original interrupted PI6 executes normally, incrementing
the source-linked clock to `0377771`. The controller then
observes SEVEN additional distinct real PCLK PI6 entries at
original `proc_sched_pi_tick=017225`. Each is followed by an
original `proc_idle_loop=017552` completion, so a repeated
SIMH breakpoint at one unexecuted instruction cannot count
as another timer tick. The final pair proves:

| Original physical clock state | Full clock (octal) | Masked 17-bit clock |
| --- | --- | --- |
| Immediately before last boundary PI6 | `0377777` | `0377777` |
| After original PI6 AOS and idle return | `0400000` | `000000` |

The original retry pending word remains `401000377770`
during the actual hardware rollover. No premature retry or
user dispatch is observed at the boundary.

Three new genuine native slot0 reaper calls at fixed KCORE
`proc_finish_service_v881=013065` occur at precisely:

| Native retry | Full original PCLK (octal) | Saved pending/delay/epoch |
| --- | --- | --- |
| First after wrap | `0400770` | `401000000770` |
| Second | `0401770` | `401000001770` |
| Third | `0402770` | `401000002770` |

Each native call occurs exactly `01000` octal ticks (512
decimal) after the prior retry epoch, including across the
masked clock wrap, and each has its own original idle-loop
completion. Shared backoff remains capped at `01000` and
pending remains armed after all three failed reconciliations.

The complete physical ten-entry MM table remains unchanged:
RAW source `111200..120000` owner3 pin0; foreign temporary
PROCESS target `120000..130000` owner2 pin0, compacted target
MM metadata `024707`; sole live slot3 `047017600001`
orphan parent0/TRANSITION/VMbase0; private executive CTX_PC
`340000016704`, FDCTL `000000004010`, and uarea word067
escrow `040000120000`. ROOT and all retired/free user slots
also remain unchanged. There is NO native guest HALT, forced
FINALIZE or test-only foreign-owner repair. The simulator
ends only by operator EXIT after the final physical snapshot.

The source-linked read-only positive verifier checks the
complete unchanged original MM/ROOT/CTX/escrow and strict
real PCLK/idle instruction gates. It independently runs the
accepted v1913 and v1911 physical oracles against the SAME
original pre-rebase trace, confirms both test-only clock
deposits are paired, requires exact seven observed original
PI6 increments, the true `0377777->0400000` crossing,
and three 512-tick native retry checkpoints.

FOUR independent unsafe physical mutants reject: mismatched
timer/epoch rebase, missing clock rollover, an early post-wrap
retry and false foreign MM owner repair. The first verifier
attempt had incorrectly fixed the nondeterministic pre-rebase
clock at `042342`. The accepted guest's actual pre-rebase
clock was `043736`; correcting this *verifier assumption*
to a same-run physical reading made the unchanged trace PASS.
There was no KCORE, assembler or simulator production defect.

Accepted normalized physical SHA256:
`7b7d9afefbe9e2d59e8fc46876d22ce57e3538cb0cacc39ad6fd3d09c72fa705`.
Source-linked simulator script SHA256:
`932f7607e91bb140e119158120f17938b643c1cfbec6bd74cef737fc42550c1f`.
Original KCORE SHA256:
`a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`.
Original optional PURE SHA256:
`09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`.

Testkit:
`daimos-testkit/tests/validation/pure-k05-clock-wrap-v1914/`.
DAIMOS design: `DAIMOS/design/kcore-k05-clock-wrap-v1914.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-clock-wrap-v1914-physical-v1/`.

**Scope:** the specific controlled near-wrap/reaper ordering
is physically proven with unchanged production code and RAM.
This is not a proof of arbitrary uptime, all clock-counter
states, every MM/PI6 failure permutation, or cross-platform
release acceptance. K05 remains OPEN; K06-K09 OPEN.
No fixed KCORE/PURE image, C/ASM source, BSS/RAM or default
feature policy changes; RAW PURE and mode3 remain OFF.
