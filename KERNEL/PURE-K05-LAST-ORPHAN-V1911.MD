# PDP-6 K05 v1911: quarantined foreign FULL as the last live user

Date: 2026-11-24. **Original 48K PDP-6 physical PASS (scoped).**

## Why this matters

The v1906 optional RAW PURE target-base escrow retains a
still-live foreign-claimed temporary FULL PROCESS extent under
TRANSITION, without forfeiting the original PACKED source or
the saved private executive stack. v1910 proved that its
ordinary parent can EXIT normally and adopt the child into
parent0 while releasing only parent-owned MM. v1911 tests
the next true boundary: whether slot0 stays alive after ALL
the remaining ordinary children exit and the unrepaired
foreign-claimed child is the only live process.

The same real original 48K five-child compressed TSFS guest,
permanent KCORE and optional PURE MRES execute unchanged.
One genuine PACKED source `mm_free` returns -5; the source
pin is released and only the exact provisional full target's
original MM metadata is test-retagged owner3 -> foreign owner2.
Genuine parent TERM(3) refuses (`TR`). Genuine user SYS_EXIT(0)
enters permanent `proc_exit_current=013655`, invokes slot0
owner finalizer `012610`, and retires the parent (v1910).

## Native last-user observations

After original parent EXIT, actual original child EXIT
breakpoints see `proc_current_slot=4`, then `5`, then `6`.
There is an independent original fixed-KCORE idle-loop entry
at `017552` after EACH of these three EXIT requests. No
SIMH repeated breakpoint at the same unexecuted PC is counted
as another EXIT. Final `proc_high_slot=4`, current slot0,
slots 1,2,4,5,6 FREE; original slot3 is the sole live user.

The complete physical MM table changes **16 -> 10** records,
with exactly six extents freed:

| Original resource | Address range (octal) | Owner |
| --- | --- | --- |
| Child4 KERNEL_DYNAMIC uarea | `047427..050037` | `01004` |
| Child5 KERNEL_DYNAMIC uarea | `050037..050447` | `01005` |
| Child6 KERNEL_DYNAMIC uarea | `050447..051057` | `01006` |
| Child6 PACKED RAW | `061200..070000` | `6` |
| Child5 PACKED RAW | `071200..100000` | `5` |
| Child4 PACKED RAW | `101200..110000` | `4` |

No unrelated MM extent or owner changes. The original slot3
descriptor remains `047017600001`: parent0 orphan with
TRANSITION, VMbase0; authoritative PACKED RAW source
`111200..120000` owner3, full provisional PROCESS target
`120000..130000` **foreign owner2**, both unpinned,
are still physically allocated. Original private executive
CTX_PC `340000016704`, FDCTL `000000004010`, and already
existing uarea word067 target escrow `040000120000` stay exact.
The original ROOT is unchanged.

Three independent original slot0 `proc_finish_service_v881`
entries at permanent KCORE `013065` are separated by actual
idle-loop completions. Their physical PCLK clock values are:

| Retry | Original PCLK (octal) | Shared backoff (octal) |
| --- | --- | --- |
| 1 | `020046` | `01000` |
| 2 | `021046` | `01000` |
| 3 | `022046` | `01000` |

Thus native original clock advances **exactly `01000` octal
ticks (512 decimal)** between successive real retries, with
no new process table or additional fixed resident RAM. The
entire MM10 table, ROOT, orphan state, executive stack and
escrow are unchanged throughout. The last original PCLK/PI6
at `proc_sched_pi_tick=017225` saves real hardware word
`340000017617`, interrupted PC `017617` inside native slot0
idle. There is no native guest HALT and no unsafe child free;
the controller explicitly issues simulator operator EXIT
after its bounded observations.

The source-linked read-only physical verifier proves exact
native EXIT and idle break counts, original MM source -5,
foreign-owner injection, source/target preservation, all
six peer MM deletions, the single remaining orphan and
three distinct 512-tick retry intervals. Three targeted
unsafe physical mutants reject a falsely live sibling,
cleared pending retry and missing target escrow at last PI6.

The initial v1911/v1 experimental transcript was rejected:
the controller rearmed the same SIMH EXIT breakpoint without
advancing its PC, producing three stops of the SAME slot4.
v1911/v2 fixes this by requiring original idle completion
between each child EXIT and between each retry checkpoint.
This was a test-controller problem, not a kernel defect.

Accepted physical normalized SHA256:
`2863df28f5712639a8c42ecce7c6b6566cd9328f06cbbee7809f2642dd60ff9c`.
Physical control script SHA256:
`58fdb78e540ce849d92193a0930400ac7f28f970325277bb6d8806f68741c9d7`.
Unchanged fixed KCORE:
`a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`.
Unchanged optional PURE MRES:
`09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`.

Testkit:
`daimos-testkit/tests/validation/pure-k05-last-orphan-v1911/`.
DAIMOS design:
`DAIMOS/design/kcore-k05-last-orphan-v1911.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-last-orphan-v1911-physical-v2/`.

**No production C/ASM, IMAGE, BSS, fixed RAM, process storage
or default feature policy changes.** K05 still OPEN for
permanently unknown-owner terminal policy, other physical
MM/PI6 orders, and wider acceptance. Three finite retries
do NOT prove indefinite survival; the foreign-owned target
must not be freed without authoritative reconciliation.
K06-K09 remain OPEN, RAW PURE and mode3 default OFF.
