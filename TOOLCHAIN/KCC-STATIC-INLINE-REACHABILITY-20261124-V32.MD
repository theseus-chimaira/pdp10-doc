# KCC v32: reachability-correct static inline code emission

Verified on 2026-11-24 for the Unix-hosted KCC cross compiler. This
describes an implemented compiler design, not a deployment handoff.
Native DAIMOS KGEN does not yet implement the file-backed emission
mechanism and must be validated separately.

## Problem

The upstream original `20011115-1.c` defines two `static inline`
functions. `bar()` compares the addresses of `foo` to themselves,
returns one for the known-equal case, and contains a fallback call
to `foo()`. `foo()` itself refers to an undefined external symbol.
Previously KCC emitted both internal function definitions eagerly,
even if `foo()` was never reached; it then requested the external
symbol at link time. This created four genuine `link_fail` results
under the KCC PDP-6 and base profiles, in both optimizer settings.

There were three contributing causes, not a linker malfunction:
first, identical function designators were not folded to their known
equal address; second, the unoptimized emitter retained unreachable
statements after an unconditional return; third, the definition
emitter used parser reference counts and wrote all function bodies
before knowing their actual reachability. Removing only the call
did not remove the undefined reference from the unused body.

## Corrected semantics and control flow

The front end now folds same-symbol function-designator `==` to one
and `!=` to zero, after normal binary operand type checking. This
language identity is independent of the `-n` optimizer switch.
The statement emitter recognizes a guaranteed return, jump, break
or continue and does not generate subsequent unreachable statements
unless the suffix contains a user or switch label. It also respects
a compile-time constant if-condition in both optimizer modes.
The bounded label check is critical: an external `goto` may enter
an else arm despite the original enclosing condition being true,
or enter a statement after a previous return. These cases have
their own PDP-10 guest regression with positive executed oracles.

## Deferred function body emission

For Unix-hosted compilation only, static-inline function assembly
is written to a single temporary spool file. Each definition has a
small descriptor: its function symbol, file offset and length,
list of distinct *emitted* symbolic dependencies, and a live flag.
Neither a full AST nor a function-sized assembler buffer remains
resident. The temporary spool file consumes disk, not the native
compiler's constrained PDP-10 address space.

Normal function and global-data emission supply the reachability
roots. When the code emitter writes a symbol reference, it marks
that symbol in the root output or records a dependency on the
currently captured inline function. Definition labels alone do not
make a deferred function externally reachable. The root set is
then propagated through each live candidate's references to a fixed
point; cycles are safe because the live flag is monotone.

At module completion, reachable bodies are copied in source order
to the real assembler output stream. Output section state is saved
and restored when each candidate is captured, and each replayed
body starts with an explicit text-section directive. The assembler
postamble issues `.extern` only for referenced external symbols in
live assembly, not those mentioned by source paths eliminated from
the output. Missing providers required by live code remain real
link failures; none is ignored or given a dummy definition.

This introduces one shared scratch file and a small number of
heap-allocated descriptors per static-inline function. CPU time is
limited to the number of deferred definitions, recorded dependencies
and replayed assembler bytes. Only needed definitions affect the
guest program's code size and runtime memory footprint.

## Evidence and installation

The original four KCC `20011115-1.c` cases now all pass without an
emitted `foo` body or a reference to `undefined_reference`.
`kcc-static-inline-reachability-v32.c` passes 22/22 KCC guest cells
testing transitive calls, a function pointer initializer, mutual
recursion, preservation of a real external `abort` dependency and
absence of a dead undefined provider. The new
`kcc-inline-branch-labels-v32.c` also passes 22/22, including
goto targets after a return and inside an otherwise excluded else
branch.

The completed, source-pinned broad acceptance selection contains
15 sources and **238/238 fresh KCC PASS** across PDP-6, KA10, KI10
and KS10 native/base/PDP10 families and both optimization modes.
All cells were rerun after installing the compiler at
`$HOME/cross/bin/kcc`, with `-j4`, strict exit policy, zero skipped
old-green checkpoints and no user assertion modifications.

The installed compiler has SHA256:
`aaeb1d409dabc1c5ade71c36f763cca2727cbd86050cc5fc69b986227c160017`.
The v31 binary was saved as
`$TMPDIR/pdp10-kcc-inline-v32/kcc-before-v32`.
The core freestanding `libp10c-v2.a` remains SHA256
`ac09b8a362e4ea3ca8a7e69a2ca5a98af5a76cc85a7db909b6b3a134afb7ee41`.

The historical 34 exact KCC linker-failing cells now yield 30 PASS
and four genuine `puts` link failures, both in a staged replay and
in the independent final installed-compiler replay. The latter ran
all 34 original cells afresh, with zero old-green skips, in
`pdp10-testkit/workdir/kcc-known-link-independent-final-v32`.
The remaining failures require a selected real console, not a
compiler change or a fake bare-metal stub.

The testkit evidence is saved under
`history/PDP10-KCC-STATIC-INLINE-REACHABILITY-20261124-v32.md`;
source-pinned acceptance manifest:
`manifests/PDP10-KCC-STATIC-INLINE-ACCEPTANCE-SOURCES-20261124-v32.txt`.
The original-cell selection remains the v31 KCC manifest.

## Important open boundaries

Native DAIMOS KGEN has a separate file-service and 64K-word
address-space constraint. The Unix `tmpfile()` mechanism has not
been ported to its native bootstrap; native symbol-emission closure
must use an actual supported temporary-file interface and receive
independent machine tests. The source intentionally compiles the
new spool logic only under `HOST_UNIX`.

The upstream `20001205-1.c` uses GNU extended assembly constraints,
which both KCC v31 and v32 reject at the parser stage; 22 historic
cells remain outside this strict static-inline acceptance. An old
testkit codegen assertion incorrectly required the public `helper:`
label for a static function; it now checks the real `$helper:`
internal label and passes on both v31 and v32 without loosening the
warning counts.
