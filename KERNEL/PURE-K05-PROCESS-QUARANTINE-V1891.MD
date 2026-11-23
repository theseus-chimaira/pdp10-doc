# PDP-6 K05: real quarantined process metadata after FULL pin2 failure

Date: 2026-11-23. **Scoped actual 48K physical PDP-6 PASS**.
K05-K09 remain OPEN, RAW PURE and mode3 default OFF.

## Physically verified original process state

The unchanged current source-built v1855 48K PURE-on PDP-6 guest
has five original RUN/PIPE children and selected TSFS root. The
source-linked v1890 procedure waits until the original slot3 FULL
loader owns both the once-pinned PACKED source and complete PROCESS
target, then uses a test-only SIMH deposit to change only the target
MM metadata pin count 1 -> 2. The source remains pinned1. With
the original loader and source drive still present, original FULL
returns `MM_ERR_BUSY=-5` at PC `033067` (octal). A real subsequent
permanent KCORE PCLK handler at `017205` verifies unchanged MM
owners/pins and selected root. The entire original v1890 physical
oracle PASSes again inside this v1891 run.

The new physical checkpoint reads the *actual* original KCORE
`proc_table` pointer at permanent physical address `024615`,
observing `044712`. This pointer is checked against the separately
captured original v1890 clock value. The simulator then reads the
original three-word process descriptors at this live address.

* Slot 3 at physical `044723`: **TRANSITION=1**, original state
  `PROC_SRUN=2`, physical VM base 0. The withheld complete target
  is not published as an executable resident APR.
* Slots 4, 5 and 6: original `PROC_SRUN=2`, **TRANSITION=0**, VM
  base 0. Their independent PACKED owner reservations and process
  descriptors remain isolated from slot3's physical uncertainty.
* Parent slot 1: `PROC_SLEEP=3` while awaiting child completion.
  Previously completed slot2 is free.

The testkit independent physical oracle runs the full previous
v1890 MM source-target/root/PCLK verifier first. Four separately
executed mutants are then rejected at their intended original
process-word checks: missing slot3 TRANSITION, false nonzero VM
base, invented slot4 TRANSITION, and forged proc_table pointer.

The original simulator completes its *designated bounded test*
cleanly at the process-table snapshot. It does NOT complete the
parent's native five-child WAIT/Z/HALT workload after fault;
that would be a separate scheduling/progress acceptance requirement.

Original source image SHA256, unchanged:

* Fixed KCORE:
  `33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`
* Optional PURE MRES:
  `a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`
* v1891 original normalized physical process-table transcript:
  `dfdefc29dd3f4665223192754a97693e4425a001b63e7f46459878cbc607e76b`

No source code, fixed KCORE word, optional MRES word, static data,
process field, MM descriptor representation or feature default
was added by this test. The versioned testkit files live under
`tests/validation/pure-k05-physical-overpin-v1890/` with the
`*-v1891*` suffix. The source design evidence is
`design/kcore-k05-process-quarantine-v1891.md`.

## Limits

The preserved physical MM arena contains no free contiguous area
large enough for another **aligned `010000`-word** full PROCESS APR
while slot3's extra pinned target is held. Consequently, the
SRUN/nontransition status of peer slots4..6 does not mean they
can execute or exit. A separate disposable guest attempt to wait
for a new postfault child `w` was stopped without a completion
marker; it is not an accepted test and is not proof of a scheduler
defect. Future physical bystander *user* progress needs an
independent already-resident peer or a rigorously selected larger
memory-pressure fixture that still produces the original FULL
transaction and its exact owner/pin fault.

Other persistent MM source free/unpin/allocator uncertainty,
terminal cleanup, PI6 interleavings and K05-K09 release remain
OPEN. Preserve the v1658 permanent MM quarantine policy; never
blindly free or unpin a potentially unique physical owner.
