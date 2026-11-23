# PDP-6 K05 v1897: completed target rollback free, lost success reply

Date: 2026-11-23. **Scoped original 48K PDP-6 physical PASS**,
with original five-child EXIT/WAIT and real machine HALT.
K05-K09 remain OPEN; RAW PURE/mode3 OFF by default. No permanent
KCORE or removable PURE MRES code/RAM change.

## Original FULL owner contract

The original v1839 source-free reconciliation uses the physical
MM extent table to distinguish a PACKED source free that really
succeeded despite an error reply, a genuinely refused source
free with two exact unpinned owners that permits checked target
rollback, and ambiguous owner sets that must remain quarantined.
The original `fail_target` cleanup then calls the actual native
`mm_free(target,MM_TYPE_PROCESS,slot)` to release the complete
temporary PROCESS APR. A failed reply from THAT call does not
prove whether target deletion committed. The original C returns
`MM_ERR_BUSY`, retaining the RAW PACKED suffix and TRANSITION,
without publishing a speculative APR or freeing another owner.

v1896 physically proved the target free GENUINELY refused because
the target had a persistent extra pin. v1897 physically proves
the opposite completed-deletion case with ONLY a falsified
postcommit return. The unchanged original guest must be able
to retry, reconstruct the process again, and complete all five
original child WAIT operations without leaking the target.

## Source-linked original PDP-6 execution

The original source-built v1855 48K PDP-6 original KCORE/PURE
images, parent INIT, five child executables and secondary DTC3
media are reused, including one TSFS-root-backed child and four
compressed secondary-backed children with original RUN/BRK/PIPE
pressure, writable-data sentinels and typed EXIT/WAIT assertions.
The original installed PURE base `032417` octal comes from the
actual running KCORE reclaim JRST, not a guessed address.
Compiled original FULL DOBJ symbol `.text+000504` and `mm_free`
relocations `.text+000674` and `.text+000755` are checked against
the original linked native `PUSHJ 17,mm_free` instructions:

* original bounded FULL loader `036202` octal;
* real PACKED-source free call `035310`, return `035311`;
* real PROCESS-target rollback free call **`035371`**, return
  **`035372`**;
* original FULL service return `033067`.

At original FULL pinned stage, MM count18 includes slot3 PACKED
RAW `111200..120000` pin1 and separate complete PROCESS target
`120000..130000` pin1. The original FULL completes data loading
and unpins both; one test-only external source pin makes the
original source `mm_free(PACKED)` genuinely refuse with its OWN
`MM_ERR_BUSY=-5`. The controller releases that external pin;
original v1839 exact MM scan then sees both original owners
complete and pin0, and enters real `fail_target` cleanup.

At actual original target free call `035371`, the full target
`120000..130000` is exactly pin0. The original unmodified fixed
KCORE `mm_free(PROCESS)` is allowed to execute normally. At the
actual return `035372`, native AC1 is **0** (`MM_OK`), target
PROCESS has been **physically deleted**, original MM count falls
from 18 to 17, and the only surviving slot3 MM owner is exactly
PACKED RAW `111200..120000` pin0. Every unrelated owner/pin and
selected TSFS root `070101000000` octal remain unchanged.

ONLY AFTER this physical proof does the private test controller
deposit AC1 `777777777773` (= `MM_ERR_BUSY=-5`); no original
code, MM descriptor, source, media or user memory is changed.
The original FULL service returns that failure at `033067`,
with original slot3 process descriptor `044723=046031600401`
still TRANSITION and VM word `044724=010000000000` physical
base0, retaining the sole PACKED RAW source (MM count17).
There is no invalid runnable APR publication and no speculative
second target deletion.

The unchanged original user workload then retries the child,
executes all original five typed EXIT/WAIT and PIPE/sentinel
checks, emits parent `Z` and physically HALTs at
`HALT instruction, PC: 000000`. The final original MM table
contains 8 entries, none of them remaining slot3 RAW PACKED
or PROCESS allocations. Original selected TSFS ROOT never
changes across the failed reply/retry/terminal sequence.

## Actual physical MM stages (octal)

| Original checkpoint | MM count | slot3 PACKED source pin | slot3 PROCESS target pin |
|---|---:|---:|---|
| Pinned original FULL | 18 | 1 | 1 |
| Original source-free invocation | 18 | 0 | 0 |
| Genuine original source free -5 | 18 | 1 | 0 |
| External source pin released | 18 | 0 | 0 |
| Original target-free invocation | 18 | 0 | 0 |
| Original target free SUCCESS at `035372` | 17 | 0 | gone |
| Only AC1 modified to false -5 | 17 | 0 | gone |
| Original FULL returns -5, TRANSITION/VM0 | 17 | 0 | gone |
| Five children reaped; original HALT | 8 | gone | gone |

## Independent audit and limitations

The read-only physical verifier checks both original compiled
and linked `mm_free` sites, original source/target type/owner/
size/base/pin, genuine PACKED source free refusal, unpinned
pair before target cleanup, original successful target free
AC1=0 and ACTUAL deletion, sole authoritative RAW source,
test-only lost reply AC1=-5, original conservatively failed
FULL service with TRANSITION/VM0, unchanged unrelated MM/root,
five typed child EXIT/WAIT, terminal owner cleanup and HALT.

**TWELVE independently executed unsafe physical transcript
mutants** are rejected at exact original source/pin/free,
process/owner, terminal MM or HALT checks. The first negative
oracle incorrectly expected the MM count only once per
original physical snapshot. Correcting this test-only duplicate
count parser left the original physical trace and all original
source-linked checks intact.

Accepted original normalized PDP-6 transcript SHA256:
`6aeb9da4b13d471ca300b2091b4b663be867f24c3e54850579287f140fa6d11e`.
Original unchanged fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Original unchanged optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Private real physical evidence:
`$TMPDIR/daimos-k05-target-free-lost-reply-v1897-real/`.
Full DAIMOS design/state:
`design/kcore-k05-physical-target-free-lost-reply-v1897.md`.
Executable physical testkit:
`tests/validation/pure-k05-target-free-lost-reply-v1897/`.
Testkit targets:
`make test-pure-k05-target-free-lost-reply-physical-v1897` and
`make test-pure-k05-target-free-lost-reply-audit-v1897`.

This closes the exact original target `mm_free(PROCESS)` COMPLETED
but success reply LOST case, including eventual normal child
retry/WAIT/HALT. This does NOT prove foreign MM owner states,
permanently failed unpins, terminal cleanup of still-quarantined
processes, arbitrary PI6 timing, 64K/KA10 or K05-K09 release.
