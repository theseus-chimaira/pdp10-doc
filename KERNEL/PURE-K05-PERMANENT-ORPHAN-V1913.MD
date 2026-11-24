# PDP-6 K05 v1913: permanent last-orphan quarantine without MM owner repair

Date: 2026-11-24. **Scoped original PDP-6 48K physical PASS.**

## Terminal question

The v1912 regression proved the final original child can resume
its private executive continuation and reach a genuine normal
guest HALT **after** the artificial foreign MM owner is corrected.
v1913 tests the opposite ordering: the exact foreign-owner
ambiguity is NEVER reconciled during this run. Once the parent
and all other children exit, the final TRANSITION-held child
still has a retained authoritative PACKED RAW source, a
foreign-claimed provisional PROCESS target, and a live saved
executive stack. Freeing any of these speculatively could
destroy the only legal continuation or reassign unknown memory.

The existing fail-closed policy must therefore preserve this
child as LIVE despite there being no runnable user APR. The
machine must remain in bounded slot0 recovery/idle, not enter
normal last-process HALT, until trustworthy MM reconciliation
actually becomes possible. This is resource retention, not
automatic recovery from permanently incorrect metadata.

## Source and original 48K physical evidence

The test uses exact original v1910 source-built 48K five-child
compressed TSFS guest, byte-identical fixed KCORE and optional
PURE MRES. The complete original v1911 sequence runs first:
genuine PACKED-source `mm_free=-5`, test-only target owner3 ->
foreign owner2, native parent TERM refusal, real parent EXIT,
distinct healthy child slots4/5/6 EXIT, only child3 still live
with MM10, three capped native slot0 retries, and original
post-parent and last-orphan PI6. The entire pre-extension
trace passes the independent strict v1911 oracle again.

v1913 then runs **12 additional original slot0 native**
`proc_finish_service_v881=013065` calls with distinct original
`proc_idle_loop=017552` returns after EVERY call. The
physical original PCLK values increase exactly `01000` octal
ticks (512 decimal) between native retry entries, from
`027342` at new retry1 through `042342` at retry12, and
also from v1911 retry3 to v1913 retry1. Original shared
pending remains set and its backoff stays capped at `01000`.

Two MORE real PCLK PI6 enter original fixed-KCORE
`proc_sched_pi_tick=017225` after additional retry6 and
retry12. The actual machine-saved PI6 words are
`340000017553` and `340000017564`, each interrupting the
genuine slot0 idle code. Counting original v1911's two
PI6 checkpoints, the full run observes FOUR genuine PI6s
while following the original parent/last-orphan lifecycle.

| Still-live resource | Original physical state throughout |
| --- | --- |
| MM extent table | Exactly 10 records, unchanged across all extra retry entries, idle returns, PI6 |
| RAW PACKED source | `111200..120000`, owner3, pin0 |
| Provisional full PROCESS target | `120000..130000`, foreign owner2, pin0, compacted MM metadata `024707` |
| Sole child process descriptor | `047017600001`, parent0, TRANSITION, VMbase0 |
| Saved executive continuation | CTX_PC `340000016704`, FDCTL `000000004010` |
| Target-base escrow | Existing uarea word067 `040000120000` |
| Live process bounds | `proc_high_slot=4`, current slot0, every other user slot FREE |
| ROOT | Unchanged throughout |

There is no controller owner-repair deposit after the original
injection, no synthetic MM return, no forced FINALIZE, no
TRANSITION unlock and NO normal or diagnostic guest HALT.
The simulator exits only by its operator command after the
last original physical PI6 evidence has been collected.

Original assembler policy was also reviewed: the non-sleeping
`proc_finish_service_v881` keeps a pending failure marker and
reports true last-user completion only after a successful
release; `proc_has_live_user` treats a still-live TRANSITION
descriptor as active. `proc_idle_loop` can jump to
`proc_exit_halt` only on a genuinely completed last-user
cleanup. **No runnable user** is not equivalent to **no live
user**. There is no new fixed memory table or additional
per-process field to carry this policy.

Four independently executed forged physical trace controls
reject false owner correction, premature transition-clear,
missing target escrow and false 512-tick clock intervals.
FIFTEEN genuinely clock-separated capped last-orphan retries
are now physically observed across v1911 and v1913. This
is bounded evidence, not a mathematical proof of indefinite
uptime or all 18-bit timer-counter wrap boundaries.

Accepted normalized original physical SHA256:
`88867609a4329cd0a352f9681a3c59e34afe24b557ed20c3354d84918a0b7529`.
Source-linked simulator controller SHA256:
`677886f622428b6f579a09391cd8219a8a0fc29e0c2f32b49dae824fd4bb7486`.
Fixed original KCORE SHA256:
`a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`.
Original optional PURE SHA256:
`09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`.

Testkit:
`daimos-testkit/tests/validation/pure-k05-permanent-orphan-v1913/`.
DAIMOS in-tree design:
`DAIMOS/design/kcore-k05-permanent-orphan-v1913.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-permanent-orphan-v1913-physical-v1/`.

**K05 remains OPEN.** This test supports the safe conservative
policy while an MM owner remains unresolved, but does not make
permanent foreign ownership safely reclaimable or prove
unbounded execution. Remaining allocator/PI6/wrap/platform
cases need separate evidence. K06-K09 OPEN. No production
C/ASM, fixed RAM, KCORE/PURE IMAGE, BSS or feature default
changed; RAW PURE and mode3 remain disabled by default.
