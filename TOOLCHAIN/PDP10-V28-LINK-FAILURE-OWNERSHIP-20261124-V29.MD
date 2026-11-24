# PDP-10 compiler linker-failure ownership, 2026-11-24 v29

## Verified scope

The latest exact replay of the *historically failed* v28 subset
executed 4,216 selected compiler/CPU/optimization cells. Its results
were 327 PASS, 2,810 run_fail, 1,029 compile_fail, 50 link_fail,
zero timeout and zero ICE. This is not the outstanding full matrix
sweep and must not be interpreted as one.

The 50 link_fail cells were copied unchanged into the testkit's
versioned `manifests/PDP10-V28-LINKER-FAILURE-CELLS-20261124-v29.tsv`
(SHA256 `2099dc6307fa415d5e6cd6f4369aafe9f00fa067bb1d4cf5de97db46735476a9`).
A new, independent `pdp10-testkit` run with `-j4` executed all 50
against the installed GCC/KCC and returned 50 link_fail, zero PASS,
zero old-green skips, zero ICE and zero timeout. ICE-only exit status
zero does **not** imply linker correctness. The full local evidence
resides in `workdir/known-link-failures-v29/`.

## Ownership by unresolved symbol

| Symbol | Cells | Owner | Required treatment |
| --- | ---: | --- | --- |
| `alloca` | 22 | KCC compiler | Compile a real caller-frame dynamic allocation with proper frame lifetime/unwind; no external fake helper |
| `puts` | 20 | Compiler runtime and chosen OS | Select a real hosted console backend and test in that environment, not in implicit freestanding links |
| `undefined_reference` | 4 | KCC compiler | Do not emit the unused `static inline` function's unreachable external relocation |
| `c` | 4 | KCC preprocessor | Correct macro rescanning of `c(c)(c)` to its static `d` object, preserving standard macro recursion semantics |
| **Total** | **50** | 34 KCC, 16 GCC | Seven unique sources, eight compiler/source groups |

KCC's installed output for GCC historical 20011115-1.c constructs
both identical `$foo` addresses, compares them and retains the dead
`$foo` call. The unused `$foo` function body itself includes a real
relocation to `undefined_reference`. An isolated experimental identity
comparison fold was semantically valid and eliminated the unreachable
branch, but left that function body and relocation. The experiment
was reverted rather than accepting a non-fix. The linker should not
ignore relocations that actually exist.

KCC's preprocess of historical macro3.c yields `if (c(2) != 4)`
where the C99 original intends the static `d = 4` object. The
generated `PUSHJ 17,c` establishes this as a macro-rescan compiler
defect, not a missing runtime function. The original assertions must
stay unchanged.

For alloca, the unchanged 20020314-1.c has a real `alloca(16)` call
within a double arithmetic and function-call path. KCC emits an
ordinary call to an undeclared external and then casts its integer
result to a pointer. Stack allocation and cleanup belong in KCC's
function-frame machinery; supplying a callable symbol would not by
itself satisfy that ABI and lifetime.

The upstream macro/digraph/stringification sources use `puts` on
their failure paths before `abort()`. These paths remain correctly
reachable and must not be removed to hide the missing dependency.
GCC already installs the optional, separately linked
`libp10stdio-daimos-v1.a` implementation backed by DAIMOS's real
`dsys_writechar`; its guest validation is documented in
`TOOLCHAIN/GCC-DAIMOS-CONSOLE-ARGV-GUEST-20261123-V1822.MD`.
That backend is *not* a freestanding console, and the ordinary
testkit bare-machine runner must not silently link it. KCC's
console ABI/provider must likewise be explicitly validated.

## Result and next priorities

**The error taxonomy is complete, but all 50 link failures remain
open.** No source assertions, runtimes, linker implementation or
installed compiler binaries were changed for this analysis. There
is no demonstrated DLINK symbol-resolution bug among these 50.

Prioritize fixing KCC macro rescanning (4) and unused static-inline
emission (4) with exact source-linked and neighboring macro tests;
then implement KCC alloca stack semantics (22), preserving ABI and
frame unwinding. Treat the 20 `puts` failures separately as tests
requiring an explicitly chosen real I/O environment. Any closure
must be demonstrated by a fresh strict test, never an ICE-only exit.
