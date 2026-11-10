# KCC symbolic ADDI and constant-index safety (2026-11-10, v1392)

## Failure and ownership

During staged DAIMOS PURE RAM restore acceptance, an optimized native
test using fixed slot `swap_slots[1]` unexpectedly changed the saved
executable backing key for mount 18. Its generated PDP-6 code
contained an instruction of the form:

```asm
    movem 7,swap_slots(1)
```

Here `(1)` is a hardware **AC1 index register**, not the intended
C array subscript. The valid constant destination is:

```asm
    movem 7,swap_slots+1
```

The original register value required by the indexed form was no
longer available on the executing path. PURE DROP was not the
operation corrupting backing ownership: the wrong store was emitted
in the optimized test setup before DROP entered. This is a KCC
peephole compiler defect. Changing DAIMOS array syntax, weakening
the mount guard or forcing unoptimized kernel builds would only
hide it.

## Root cause and v1392 correction

`kcc/ccopt.c` `foldplus` converts a symbolic `ADDI R,SYM` to
`MOVEI R,SYM(R)`. That transformation is useful for genuinely
dynamic register indices. However, when R actually holds an
immediate constant, a later independent peephole transformation
can remove the corresponding register definition, leaving a
self-indexed symbolic effective address dependent on stale AC
contents.

The v1392 correction retains the original dynamic form unless a
local backward scan finds an unskipped immediate MOVE defining R,
with no intervening register modification, skipped instruction,
call, jump or conditional control-flow barrier. In that provable
case it folds the constant into the 18-bit symbolic offset and
sets the index register to zero. The original definition is not
unconditionally deleted; normal use/liveness handling remains
responsible for it. This is a targeted proof-based transformation,
not a blanket disabling of symbolic address folding.

The change adds no compiler static data table, target runtime
helper, kernel call or permanent DAIMOS storage word.

## Tests and limitations

The compiler was built with `-std=c99 -funsigned-char -O2`,
reproduced byte-for-byte from a fresh release build, and installed
in `$PDP10_PREFIX/bin/kcc`. Its previous binary was retained
outside the source tree for rollback. Acceptance includes 21
focused DAIMOS PURE/EXEC host and native PDP-6 test scripts and five
selected KCC ABI/runtime/parser/float scripts. The source-linked
`daimos-testkit/tests/validation/kcc-symbolic-address-fold-v1392`
also checks emitted addresses and process backing identity.

Two distinct native fixture issues were isolated while testing:
an older mock VFS reader previously wrote three header words
for a two-word request, and new v1391-only VFS metadata cases
changed the historical v1387 test layout when compiled
unconditionally. The former was corrected before v1390; the
latter is now gated to FULL-loader test builds. Baseline v1387
passes under both the original and corrected KCC.

**Acceptance limit:** The exact wrong indexed store was observed
in the earlier larger native diagnostic fixture. The cleaned
versionized regression does not reproduce that miscompile under
the old binary, so a minimal, independently executable
red-old/green-new reproducer remains outstanding. The complete
PDP-10 compiler matrix (over 100,000 enumerated combinations) was
not run. A separate broad PDP-10 verification sweep was halted
after an AAP batch50 emulator timeout waiting for CPU status;
that check is not counted as passing and has not been attributed
to the symbolic-address correction. None of these tests
establishes production DAIMOS KCORE integration or a
resident-memory saving.
