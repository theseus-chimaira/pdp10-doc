# KCC induction lifetime and return ABI (v59w)

KCC d15edae; testkit e09b9be7. Follow-up to v59p and v59v.
The eleven optimized failures in pdp10-prefetch-invalid-address-v1.c
were independent of prefetch: the first initialization loop kept
a derived array address in an ordinary VREG across a ternary.
gternary spilled it and later code restored it to another physical
AC. The emitted back-edge incremented a different AC from the one
live at the loop head. The no-prefetch semantic regression reproduces
the old compiler failure at assertion 1.

Before generating the prologue, scan eligible statement children
with a bounded visit count and recursion depth. If a preserved AC
is available after normal register allocation (and the VLA anchor
reservation), reserve it for the function's induction loops. Keep
the derived pointer outside the VREG allocator and generate copies
from the fixed physical AC. Initialize it before the loop head and
increment that same AC on the back-edge. Sequential eligible loops
reuse the reserved AC. The prologue and ordinary/tail epilogues
include the one extra saved word; no parser-tree mutation or target
static storage is needed. When no preserved AC is free, the ordinary
indexed address generator applies. The original narrow selection
policy remains: no nested active loops, explicit calls or general
statement-level joins. Require full-word local nonvolatile IVs with
no escaped address, and reject opaque assembly, statement expressions
and declaration/dynamic-storage nodes. No new optimization scope.

The independent ABI caller also exposed an older VLA return defect:
promote_abi_params runs after return nodes are parsed. A function
saved a newly allocated AC10 but a direct return used the earlier
Nreg count, so AC10 retained the argument value rather than the
caller's preserved value. Use the final _reg_count plus the optional
induction AC on every return. This applies even with optimization
disabled. The old compiler fails the raw noopt VLA probe at call 3
with AC10=3 instead of its sentinel. The final compiler passes
normal, tail and VLA returns, including exact SP and AC10..AC16;
the actual optimized tail target checks registers before entry.
The noopt ordinary nested-call target checks only the outer caller's
registers after return, not its callee's legitimate working values.

Final acceptance: 260 distinct selected matrix cells, 252 PASS,
8 pre-existing policy_fail. All 246 semantic cells PASS:
132/132 prefetch companion/original cells, 22 new induction-lifetime
cells (ternary, register pressure/exhaustion, tail/early return,
VLA frame and global IV), 22 alloca lifetime, 22 shared tail epilogue,
22 base VLA, 22 register prefix-update, and 4 v1395 induction-call
cells. Six optimized loop-induction codegen checks PASS; the six
noopt loop policy checks and two preserved-BLT policy checks remain
failures as in the original sweep. The positive loop codegen check
now requires a preserved induction AC, not merely an ADDI. The
negative checks are unchanged. Existing v1395 call-safety assembly
test PASS (full and generator-only modes). The independent assembly
ABI suite passes 8 guest runs, 24 calls across PDP-6, KA10, KI10 and
KS10 opt/noopt. No regression of selected original passes. No new
target runtime helper or changed oracle/acceptance limit.

One saved stack word per function with eligible loops; no peak
total stack, execution timing, native self-host or extended-mode
claim. This repairs additional semantic/ABI defects, not original
link cells: the original 404 link cohort stays at 184 confirmed
runtime PASS, 24 corrected registrations, 196 unresolved/unverified.
No full-matrix sweep has been performed.

Evidence: `$HOME/tmp/kcc-induction-v59w.G80yx7q1`, replay-v2,
guards-v4, extra-v4, abi-v5.log, abi-before/run.log, before-regression,
call-safety-v3, build-v4.log. Installed compiler-only archive
install-v4/local.tar.gz SHA256:
`1c60c929514f17743ac0bc827359195a3258fa8e58b63b46513df1070250e548`.
