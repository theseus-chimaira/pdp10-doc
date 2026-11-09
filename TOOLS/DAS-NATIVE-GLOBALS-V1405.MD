# DAS native DOBJ global-symbol phase transport v1405

Date: 2026-11-09.  Scope: completed spill-backed symbol metadata
stage, not complete native DOBJ1 serialization.

## Design

Native DAS1 now preserves `.globl`, `.extern`, `.entry` and public
common symbols for the relocatable-object path.  Every distinct
global obtains a stable one-based index.  A second declaration of
the same name reuses the index.  Each global is represented by a
normal-width symbol scratch record using a separate hash namespace,
with its value encoding the index.  No resident global-name vector
or duplicate source buffer is allocated.

The private DASIR2 state extends to carry the global count and
entry-symbol index.  Existing symbol-record streaming transports
the underlying name and index markers.  DAS2 verifies every marker
in original index order and checks that the reconstructed total and
entry index match the transmitted state.  Bad or incomplete phase
metadata is rejected.

## Acceptance

`daimos-testkit/tests/validation/das-native-global-spill-v1405`
builds a complete isolated grouped D6FS root using the corrected
native assembler and v1401 compiler.  Five strict real PDP-6 SIMH
probes pass: native KCC C89 compilation, unchanged native DXR
assembly, one preserved KCC global, an assembly fixture with three
unique globals and entry 1 despite duplicate declarations, and
rejection of unfinished output without creating a false DOBJ file.

The native diagnostic reports `GddddddEdddddd` with fixed-width
octal global count and entry index using the existing console
character syscall.  It is used only on the currently unsupported
native `-C` path; no additional resident diagnostic buffer exists.

## Open work

The stage does not yet track full section-relative and external
relocation records nor write DOBJ1.  `DAS -C` still fails by design.
The next implementation must use spill-backed relocation storage,
emit correct DOBJ1 symbol and relocation metadata, and pass native
`KCC -> DAS -A -C -> DLINK -> RUN`, without a host conversion step.
Independent 96K self-hosting and full release qualification remain
open.  Final KCORE size reduction is the last KCORE work stage.
