# Native DAS relocatable DOBJ1 output v1405

Date: 2026-11-09.  Scope: resident assembler object-generation
correctness, not KCORE size optimization or production installation.

## Native object path

The `DAS -A -C -O MAIN.DOBJ MAIN.A7` command accepts KCC packed
ASCII7 source and emits standard DOBJ1 relocatable object records.
The native DAS1 and DAS2 phases retain the existing streaming
DASIR2 architecture.  DAS1 exports its declared globals through
an object-only DASIR2 section and retains names in a scratch
wordfile.  DAS2 reconstructs those names, records local and
external RH18/LH18 relocations in another scratch wordfile, then
streams the code, data, definitions and relocation records to
DOBJ1.  The header is completed after the body and metadata have
been emitted.  No large resident symbol or relocation tables and
no intermediate host object conversion are used.

The ordinary S6REC/default and ASCII7 DXR output behavior is
unchanged.  Object-only phase state does not alter the normal
DASIR2 DXR wire format.  The DOBJ1 result remains compatible with
the native DLINK streaming reader and matches host DAS output
word for word on the tested fixtures.

## PDP-6 acceptance evidence

The versioned regression is `daimos-testkit/tests/validation/
das-native-dobj-v1405`.  The full grouped 237-file D6FS image
is staged separately, verified CLEAN, and run under PDP-6 SIMH
on cloned media.  The default 29-probe suite compiles five C89
modules natively: external-function caller/provider,
external-data caller/provider and a local-data reference.

All **29/29** probes passed on the original v1405 build.
Every packed ASCII7 and native DOBJ1 file matched the independent
host output.  Three complete native DLINK images matched the
host-linked goldens, and all three native C89 programs executed
with return status 7.  An independent single-program run passed
6/6.  A local-data-only clean-boot run passed 7/7.  The final
source rebuilt both host DAS executables without C89 warnings,
and the entire previous v1402 ASCII7/S6REC/quoted-literal/DXR
compatibility suite passed **17/17** on that source.

## Outstanding reliability and coverage

One full 29-probe retry using the final DAS binaries passed
16 probes but failed to finish the 17th: guest KCC stalled while
compiling the local-data C89 fixture.  The final native phases
differ from the originally accepted images only in source-line
diagnostic numeric constants.  The repeated sequence is *not*
accepted as reliable yet.  The stalled transcript was preserved;
diagnose cumulative guest memory/swap, D6FS scratch and process
lifetime rather than hiding the failure by rebooting between
each compile.  No production toolchain was installed.

Less common relocatable `.set` aliases and mixed LH/RH relocation
expression forms require targeted semantic tests.  The independent
unoptimized `-N` host/native comparison, 96K working-set limit,
robustness under media faults and complete native self-hosting
are separate open release gates.  The final KCORE resident-size
optimization remains deferred until all essential KCORE work
and integration acceptance are complete.
