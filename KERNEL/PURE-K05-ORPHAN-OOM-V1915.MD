# PDP-6 K05 v1915: requester-only BRK capacity OOM with foreign FULL child

Date: 2026-11-24. **Original source-built 48K PDP-6 physical PASS, scoped.**

## Source policy and question

The K05 v1910-v1914 physical tests established that a child
stalled in RAW PURE FULL rollback can remain LIVE under a
foreign-claimed temporary PROCESS target, even after its
parent and other processes exit. Its original PACKED RAW
suffix and private executive continuation cannot be freed
without definitive MM ownership. v1915 tests a separate
ordinary process's genuine physical-memory capacity failure
while that foreign owner is still quarantined. Rejected
allocation must affect the requester, NOT steal another
owner's retained RAW/PROCESS extents or force a premature
last-user HALT.

The original PDP-6 VM BRK path first checks whether its
requested aligned APR could EVER fit inside a real managed
physical MM arena. A mathematically impossible request is
refused with MM NOMEM before compaction or destructive
swap-victim reclamation. The original BRK syscall then enters
its requester-only fatal-OOM exit path with status1. This
is **an early capacity impossibility** test, not a general
fragmented-allocation or descriptor-table exhaustion test.

## Original guest and source-linked test-only user gate

The unchanged fixed PDP-6 KCORE, optional PURE MRES and
original 48K compressed-TSFS boot are source-built again.
Only the TEST USER executable differs: the v1907 original
five-child pressure/TERM fixture adds one C89 call behind
the existing test-only user gate `v1901_term_gate`. The
original PACKED-source `mm_free` first genuinely returns -5,
the original full target is test-claimed by foreign owner2,
and the parent's genuine TERM(3) attempt is refused. Only
then does the controller open the parent user gate, causing
it to emit `P` and invoke ordinary `dsys_brk(0130000UL)`.
If this syscall returns, the guest prints `!O` and the test
fails. No test-time allocator call, executable patch or
fabricated MM error return is supplied by the controller.

The original `mm_arena_fits_v1260` physically executes at
PC `002154` with the real AC1=`0130000` and AC2=`002000`
octal. Its sole original physical arena is
`044633..0140000`, size `073145` octal words. After APR
alignment its largest possible interval is `072000`, so
the requested `0130000` words cannot fit even if every
other process allocation were first reclaimed.

Original `proc_fatal_close_current_v879=013713` then runs
with AC1=1 and the current original parent at slot1.
The real native `proc_exit_current=013655` and permanent
slot0 finalizer release only the requesting parent's own
KERNEL_DYNAMIC u-area `045761..046371` owner `01001`
and PROCESS `130000..140000` owner1: **MM18 -> MM16**.
The original MM18 table did not change during the BRK
capacity check, fatal-OOM entry or parent SYS_EXIT entry.

## Native child cleanup and last-owner conservation

The three remaining ordinary child users slots4,5,6 execute
distinct original user-mode EXIT syscalls, each followed by
its own native `proc_idle_loop=017552` return before the
next EXIT stop is armed. Their exact six KERNEL_DYNAMIC and
PACKED_PROCESS extents disappear lawfully, with no unrelated
MM changes: **MM16 -> MM10**. All ordinary active user slots
are FREE. The sole last-LIVE child slot3 is orphan parent0,
descriptor `047017600001` (TRANSITION and VMbase0), with:

| Existing original resource | Physically preserved |
| --- | --- |
| Authoritative PACKED RAW | `111200..120000`, owner3, pin0 |
| Unretired provisional PROCESS target | `120000..130000`, foreign owner2, pin0 |
| Private saved executive context | CTX_PC `340000016704`, FDCTL `000000004010` |
| Target identity escrow | uarea word067 `040000120000` |
| Parent and live process bound | parent0, `proc_high_slot=4`, slot0 running idle |
| ROOT and unrelated MM owners | Entirely unchanged |

The original slot0 `proc_finish_service_v881=013065` then
performs three genuine recovery scans at physical clock
`024075`, `025075`, `026075` octal, exactly `01000`
(512 decimal) ticks apart. The shared pending retry flag
and capped delay remain armed at every native checkpoint.
The final PCLK PI6 saves original hardware word
`340000017565`, interrupting actual slot0 idle. The
original PDP-6 does NOT execute a guest HALT. The simulator
is ended by the operator after the final snapshot.

## Strict source-linked acceptance

The v1915 positive audit checks the byte-identical original
KCORE/PURE hashes, original user-mode BRK and exact ACs,
the physical arena geometry and native OOM control flow,
the parent-only MM18->MM16 release, and every exact
six-peer MM16->MM10 removal. It requires preservation
of the foreign target, original RAW, process descriptor,
saved executive context, target escrow and ROOT at EVERY
OOM/EXIT/retry/PI6 checkpoint. The controller observes the
complete real process table `045700..045726` and confirms
the source-linked KCORE `proc_table` variable at `024635`
points to `045700` rather than treating the pointer as
the table's physical address.

Four independent unsafe physical transcript mutations
all reject: a false native BRK target, an unauthorized
foreign PROCESS owner change, lost private escrow, and
premature last-orphan FREE. Original KCORE/PURE binaries
are not modified. The user executable and its TSFS metadata
are test-only changes, not production code.

Two preliminary v1915 runs were correctly rejected for
incomplete test-controller observations. The v1915/v2
snapshot did not include slot6 word `045722`. The next
v1915/v3 snapshot mistakenly read KCORE variable `proc_table`
as an inline table, omitting `045722` again. These were
addressing errors in the test controller, NOT an observed
kernel or compiler defect. Accepted v1915/v4 observes
all original process descriptor words. No workaround or
forced MM outcome was introduced.

Accepted original normalized physical SHA256:
`fc8ac14b4c1a31507d974982fb624a52abad72f2d15250c6683a609d9ea53d58`.
Source-linked simulator script SHA256:
`d0a6860c3fc8fb4979ec4f73dccd84d0d17f26281e8a7de36ed701e69b995bfc`.
Byte-identical fixed KCORE SHA256:
`a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`.
Byte-identical optional PURE SHA256:
`09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`.

Testkit:
`daimos-testkit/tests/validation/pure-k05-orphan-oom-v1915/`.
DAIMOS design:
`DAIMOS/design/kcore-k05-orphan-oom-v1915.md`.
Original physical evidence:
`$TMPDIR/daimos-k05-orphan-oom-v1915-physical-v4/`.

**K05 OPEN:** this establishes requester-only capacity OOM
without corruption of a foreign-held child, but not general
fragmentation/compaction MM refusal, descriptor exhaustion,
every PI6/allocation interleaving or cross-platform release.
No production C/ASM, fixed RAM/IMAGE/BSS or default policy
change. K06-K09 OPEN; RAW PURE and mode3 remain OFF.
