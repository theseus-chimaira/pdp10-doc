# PDP-6 K05: real resident user progress after quarantined FULL v1892

Date: 2026-11-23. **Scoped original 48K full-system physical PASS.**
K05-K09 OPEN; RAW PURE and mode3 OFF by default.

## Missing evidence after v1891

The original PDP-6 v1855 FULL commit guard physically refused an
artificial second MM pin on the complete destination at source-linked
FULL service return, preserving the authoritative PACKED source
and the temporary PROCESS target. The v1891 physical process table
showed actual slot3 TRANSITION with VM base zero, and unaffected
slots4..6 nontransition. Their PACKED images could not fit another
aligned `010000`-word full APR in the original 48K residual gap;
it was therefore invalid to infer resumed user execution from an
unaffected scheduler descriptor or a later PCLK interrupt alone.

## Already-resident original parent as autonomous bystander

v1892 rebuilds the same original five-child 48K pressure fixture
with **only a testkit INIT source change**. Everything through five
successful RUN admissions, refused sixth RUN, BRK expansion,
PIPE, writable-data sentinel checking, native compressed secondary
DTC3 file creation, TSFS namespace and `L` pressure marker remains
identical. After the original `dsys_close(wr)` release the parent
(slot1) executes a test-only `dsys_sleep(0100U)`/sentinel-check/
`dsys_writechar('H')`/`dsys_writechar('B')` loop in its **already
resident** PROCESS APR. It does not block forever in WAIT for a
known irrecoverable child. There is no privileged new syscall,
kernel instrumentation, new user process or new VM allocation.

The source-built *permanent* KCORE and *removable* PURE MRES images
are byte-identical to accepted v1855/v1890/v1891:

* Fixed KCORE SHA256
  `33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
* Opt-in PURE MRES SHA256
  `a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

The test-only INIT user DXR SHA256 is
`620fcfaddedd5a4abea8b35e11a99e371fa624420ce479a4e4de69fb92c24e27`.
Its image text changes from `001246` to `001222` octal words
(20 decimal fewer), BSS stays `004001`. The complete mapped parent
APR remains `010000` octal words. This is NOT a production code or
RAM optimization; no production KCORE/MRES word changes.

## Physical source-linked sequence and precise active user

The freshly booted KCORE reclaim JRST still yields installed PURE
base `032417` octal, bounded FULL entry `036202`, and actual service
return `033067`. After the five-child RUN/BRK pressure, slot3
owns separate once-pinned PACKED suffix `111200..120000` and full
PROCESS target `120000..130000`. Only the private SIMH controller
changes its real MM target pin count from 1 to 2 by depositing
`024705: 000011000003 -> 000021000003`, with both original physical
words actually examined. The complete compressed source on
independent DTC3 stays attached; the production loader succeeds,
but original FULL returns `MM_ERR_BUSY=-5`. An actual fixed-KCORE
PCLK PI6 at `017205` and entire MM/process snapshots verify
slot3 TRANSITION, VM=0, RAW source pin1 and temporary target pin2.

ONLY THEN does the test controller remove the PCLK breakpoint and
arm live `expect "HB"` on the original guest terminal. The first
new parent heartbeat stops the simulator. The controller re-arms
the same user-output expectation and requires a SECOND genuinely
new heartbeat after further original clock ticks. The complete
source-linked original hardware snapshots show (octal):

| Field | First new HB | Second new HB |
|---|---|---|
| `proc_current_slot` | 1 | 1 |
| Parent `PROC_SRUN` | 2 | 2 |
| Parent VM base | `130000` | `130000` |
| Parent PROCESS MM owner | `130000..140000`, pin0 | same |
| Slot3 TRANSITION / VM base | 1 / 0 | 1 / 0 |
| Slot3 PACKED RAW suffix | `111200..120000`, pin1 | same |
| Slot3 provisional full PROCESS | `120000..130000`, pin2 | same |
| Original `proc_timer_clock` | `021610` | `021710` |

All other original MM extents and selected TSFS root match the
earlier physical postfailure/PCLK owner snapshot. The two fresh
user output markers and `0100` octal subsequent clock ticks
demonstrate **actual user-mode parent process forward progress**
with another process permanently quarantined. The physical current
slot itself proves the active code belongs to slot1; this is not
inferred from scheduler eligibility or a clock handler alone.

## Negative controls, physical provenance, limits

The initial physical v1892 controller retained the old PCLK
breakpoint, so its supposed heartbeat stops were ACTUAL PCLK
traps. The strict heartbeat oracle REJECTED that experiment;
only the testkit controller was corrected by inserting `nobreak`
before the two `HB` watches, then the identical production binaries
and test-only INIT were run again. The second actual original
SIMH run, source-linked verifier and complete read-only transcript
audit PASS. SIX separately executed unsafe physical trace mutations
are each rejected: a missing second real HB, missing resident
parent, wrong current user slot, lost slot3 TRANSITION, freed/raw
source pin, and a clock that did not advance between user HBs.

Accepted ORIGINAL physical transcript SHA256:
`7fbc1e9fa203c2c146861d0f951e222f22721f221e3f095c300b23d96ca72f86`.
Original source-built 48K image and private SIMH evidence:
`$TMPDIR/pure-brk-pressure-v1614.UQzCQO/`,
`$TMPDIR/daimos-k05-resident-peer-v1892-physical-v2/`.
Reproduction and audit Makefile targets:
`test-pure-k05-resident-peer-build-v1892`,
`test-pure-k05-resident-peer-physical-v1892`, and
`test-pure-k05-resident-peer-audit-v1892` in daimos-testkit.
Full design: `design/kcore-k05-resident-parent-v1892.md`.

This settles **independently resident parent user progress**,
NOT a different PACKED secondary child restoring or exiting,
full parent WAIT/Z/HALT after the fault, terminal cleanup,
other permanent source unpin/free or allocator failures,
different PI6 interleavings, KA10 or production PURE acceptance.
K05-K09 remain OPEN; default RAW PURE and mode3 remain OFF.
