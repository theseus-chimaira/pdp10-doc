# PDP-6 K05 v1899: actual source mm_unpin -4 and original child recovery

Date: 2026-11-23. Scoped original 48K PDP-6 full-system physical
**PASS**, including five original children typed EXIT/WAIT,
mutable-data sentinels and actual parent hardware HALT. K05-K09
remain OPEN; RAW PURE/mode3 OFF. No permanent KCORE/PURE code,
RAM, process field or executable-format modification.

## The original FULL source-unpin failure contract

Original `proc_pure_ram_restore_v1391` reconstructs a complete
PROCESS target while retaining the authoritative PACKED RAW
mutable suffix. After v1855 PI-atomic exact-one-pin owner check,
it unpins the temporary target first, then the RAW source.
If `mm_unpin(source)` returns error, the C transaction marks
source ownership uncertain and does NOT issue a speculative
second unpin on that last authoritative RAW extent. Its target
rollback is allowed to release only the exact physically
complete unpinned PROCESS target; the child must remain
TRANSITION/VMbase0 until a safe subsequent restore.

In v1898, both real native TARGET-unpin calls refused -4 after
a test-only target zero-pin deposit and kept both original MM
reservations. v1899 instead makes the ORIGINAL SOURCE native
`mm_unpin` genuinely refuse after the original target unpin
SUCCEEDED, then proves successful target disposal and all-five
child retry/WAIT/HALT on the unchanged original machine.

**Fault-model qualification:** The original v1855 owner guard
and target/source commit unpins execute under PI exclusion.
The normal asynchronous MM client cannot remove the source pin
between the exact owner proof and source unpin; private SIMH
changes the physical source descriptor during a stopped native
instruction. This is an injected unexpected MM state, NOT a
naturally occurring interrupt interleaving.

## Source-linked original PDP-6 execution

The original source-built v1855 48K fixed KCORE and optional
PURE MRES, normal INIT, root TSFS child and four secondary
compressed DTC3-backed executable children run unchanged.
Their original RUN/BRK/PIPE workload, writable sentinels,
typed EXIT/WAIT and parent Z/HALT are independently checked.
The KCORE reclaim JRST physically locates installed PURE MRES
base `032417` octal. The original compiled FULL DOBJ function
symbol `.text+000504`, all FOUR `mm_unpin` and both `mm_free`
relocations are checked against the native linked `PUSHJ`
opcodes. Actual original physical addresses (octal):

* successful target commit `mm_unpin` `035271`;
* source commit `mm_unpin` **`035276`** and native return
  **`035277`**;
* original temporary PROCESS rollback `mm_free` `035371`,
  return `035372`;
* original PURE FULL service return `033067`.

At pinned FULL the MM table contains 18 original extents,
including authoritative PACKED RAW `111200..120000` pin1 and
complete PROCESS target `120000..130000` pin1, owned by slot3.
The normal target unpin succeeds under original PI exclusion.
At the original source-unpin call, physical RAW source is
pin1 and complete target is pin0, MM count18. The private
test controller changes ONLY source metadata word `024703`:

    before: 000014000003 (PACKED, slot3, pin1)
    after:  000004000003 (PACKED, slot3, pin0)

The unchanged ORIGINAL fixed-KCORE native `mm_unpin(source)`
genuinely returns AC1 `777777777774` = `MM_ERR_INVAL=-4`
at the actual return `035277` because its actual source
descriptor is already unpinned. Original MM still owns both
extents, source pin0 and target pin0. The C caller marks its
source pin uncertain, enters the original `fail_target` cleanup
WITHOUT a second source unpin, and lets the real original
fixed-KCORE `mm_free(PROCESS)` release ONLY the unpinned full
target. That original native MM call returns **AC1=0** at
`035372` and physically deletes target `120000..130000`.

The MM count drops **18->17**. Sole remaining slot3 MM owner
is exact PACKED RAW `111200..120000` pin0. Original FULL
service returns -4 at `033067`, preserving slot3 TRANSITION
and VMbase0. The original slot3 process meta at physical
`044723=046031600401` retains TRANSITION, and VM word
`044724=010000000000` has physical APR base zero. The
selected TSFS root `070101000000` octal and unrelated
physical MM extent records/pins are unchanged throughout.

The unchanged original five-child guest retries the child and
finishes original PIPE/sentinel/typed EXIT/WAIT assertions,
prints parent `Z` and actually executes
`HALT instruction, PC: 000000`. Final MM count is 8 with
no remaining slot3 PROCESS or PACKED reservation. This is
normal terminal recovery after a transient injected MM error,
not a fabricated child release or just a resident parent HB.

## Physically measured MM checkpoints (octal)

| Original checkpoint | MM count | Source PACKED RAW | Target PROCESS |
|---|---:|---|---|
| Pinned FULL | 18 | pin1 | pin1 |
| Original source `mm_unpin` entry | 18 | pin1 | pin0 |
| Original source native -4 | 18 | pin0 | pin0 |
| Original target rollback `mm_free` entry | 18 | pin0 | pin0 |
| Original target free returns MM_OK | 17 | pin0 | gone |
| Original FULL -4, TRANSITION/VMbase0 | 17 | pin0 | gone |
| Original five child WAIT and parent Z/HALT | 8 | gone | gone |

The independent read-only acceptance verifies source-linked
native return PCs, original genuine MM_ERR_INVAL, exact owner
and pin state at each physical MM stage, native target free
result and actual deletion, retained RAW source, unchanged
TSFS root/unrelated owners, process TRANSITION/VM0, five typed
child EXIT/WAIT and actual parent hardware HALT. **THIRTEEN
separately executed unsafe physical transcript mutations**
are rejected at intended original MM/owner/pin/return/VM/HALT
invariants.

Accepted normalized original PDP-6 physical trace SHA256:
`32b178432dce4404a405b15cb7a812718115205ad4a119f376256142abc68844`.
Unchanged original fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Unchanged optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Original private simulator evidence:
`$TMPDIR/daimos-k05-source-unpin-refusal-v1899-real/`.
DAIMOS state/design:
`design/kcore-k05-physical-source-unpin-refusal-v1899.md`.
Runnable testkit physical preparation, independent oracle and
negative controls:
`tests/validation/pure-k05-source-unpin-refusal-v1899/`.
Targets `make test-pure-k05-source-unpin-refusal-physical-v1899`
and `make test-pure-k05-source-unpin-refusal-audit-v1899`.

This closes injected original SOURCE `mm_unpin` genuine -4,
verified target cleanup and all-five normal child recovery.
It does NOT prove a lost successful source unpin reply, naturally
occurring MM pin loss, foreign owners, persistent/quarantined
terminal cleanup, other PI6 timing, 64K/KA10 or K05-K09 release.
