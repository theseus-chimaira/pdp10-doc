# PI6-only MM compaction and PI5 completion-wakeup preservation v1747

Date: 2026-11-21. DAIMOS commits `d475fcd3` and `cbcd7a7e`;
testkit commits `825c5ac` and `aca082a`.

## Failure mechanism and ownership contract

`mm_move_extent` previously held `mach_pi_disable` (global PI off)
while calling `vm_extent_move` to copy a complete process image.
As in the earlier allocator/BRK root DRL failures, the long interval
could starve Type 136 DCT PI3 data service and Type 270 PI5
completion. This was a source-derived risk; it was not necessary to
invent a second physical DRL reproduction to justify the correction.

The v1745 MM code inhibits only scheduler PI6 across descriptor
validation, temporary removal/reinsertion, VM copy, and publication.
On normal and typed-refusal returns it restores PI6 only if the
original CONI PI snapshot had PI6 enabled. Both impossible MM owner
loss HALTs explicitly disable global PI before HALT, preserving
the original distinct fail-stop PCs and error classifications.

## The separate PI5 wakeup race

Leaving PI5 enabled across physical relocation uncovered an actual
sleep/wakeup race. `vm_extent_move` stored the original SLEEP state,
temporarily marked the process STOP, and on completion blindly
restored the old SLEEP state. A PI5 disk-completion wake occurring
while the image was being copied cleared the process's wait field,
but did not make the STOP process runnable. The old implementation
could therefore strand a process whose I/O had completed.

The v1745 VM code inhibits PI5 only around the small initial and
final process-state/runqueue transitions. It leaves PI3 available
even there and PI3/PI5 available for the full physical copy. At
commit it checks the current wait field: when the original state
was SLEEP but PI5 has cleared the wait, the restored state is RUN
and the process is enqueued exactly once. Otherwise it restores the
original state. The PI5 enable state is restored exactly as entered.
This uses one transient stack word during each short transition,
with no new permanent or per-process RAM allocation.

## Acceptance

The actual pre-fix `vm_extent_move` failed a source-linked native
PDP-6 wakeup test with exit code 5; the repaired code passes with
PI3/PI5 verified on during copy, PI5 inhibited during transitions,
and correct PI5 restoration. The v1745 MM pin-race test passes and
rejects four unsafe mutants. The full K01 native matrix passes
34/34 cases, including owner, capacity, rollback, original
MM-loss HALT, mapping and runqueue tests.

The **full physical 256K PDP-6 KCC compaction test v1747 passes**:
two `mm_compact` breakpoints (octal `002030`) and two actual
`vm_extent_move` entries (octal `023416`), complete guest child-data
verification, the `CQ` completion marker and HALT. Exact compiler,
source, guest transcript and SHA256 provenance are in
`daimos-testkit/tests/validation/
daimos-mm-compaction-pi6-physical-accept-v1747.md`.

A separate legacy GCC v561 fixture halted at PC zero after device
initialization and before reaching compaction; that run was not
reclassified as a pass. The new test uses the current installed KCC
and unchanged guest source/oracle. A full KCC 256K ROOT/DSH/TRUE/HALT
guest also passed after the code changes.

## Remaining work

No functionality, K04 policy or process accounting was removed. The
change adds executable instructions for the necessary PI5 state
transitions but no permanent data. Further RAM/code-size analysis
should quantify the increment before the strict 32K KCORE budget
can be accepted. The other short MM PI masks and K04 permanent
provider refusal policy remain separate open work.
