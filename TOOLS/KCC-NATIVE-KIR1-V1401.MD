# Native KCC KGEN KIR1 stack-lifetime acceptance v1401

Date: 2026-11-09.  Scope: native PDP-6 compiler correctness, not a
production DAIMOS kernel change or a final size-optimization pass.

## Independent defects

At v1373, full native KCC compilation of a multi-function C89 test
reported `gaddress: bad Sclass 0`, while standalone KGEN failed with
a UUO at 044572 and exit status 126.  Both optimized and `-N` KGEN
could fail, including on a one-function input.  These were independent
defects, not evidence of a wrong KIR1 symbol-class wire format.

The v1395 host compiler corrected a call-unsafe loop-carried induction
pointer.  Native KGEN's code for KIR1 graph reconstruction had
preserved a register address across a call and subsequently stored
through a stale register.  The KIR1 symbol was decoded with its proper
class 17, then corrupted by that generated code.  Disabling the
induction-address fast path for loop bodies containing calls, while
retaining it for call-free loops, corrected multi-function integrated
compilation and host/native optimized output equality.

Standalone KGEN's remaining failure came from the stack lifetime of
two large local arrays.  `kir_read_next()` reserved an automatic
`INT w[KIR_MAX_RECORD_WORDS]`, with `KIR_MAX_RECORD_WORDS = 512`.
It called `readgraph()`, which reserved another 512-word automatic
array.  The nested frame exceeded the PDP-6 native process startup
stack margin before symbol record reconstruction could finish.

In v1401, `readgraph()` receives the existing caller-owned `w`
buffer and copies the 24-word KIR1 graph header locally before
starting subsequent reads.  The serialized KIR1 format, record
limits, symbol IDs, value representation and heap ownership are
unchanged.  The fix removes the second 512-word automatic array and
introduces no heap allocation.  It is a correction of record-buffer
ownership and stack lifetime, not a workaround for the crash.

## Acceptance

The testkit is `daimos-testkit/tests/validation/
kcc-kgen-stack-lifetime-v1401`.  Its versioned runner first builds
the staged host KCC from the corrected source, then all five native
KCC/KCPP/KPARSE/KGEN/KOPT executables.  The actual normal grouped
237-file D6FS root is rebuilt with those executables; `d6fsck`
reports CLEAN.  Guest tests run in a native PDP-6 SIMH session with
isolated media.  No installed compiler or production media changes.

The 16 strict guest checks cover minimal and multi-function source:
KCPP and KPARSE processing; optimized KGEN and `KGEN -N`; KOPT
replay of both output formats; integrated KCC; and exact guest `CMP`
against independent host assembly for optimized replay and integrated
compilation.  Each compiler command must return status zero.  A
nonzero status or UUO is a failure, not a transcript-only PASS.

All 16 checks pass.  Previous `gaddress: bad Sclass 0` and KGEN
UUO/126 failures do not recur.  The test stages a fresh complete
image rather than reusing an earlier instrumented KGEN binary.

## Open release gates

Independent `-N` host/native *assembly identity* remains untested:
the v1401 `-N` probes verify successful KGEN and KOPT execution.
The native compiler's packed ASCII7 assembler output still needs a
separately accepted conversion to canonical uppercase SIXBIT/S6REC,
followed by native DAS, DLINK and actual execution of the new DXR.
The full 96K working-set/stack and self-hosting qualification also
remain open.  Do not install this staging compiler as the production
bootstrap until those gates pass.
