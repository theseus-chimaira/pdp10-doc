# KCC GNU label byte coordinates (v59l)

KCC 832b3fa; testkit d3816302. All 22 original KCC cells of
920302-1.c timed out in the November 25 sweep. Runtime N_ACONST
and static gizptr(N_ACONST) materialized raw word addresses,
despite the GNU label value type being void *. The existing GNU
void-pointer arithmetic computes differences in C-byte units
and uses ADJBP for addition. The raw word address has no valid
byte-pointer size field, so the reconstructed target is wrong.

Both runtime and static label values now designate the first
9-bit C byte of the instruction word. The static initializer
uses the ordinary byte-pointer relocation and runtime generation
uses code10(P_MOVE,...,TGSIZ_CHAR,0). Local indexed JRST consumes
the right-half word address. This is consistent with the existing
GNU byte-unit arithmetic and does not change general void-pointer
casts, the function-pointer ABI, or add a runtime helper or
persistent storage. This repair is for native local addressing;
extended-section label arithmetic is not established by this test.

The original unmodified 920302-1.c passes all 22 selected cells.
The same replay includes 38 existing GNU void-arithmetic and
object pointer round-trip runtime cells (60/60 total). Three
existing direct/offset/identity label regressions are enabled
for all 22 KCC profiles each and pass all 66 cells. The new
kcc-label-byte-offset-v59l fixture checks noncoincident labels,
both signs of short offsets, static label-plus-four relocation,
round trips and a bounded indirect dispatch. The baseline
compiler fails its first identity check (guest exit 1), before
any indirect jump. All 22 repaired fixture cells pass. Total
148 selected runtime passes, no failures. The previous identity
fixture alone is insufficient because its labels may coincide.
No NO_LABEL_VALUES bypass, changed historical source, or raised
timeout/step limit in the acceptance replays.

Evidence: `$HOME/tmp/kcc-label-offset-v59l.ir3eG9bj`.
replay-v1, label-replay-v1, regression-v1 and before-regression.
Installed compiler-only local.tar.gz SHA256:
`434387b5b6ff7ac400b158014df585b201cdc661194330f4e95e21d4329d8529`.
Original sweep is unchanged. No whole-matrix, extended addressing,
self-host compiler, or target size/timing measurement claim.
