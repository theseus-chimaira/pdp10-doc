# KCC constant-expression query (v59t)

KCC 5058650; testkit a0ce2cf8. The two original link_fail
cells of tests/semantic/upstream/extensions/bcp-1.c referenced
func, which is deliberately declared but never defined. KCC
treated __builtin_constant_p as an ordinary undeclared function
and generated its argument's call rather than querying constancy.

Recognize the direct undeclared GNU builtin in primary(). Parse
one assignment expression with constant-expression restrictions
temporarily suspended, apply ordinary array/function conversion
and the existing expression folder even in noopt compilation.
Recognize resulting integer, floating, numeric pointer and
string constants. Other expressions conservatively return zero;
relocatable object addresses are not numeric constants. Replace
the whole operand tree with an integer literal, not a call or
runtime helper. Restore constexpr, stackrefs and ra_expr after
parsing. A nested query is supported. A parser nesting counter
prevents unevaluated alloca calls from reserving a dynamic frame.
Ordinary alloca calls keep the established lifetime machinery.
Missing or extra arguments produce ordinary diagnostics.

The original two failing cells pass and the unchanged historical
source passes all 22 applicable KCC profiles. The GCC-specific
__OPTIMIZE__ section is not exercised by KCC, which does not
define that macro; no claim of inline propagation or optimized
literal subscript inference. Returning zero when the front end
cannot prove constancy is permitted. Explicit builtin declaration
handling, native legacy EXTERN emission and all possible parser
bookkeeping for GNU statement expressions inside the query have
not been independently verified. Host output uses actual emitted
references, so discarded trees require no external provider.

The new kcc-constant-query-v59t fixture checks enumeration and
static initializers, folded arithmetic, wide and FP literals,
strings, sizeof, volatile operands, relocatable addresses,
unresolved objects/functions, increments, assignment, comma,
short-circuit and nested queries, and alloca in an unevaluated
operand. Old KCC fails its constant initializers. New KCC passes
22/22. Malformed query arity is covered by 22/22 negative cells.
All 44 generated assemblies from the semantic query fixture and
historical source were checked for unwanted builtin or external
operand references; none found. Alloca-lifetime, static alloca
shadow and prefetch side-effect guards pass 66/66. Final distinct
selected cells 132/132 PASS (includes the historical 22; the
initial exact two-cell replay is a duplicate). No source oracle,
acceptance limit or runtime provider changed.

Original 404 link failures: 130 confirmed PASS, 24 invalid
registrations corrected, 250 unresolved/unverified. This is not
a fresh full-matrix result. GCC optimization-dependent builtin
semantics, extended mode and native self-host remain unverified.

Evidence: `$HOME/tmp/kcc-constant-p-v59t.BCL2FWz7`, replay-v1,
regression-v1, guards-v1, before-regression/compile.log,
build.log, kcc.before. Installed compiler-only local.tar.gz
SHA256 `2f1665f08a247f74298fb7da257a2d2301947095777c8fda0adf8d6a47d780fa`.
