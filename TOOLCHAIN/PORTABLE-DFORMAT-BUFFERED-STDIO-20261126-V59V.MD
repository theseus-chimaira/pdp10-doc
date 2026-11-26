# Portable D-format buffered stdio v3 (v59v)

KCC 325a20b; GCC 6c259f63; testkit c2cda283. Completes the
floating-format part of the v59u sprintf cohort. Both compiler
trees have identical p10stdio-format-v2.c, p10stdio-format-v3.c
and p10stdio-float-v3.h source copies, compiled separately with
their own private va_list ABI. The v3 wrapper enables the new
code conditionally; independently rebuilt v2 archives compare
byte-identical to their previously installed counterparts.

The optional libp10stdio-v3.a and p10stdio-format-v3.s are
installed in each compiler's freestanding directory by the
normal build/install rules. The semantic runner now selects v3
for the four buffer interfaces. Explicit integer-only links can
continue selecting v2. The core C archive and DAIMOS console
code are unchanged; v2 and v3 are alternative providers, not
archives to be linked together. Neither requires heap, mutable
static storage or operating-system I/O. The callback sink in
v3 is available to future console work; no console implementation
is claimed here.

The finite portable D-format magnitude is M*2^(E-128-62), with
62 fraction bits. Decode raw words and normalize the negative
two's-complement representation using unsigned 36-bit operations.
Build an exact decimal coefficient in seventeen base-10^9 limbs,
scaling with powers of 2 (up to 64 per multiplication) or 5
(up to 25), with a decimal point offset for negative exponents.
The maximum 190 factors of five require fewer than 152 decimal
digits, so the 153-character buffer covers the entire finite
format. All limb products fit an unsigned 36-bit word. No native
FP arithmetic or wide integer division is needed. Round once
in decimal, nearest with ties to even, retaining the original
negative sign even when rounding yields zero. Carry propagation
can change the decimal exponent; general notation is selected
after rounding. Fixed, general and scientific formats f/F,
g/G, e/E support default and explicit precision, width, the
existing flags, and bounded-buffer counts. Trailing zero removal
and alternate form follow general-format rules. V3's bounded
buffer repeat counts discarded padding without visiting each
discarded character. Callback sinks continue emitting each byte.
Oversized return counts are rejected. There is no IEEE special
value support, hexadecimal FP format or long double formatter.

Workspace arrays are 17 target words and 153 nine-bit characters
(39 words), or 56 words together, excluding scalar locals, ABI
frames and call stack. No claim of measured maximum total stack
depth or runtime speed. Host archive sizes: KCC v2 17912 bytes,
v3 33040; GCC v2 11008 bytes, v3 22072. These are serialized
object archives, not linked text sizes. Keep v2 selectable for
memory-constrained integer-only consumers. A program selecting
v3 pulls the complete floating-capable formatter, even if its
runtime format strings happen not to contain floating directives.

Final acceptance against the installed v3 archives/assembly:
54/54 original sprintf cells PASS, including the twelve %.0f
indirect call cases and eight default %f many-argument cases.
The two historical struct-ret-1.c cells were previously weak
passes; the new companion includes the unchanged original
source, checks exact full output, genuine vsprintf return count
and returned structure for both direct and indirect calls.
All 66 companion cells PASS. Independent Decimal-generated raw
D-format oracle (42 cases plus four buffer/ABI guards) passes
66/66 GCC/KCC profiles, including exponent endpoints, rounding
ties and sticky bits, 190-digit precision, signed zero output,
notation selection, flags, bounded output and million-character
zero-capacity count. The generator verifies the frozen generated
source. V3 integer boundary guard passes 22/22 KCC profiles.
Total 208/208 selected guest cells PASS. Both real v3 DARC
extraction tests PASS, with negative links without the archive
failing as expected. Selector's 34 controls and installed core
runtime audit PASS. Original sweep sources/limits unchanged.
Existing KCC const-discarding string prototypes cause two
warnings in the independent archive client; not a formatter
failure. No native self-host, exhaustive all-bit-pattern test,
mixed private va_list, extended-mode or DAIMOS console claim.

The original 404 link-failure cohort now has 184 confirmed
runtime passes, 24 corrected invalid registrations, and 196
unresolved/unverified cells. All 54 original sprintf cells are
now accepted, including the two previously weak passes with
independent stronger coverage. No fresh full-matrix sweep.

Evidence: `$HOME/tmp/pdp10-float-stdio-v59v.rYuylGHS`,
replay-v1/v2, oracle-v2, struct-v2, boundary-v2, archive-v2.log,
kcc-build-v1/v2.log, gcc-build-v1.log, and v2 parity copies.
Installed incremental install-v2/local.tar.gz SHA256:
`4e0cf8b9a2ed1d30b4ce7e7ffdb3715b55f521187592bac3a81eb9d916c4ea34`.
Installed KCC v3 archive SHA256:
`3e161d1b5222dfe350bc9544ea5df32524fb616c615681c576a8f839d8f89363`.
Installed GCC v3 archive SHA256:
`05e9b0b94d594f0e14039c8bbffb38c467546524ee2e5c04c36118173aa30631`.
