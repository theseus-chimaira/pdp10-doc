# GCC packed pointer storage and word strides (v60c)

GCC `e13df473` extends the existing physical packed-scalar reference
recognizer in expr.c to pointer-valued fields. Pointers remain full
36-bit values: their encoding, null value, and call ABI do not change.
Previously an unaligned pointer load could reconstruct both halves
while the store overwrote the whole containing word.

The recognizer retains the constant physical field position and
accumulates a separate word displacement whenever all variable
array strides are word multiples. A direct, memory-backed,
word-addressed declaration can use this coordinate with the existing
native record MEM and constant scalar slices. Continuous byte-addressed
records retain their existing byte-coordinate path. Both integral
and pointer-valued packed scalars use this rule; the wider scope is
necessary to preserve neighboring indexed integer fields in the
same aggregate. The halfword-aligned crossing case uses the
existing native halfword loads/stores. No runtime routine,
new ABI, fixed storage or runtime archive change.

The initial pointer-only change passes the original constant-index
probe but fails the variable-index probe. The separate word
coordinate repairs dynamic pointer indexing; the physical-word
oracle then exposes the same issue for neighboring integer fields.
The final v3 candidate covers both. These intermediate failures
are preserved in the evidence directory.

## Acceptance

Testkit `698ae8a8` adds gcc-packed-pointer-words-v60c.c and
gcc-packed-pointer-continuous-v60c.c, each registered across 44
GCC profiles. The first independently compares the physical
halfwords of local and global aggregates, nested array strides,
pointer assignment results, null values, neighboring fields,
single-evaluation subscripts and function pointer calls. The
second tests indirect and indexed continuous records with a
nonzero byte position. Both fail with the saved v60b compiler
on PDP6 O2 and pass with the final candidate.

Original pdp10-packed-pointer-array-v1.c 16/16 PASS, new word
oracle 44/44, new continuous oracle 44/44: 104 targeted passes.

Separate 556-cell guard replay: 512 PASS, 44 existing run_fail.
Comparison with the applicable prior checkpoints and exact
v60b replay of 15 apparent recoveries: 508 prior passes retained,
four genuine packed-pointer-global-1.c recoveries and 44 unchanged
strct-pack-1.c failures (packed double fields). Eleven strct-pack-3.c
cells already pass with v60b and are not new recoveries.
Combined 660 distinct selected matrix cells: 616 PASS,
44 unchanged run_fail. No new regression.

Original 404 link-failure cohort remains 259 verified runtime
passes, 48 corrected registrations and 97 unresolved. This repair
addresses runtime failures, not that linkage cohort.

## Scope

No general claim for volatile packed fields, arbitrary indirect
word-addressed packed records, packed multiword floating fields,
extended-section runtime or native self-hosting. No full-matrix
rerun. The 44 existing packed-double failures are a candidate
for the next compiler investigation. New target runtime routines,
when required, are to use hand-optimized PDP10 assembly with
RAM consumption and execution speed prioritized. No new target
routine was required by this repair.

## Evidence

`$HOME/tmp/gcc-packed-pointer-v60c.oVkEQwwA`:
probe-v1, dynamic-v1/v2, words-v2/v3, words-before,
continuous-baseline, continuous-candidate-v3, targets-v3,
continuous-v3, guards-v3, recovery-baseline-v1,
recovery-baseline-v1.tsv, guard-comparison-v60c.tsv and build-v3.log.
Installed compiler-only install-v3/local.tar.gz SHA256:
`6dc870fbb96a9707a1a9f20e35fff075d935c69f7699a9dde2069fd7339b8c8f`.
No DAIMOS kernel changes.
