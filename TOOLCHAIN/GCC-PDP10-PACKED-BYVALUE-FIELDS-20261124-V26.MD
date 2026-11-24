# GCC PDP-10 packed 9-bit by-value parameter fields v26

Status: implemented and target-executed on 2026-11-24.

## Why the original code failed

GCC 3.2's expansion of a packed structure passed by value could
represent each 9-bit `unsigned char` formal-parameter field as a
full SImode MEM at the same containing-word address. Metadata
retained the field's distinct `p.a0`, `p.a1`, `p.a2` identity and
9-bit storage size/alignment, but the actual RTL memory operands
were indistinguishable to CSE.

At O0 the backend used MOVE/ADD on the entire 36-bit word, rather
than extracting its three 9-bit fields. At O1/O2, common-subexpression
elimination could additionally replace several distinct field
loads with one, so correcting the final printed instruction alone
could not repair the optimized compiler.

## Early RTL correction

The GCC `movsi` and `addsi3` expansion paths now recognize a
promoted exact 9-bit COMPONENT_REF rooted in a by-value PARM_DECL,
with an SImode MEM describing one 9-bit unit at 9-bit alignment.
The containing stack-word address is already the native PDP-10
word address. The FIELD_DECL gives the field's bit position.

Before CSE, the backend expands that access as a distinct
`ZERO_EXTRACT` or `SIGN_EXTRACT` of a containing-word MEM. The
full-word MEM does not inherit the narrow field's misleading
metadata, aliases conservatively and retains volatile semantics.
Consequently field identities remain distinct throughout GCC's
optimization passes.

When late combine creates a new direct SImode ADD/MOVE memory
source, final output uses a correctly positioned nine-bit LDB.
Signed fields receive the normal PDP-10 9-bit sign extension
(`TRNE`/`ORCMI`) before arithmetic. The corresponding instruction
lengths are described accurately in `pdp10.md`, including the
three- or four-instruction signed fallbacks.

This is a target backend repair, not an ABI adapter or test-side
workaround. It introduces no guest runtime helper, data storage,
permanent RAM or extra caller-visible argument slots. Wider and
cross-word fields and unrelated pointer dereferences are not
silently reinterpreted.

## Acceptance boundary

The unmodified original `gnu-packed-byvalue.c` was failing all
44 GCC profiles after the earlier v23 copy-loop termination fix.
With v26 it passes **44/44**. The new registered C89 target
regression explicitly checks 3, 6, 11 and 20-byte packed by-value
arguments and signed negative 9-bit fields. It failed with the
previous compiler but now passes **36/36**. Combined official
strict acceptance: **80/80 PASS**, no old green checkpoint skipped.

Eight adjacent registered GCC test groups (v17-v23 and historical
wide-integer/control-flow tests) also passed **312/312** cells under
strict exit policy. Total isolated v26 verification: **392/392**
fresh GCC PASS cells without skipped previous successes.

Installed `cc1` SHA256:
`752a3c6a33e112dddcb83822c59a33222f8b8017163229d30550a9a4d055fbea`.

The new compiler is installed under `$HOME/cross`. Tests and
regression provenance are in
`pdp10-testkit/history/PDP10-GCC-PACKED-BYVALUE-FIELDS-20261124-v26.md`.
Previous adjacent packed-pointer and block-copy repairs remain
separate, as do the outstanding KS10 optimized byte-pointer cases
and any unrelated full-matrix failures.
