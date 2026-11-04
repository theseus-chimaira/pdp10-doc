# P10FOLD/DLINK local relocation identity (v1141)

## Failure and invariant

DOBJ1 `LOCAL_RH18` and `LOCAL_LH18` relocations designate a section
and an offset **within the containing object**. Equal numeric offsets
in two different objects do not designate the same eventual address.
The older P10FOLD comparison treated them as equivalent, and DLINK's
fold-plan validation repeated the same mistake. The resulting plan
could redirect a branch to the other object's unrelated code.

## Matching rules

For proposed duplicate TEXT spans of equal length:

- If both local TEXT targets fall inside their respective spans,
  compare their offsets relative to the span starts and the high-half
  addends. The retained span can satisfy both relocations.
- If exactly one local TEXT target is internal, reject the match.
- For any other local section relocation, require the same input
  object and the same section/addend. An equal offset across objects
  is insufficient, including for DATA and BSS.
- Named symbol relocations retain their separate symbolic-name and
  addend comparison.

DLINK checks the same rules independently before accepting a fold
plan, including a user-supplied or stale plan. This is a correctness
constraint; it must not be bypassed to recover a few text words.

## Regression and DAIMOS impact

`daimos-testkit` target `test-daimos-p10fold-local-section-v1141`
assembles two objects with a two-instruction block and distinct
external-to-block local TEXT destinations at equal numerical offsets.
The old P10FOLD wrongly emits a cross-object fold; the new analyzer
does not. The old linker also accepts an injected invalid fold plan;
the corrected linker rejects it at relocation validation.

In the DAIMOS nested 256K PDP-6 APR/EXEC/failed-RUN regression, the
old toolchain folded `proc_discard_ok` (proc_pdp6 text offset 01217)
onto a sequence in syscall_dispatch_pdp6 (offset 01117). Both
sequences began with `SETZ 1` but their `JRST` relocations targeted
distinct object-local code. With v1141 tools the fold is absent and
the uninstrumented nested full-system guest passes its parent RUN
rollback and final WAIT. The historical EXEC/FILE/WAIT control also
passes. This correction adds no kernel source ABI or persistent data;
any resident TEXT delta must be measured from the corrected link.

The production requester-only OOM switch remains disabled. Broader
physical MM_BUSY and full-system activation gates are separate.
