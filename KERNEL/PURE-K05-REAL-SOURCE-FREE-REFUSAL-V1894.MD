# PDP-6 K05 v1894: real original MM PACKED source-free refusal and rollback

Date: 2026-11-23. Scoped **original 48K PDP-6 physical PASS**.
The full original five-child PIPE/BRK/EXIT/WAIT workload and final
HALT PASS. K05-K09 remain OPEN. RAW PURE/mode3 remain disabled.

## Contract and distinction from v1893

The v1839 FULL restoration fix examines exact MM owner/type/base/
size/pins after an error from freeing the retained PACKED RAW suffix.
If the source was actually deleted and the sole full PROCESS owner
remains, it must publish the complete target. If the PACKED source
and full PROCESS destination BOTH survive with exactly zero pins,
it must instead free only the provisional target while retaining
the authoritative RAW source. Anything ambiguous must remain
quarantined without inventing a runnable APR or freeing an
unverified unique owner.

v1893 physically exercised successful original source deletion
with a FALSE -5 return (original AC1 success was changed only
after the source actually vanished). v1894 exercises a TRUE -5
result from the original fixed-KCORE `mm_free` itself, forced by
an external temporary source pin. This is a test-only MM metadata
deposit, not a production MM patch or simulated return error.

## Source-linked physical proof

The original unchanged source-built v1855 48K PDP-6 runs five
child PURE processes, one root-backed and four compressed secondary
DTC3-backed, including ordinary PIPE/BRK pressure, writable-data
sentinels, typed child EXIT/WAIT and parent Z/HALT. DTC3 remains
attached on a private copied reel; original KCORE/PURE binaries,
guest INIT and other media are untouched.

The physical KCORE reclaim JRST and linked MRES instructions,
checked against the original compiled FULL DOBJ symbol/relocation
table, locate the exact original runtime call/return (octal):

* full bounded restore loader entry `036202`;
* original `PUSHJ 17,mm_free` for PACKED source `035310`;
* native actual `mm_free` return-status consumer `035311`;
* complete original FULL service return `033067`;
* original source MM physical metadata word `024703`.

At FULL entry the original MM owns PACKED source
`111200..120000` pin1 and full PROCESS target
`120000..130000` pin1, total 18 descriptors. The unchanged
original loader fills the target, overlays the authoritative
retained mutable suffix, verifies the v1855 one-pin pair and
then unpins both source and target. At its actual source
`mm_free` call both owners are now physical pin0 and MM count
is still 18, source word `024703: 000004000003` octal.

The private physical simulator temporarily raises only that
source descriptor pin to 1:

    024703: 000004000003 -> 000014000003

It now executes the ORIGINAL unmodified fixed KCORE `mm_free`,
which **actually refuses** the pinned source and returns its own
`MM_ERR_BUSY=-5`, AC1 `777777777773` at physical return PC
`035311`. The original MM table still contains both correct
physical owners, source pin1 and target pin0, with MM count18.
This is not a forged allocator result.

To test the authoritative exact unpinned pair (rather than the
extra-pin quarantine branch), the private external owner releases
its temporary source pin at that same physical boundary:

    024703: 000014000003 -> 000004000003

The original physical table independently confirms two EXACT
un-pinned owners. Original v1839 zero-pin reconciliation therefore
refuses the FULL transaction, frees only provisional PROCESS
target and keeps the PACKED RAW source intact and pin0. At the
original FULL service return `033067`, AC1 remains real -5,
slot3 VMbase0, no invalid RESIDENT publication; original MM
count is 17 with one correct retained PACKED owner.

The unchanged original PDP-6 user program then recovers: its
five child PIPE/sentinel/typed EXIT/WAIT checks succeed, parent
prints `Z` and the machine actually executes
`HALT instruction, PC: 000000`. Final original MM count is 8,
with no slot3 PROCESS or PACKED owner. The selected TSFS root
remains `070101000000` octal, and all unrelated MM extents and
pins remain identical through the original source-free refusal.

## Actual MM checkpoints

| Original physical moment | Descriptor count | Slot3 PACKED | Slot3 PROCESS |
|---|---:|---|---|
| Pinned FULL loader | 18 | pin1 | pin1 |
| Actual original source free call | 18 | pin0 | pin0 |
| Native `mm_free` returns actual -5 | 18 | pin1 | pin0 |
| Temporary extra source pin released | 18 | pin0 | pin0 |
| Original FULL service reports -5 | 17 | pin0 | gone |
| Parent Z/real machine HALT | 8 | gone | gone |

The independent read-only physical audit verifies every stage,
three original source-linked PCs, real source pin/no-false-return,
all MM owner pins/root, no invalid VM publication, checked target
rollback and the terminal user/hardware behavior. SEVEN separately
executed unsafe physical transcript variants are rejected:
original unexpected pin, missing injection, free wrongly
accepting source pin, missing retained pin, no external pin
release, corrupt exact two-unpinned owners, and false-success
FULL return.

## Provenance and limits

Accepted actual original normalized physical PDP-6 trace SHA256:
`4cdba1d83d04a9d92a50a7f13965f389419175cec875f28f58e94f1ebdf97071`.
Original unchanged fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Original unchanged optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Private physical evidence:
`$TMPDIR/daimos-k05-real-free-refusal-v1894-real/`.
Full DAIMOS design audit:
`design/kcore-k05-physical-real-source-free-refusal-v1894.md`.
Testkit implementation:
`tests/validation/pure-k05-real-free-refusal-v1894/`.
Testkit targets:
`make test-pure-k05-real-free-refusal-physical-v1894` and
`make test-pure-k05-real-free-refusal-audit-v1894`.

This closes **transient genuine original MM source-free refusal
with later external pin release and exact two-owner rollback**.
Persistent pin/refusal, target rollback failure, foreign ownership,
permanent allocator/unpin outcomes, terminal cleanup while
quarantined, all PI6 timings, 64K/KA10 and full K05-K09 release
remain OPEN. There were ZERO changes to production code, KCORE,
opt-in PURE MRES, RAM footprint, process fields or on-disk formats.
