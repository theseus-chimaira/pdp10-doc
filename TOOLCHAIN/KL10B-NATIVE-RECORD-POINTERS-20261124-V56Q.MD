# KL10B native record references and pointer subtraction (v56q)

GCC commit 6d14993, based on aea4633; dedicated testkit regression
commit 1f425bf. No DAIMOS or existing compiler-matrix changes.

## Native record lowering

The extended-address path walks native integral byte/halfword
COMPONENT_REF and ARRAY_REF chains, accumulating constant physical
bit positions and dynamic element counts from declared layout.
It rejects unsupported packed/bit-field layouts before expanding
indices. It handles declared static and automatic objects and
indirect record roots. Static pointers retain the symbolic global
selector and address; automatic objects use XMOVEI and TLO. Indirect
roots preserve their byte selector through pointer conversion. ADJBP
applies the combined native element displacement. The resulting MEM
retains native subword alignment and volatility. This replaces the
invalid generic BLKmode reference to a non-word-aligned member array
which previously aborted expr.c during address expansion.

## Native global differences

For two valid native OWGBPs of the same width, subtract the low
30-bit word addresses, scale by the slots per word, then add the
difference of the high-six-bit selectors. For width 9, multiply
word difference by four; for width 18, multiply by two and convert
the result to two C storage units per element. The existing front
end division converts storage-unit differences to element counts.
No correction table or double-word multiply is needed for these
widths. The local subtraction path is unchanged.

The extended-only LDBzqi_sequence pattern has been removed: aligned
byte-pointer registers do not become ordinary word addresses.
LDBzqi provides the correct byte access. A v56p optimized switch
run read the wrong op field through MOVE/LSH; v56q passes.

## Verification

The native record fixture checks static and automatic records,
indirect roots, dynamic indices, exactly-once index calls, native
byte and halfword loads/stores, and 81 byte and 25 halfword signed
pointer-difference combinations. The existing base_switch_byteptr_live
fixture also runs in the dedicated extended environment. Both pass
O0/O1/O2/Os on SIMH and KLH10 (16 independent runtime observations).
The previous global pointer execution fixture and archive regression
pass on both emulators; static-pointer object tests pass all four
profiles. Four local sources across four CPUs and four optimization
levels produce byte-identical assembly versus v56n (64 comparisons).
These are not a full local runtime sweep. Inventory audit passes.
The larger test uses a bounded 100000-step SIMH execution limit.

Evidence: PDP10-GCC-GLOBAL-RECORD-EXECUTION-20261124-v56q.tsv in
pdp10-testkit/manifests. Staged cc1 SHA256:
a211be1742a35ee21ba84b277161ebbff336bfc779cb7d6123f3547499065fde.
Staged compiler wrapper: TMPDIR/pdp10-gcc-global-record-v56q/gcc.
Production tools are unchanged.

## Remaining scope

6/7/8-bit and exact-width packed global pointers, additional record
layouts, multiple TEXT placement ranges, extended libgcc/runtime,
KCC klx and mixed ABI testing remain open. Integer multiply still
requires independent investigation; this change does not alter its
instruction patterns. No historical local matrix recoveries are
counted. No execution-speed or native RAM claims are made.
