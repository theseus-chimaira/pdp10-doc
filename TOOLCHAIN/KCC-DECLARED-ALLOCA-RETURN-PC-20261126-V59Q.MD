# KCC declared alloca and current-frame return PC (v59q)

KCC 8232032; testkit cb4bc4b0. All 22 original link failures
of tests/semantic/upstream/extensions/20010122-1.c are now PASS.
The v31 implementation recognized only undeclared direct alloca
calls. This historical source explicitly declares the size_t
prototype and also uses __builtin_return_address.

The parser now recognizes a direct call to a compatible external
void *alloca(size_t) or __builtin_alloca declaration. It selects
the existing caller-frame allocation generator and AC14 dynamic
frame anchor. Internal or already defined user functions retain
ordinary function identity. Host output uses actual emitted
symbol references rather than suppressing every alloca external
declaration, so taking a function address is not a fake builtin
implementation. The native legacy EXTERN handling remains
unchanged; it has not been independently tested here.

The new N_RETADDR expression reads the real current function
return PC. The private argument reconstruction prologue moves
the return word before counting the fixed frame. For fixed
frames the return word is at -stackoffset(17). For dynamic
frames it is at -(fnframesave+maxauto)(14). HRRZ removes the
saved local PUSHJ flags; the instruction address is converted
to the native S=9 void pointer at the first C byte, consistent
with the GNU label pointer convention. There is no runtime
helper, new permanent frame chain, or extra saved accumulator.
The final candidate recognizes the new terminal in the scratch
prepass, avoiding an unnecessary AC16 save on simple functions.

Only level zero is supported. KCC's normal ABI omits a linked
caller-frame chain. Nonzero levels receive a warning and return
NULL, as permitted by the unsupported-frame contract in GCC
3.2's expand_builtin_frame_address. This is a documented target
limitation, not an implementation of arbitrary stack walking.
The historical test explicitly allows an unavailable level-one
return address. The nonnegative integral constant requirement
is diagnosed; malformed calls fail compilation normally.
No change to the ABI, test source, assertion or acceptance
budget was made. Extended-section return PCs and native
self-hosted compilation have not been verified.

The independent assembly oracle compares the complete returned
pointer to POINT 9 at the exact instruction following PUSHJ.
It covers a leaf, a fixed-frame call with an addressable incoming
parameter, nested calls, two dynamic allocations, actual byte
stores, distinct dynamic call sites and SP restoration. It also
checks the documented unsupported-depth warning/null result.
The final run uses the compiler-only runtime prefix and checks
that no external builtin references appear in the assembly.
All ten base/PDP-6/KA10/KI10/KS10 opt/noopt combinations pass.

Final matrix replay: 22/22 original cells, 88/88 guard cells
(22 diagnosed invalid-level cells, 22 existing alloca lifetime,
22 base VLA, 22 prefix-update), and 22/22 static alloca shadow
cells (direct and indirect user-defined calls). In total 132/132
matrix cells plus ten direct assembly oracle combinations pass.
No whole-matrix or general deeper-frame support claim.

Original link cohort progress: 128/404 confirmed PASS, 276
unresolved or unverified. This does not account for subsequent
ICE-to-link-failure reclassifications. Original sweep unchanged.

Evidence: `$HOME/tmp/kcc-frame-v59q.J7imaOoT`, replay-v1/v2,
exact-pc-v1/v2.log, guards-v1/v2, shadow-v2, build-v2.log,
and kcc.before. Final installed `install-v2/local.tar.gz` SHA256:
`a33c9edcddba6d4607bed4c7e886664d1a3968b9da822edaca49a94077885cd1`.
