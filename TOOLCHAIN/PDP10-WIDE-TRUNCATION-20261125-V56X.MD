# Normalized wide integer truncation and direct global packed fields (v56x)

GCC 47178f0, testkit a9c2870. The repair applies to both local and
extended C compilation. The canonical compiler matrix is unchanged.

## Defect and correction

Normalized 71-bit integers have a 36-bit high word and 35 low payload
bits; the second physical word's high bit is not logical bit 35.
Truncation to a 36-bit integer must recover that bit from the high
word's least significant bit. Previously convert_move's generic
multiword shortcut and convert_modes' memory lowpart shortcut
returned the physical low word, silently losing this information.

Declare 71-to-36 truncation nontrivial in TRULY_NOOP_TRUNCATION and
select the target truncdisi2 expander before the generic multiword
shortcut. Respect this property for nonvolatile memory operands too.
The expander copies the low word and reuses the existing multiply-low
canonicalizer to install the correct high bit. No helper or table is
introduced. The raw72 model retains its ordinary lowpart behavior.
Narrower 9/18-bit truncations do not need this extra payload bit.

## Verification

The existing pdp10-wide-multiply-36bit-v1 test passed product equality
but failed the logical low-word comparison in all four extended
baseline v56s SIMH runs. Its first failing comparison reported
fail_id 013 octal. The repaired compiler passes all four optimization
profiles on SIMH and KLH10. The new truncation fixture covers register,
volatile and nonvolatile memory, arguments and returns, unsigned
sign-bit boundaries, negative wide inputs and multiplication residues.

Dedicated testkit test `gcc-global-wide-v56u` covers six extended
cases (normalized71, raw72, wide product, unsigned multiply, sign-bit
multiply, direct packed fields) at O0/O1/O2/Os on two emulators:
48 successful runtime observations. Both 36-bit multiplication
regressions already passed on the baseline. The new truncation fixture
also passes 32 dedicated local runs (four CPUs, two wide modes, four
optimization levels). No historical matrix recoveries are counted.
The previous 34 extended narrow, record, pointer and archive runtime
observations and four static pointer object profiles pass again.

Of 64 local assembly comparisons against v56s, 61 are identical;
three optimized byte-pointer cases differ in stack spill-slot
assignment and pass execution. Sixteen raw72 truncation assemblies
are byte-identical. No timing or peak memory measurements are claimed.
Inventory audit passes. Evidence manifest:
`pdp10-testkit/manifests/PDP10-GCC-WIDE-TRUNCATION-20261125-v56x.tsv`
contains 80 candidate runtime observations plus four baseline wide
failures, with provenance hashes. Candidate cc1 SHA256:
006f23c4ca9c803d60bca16d94b41b43e4b5cb9cf0c0f72c9fa869a2c2725024.
Staged compiler wrapper: TMPDIR/pdp10-gcc-wide-truncate-v56x/gcc.

## Exact-width extended pointer limitations

The dedicated constant-index direct packed field fixture passes on
both KL10B emulators, including signed/unsigned 16/32-bit fields,
fields crossing word boundaries and preservation of neighboring
bitfields. This uses the existing two-word hardware byte-pointer
literal machinery for noncanonical fields. It does not validate
escaping C pointers, arbitrary packed indices or packed aggregate ABI.
The architectural one-word global byte pointer selectors cover only
6, 7, 8, 9 and 18 bits. A C pointer to an exact-width packed bit
position must represent a full global word coordinate and the bit
position independently of the local P/S representation. The exact16
pointer call fixture still produces local/global hybrid literals in
the baseline; the packed exact32 static initializer crashes cc1.
The common GCC/KCC software pointer encoding, null/cast semantics and
native versus continuous stride are not yet accepted. No speculative
ABI change has been installed. KCC extended mode, runtime libraries
and multi-TEXT placement remain open.

No DAIMOS changes or production installation.
