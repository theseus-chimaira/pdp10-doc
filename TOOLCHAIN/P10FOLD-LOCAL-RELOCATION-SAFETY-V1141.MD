## v1277: Linux installed-tool regression repaired (2026-11-07)

The source-level v1141 correction described below was committed,
but the independent Linux x86_64 build host still had `p10fold` and
`dlink` binaries installed on 2026-10-13, before that correction.
The Android tools were current. With the same byte-identical test
INIT executable and DAIMOS `226e7a5f`, the Linux v1112 failed-RUN
guest halted after `<` at PC 0 while Android passed `<RILDNWP`.
An isolated 37-commit source bisect found that v1257 added the code
layout which exposed the old folder. Running the actual v1141 native
LOCAL relocation test on the old Linux `p10fold` reproduced the
wrong cross-object fold; it was not a regression in v1257's
stable-u-area scheduler protection.

Rebuilding the clean committed tools exposed an undeclared POSIX
`strtok_r` use under the strict C99/-Werror tool Makefile. The v1277
source update `daimos-tools` `61dcb15` replaces that dependency with
explicit bounded six-tab-field parsing in `p10fold`/`dlink`, rejecting
empty, missing and extra columns. `p10run` examines delimited numeric
words without a reentrant tokenizer dependency. The code paths are
portable and avoid introducing guest resident code or RAM overhead.

Both Android aarch64 and Linux x86_64 now pass the native v1141
object-identity regression and the `daimos-testkit` v1277 test
(`ce5c266`) covering an accepted empty plan and seven malformed plan
mutations. Linux also passed the native v1257 resident/fatal u-area
selector test. The corrected binary package was staged and installed
into `$PDP10_PREFIX` as
`$HOME/tmp/daimos-tools-v1277-clean-r1/local.tar.gz`, SHA256
`938caf5c4e72433a470504c8ee10526abe8707c4724db52601c863566f76d37d`.
Previous Linux binaries were preserved in the same versioned staging
tree under `previous-bin/` for rollback.

The exact current Linux v1112 full PDP-6 guest now passes its
original failed-RUN, invalid BRK, witness and cleanup checkpoints,
and the linked KCORE low endpoint `023267` octal matches Android
exactly. **No production DAIMOS kernel code was reverted or changed**
to work around the linker bug. A fresh corrected-toolchain OOM F11
full acceptance matrix must still pass before production OOM enablement.

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
