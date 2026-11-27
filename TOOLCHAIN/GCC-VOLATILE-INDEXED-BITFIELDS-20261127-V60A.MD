# GCC volatile indexed bitfields (v60a)

## Repair

GCC `cba1fade` extends the v59z physical word-stride bitfield
recognizer to volatile bitfields in the same direct, native,
word-stride record layouts. Carry TREE_THIS_VOLATILE across the
recognized reference path and into the fresh physical word MEM.
The existing slice helper preserves MEM_VOLATILE_P when constructing
the actual word access. Do not mutate the root declaration RTL.
Non-bitfield volatile references and packed/nonword-stride records
retain their previous lowering.

For a value-producing volatile assignment, return the truncated
or sign-extended source value instead of reading the destination
again. This avoids an additional device-memory read. Subword writes
continue to use the native DPB read/modify/write instruction.
Volatile reads, including discarded expressions, remain observable.
The existing compiler mechanisms implement this repair; no target
runtime helper, new ABI, additional static storage or library archive
change is needed.

## Acceptance

Testkit `520684cb` adds gcc-volatile-indexed-bitfields-v60a.c
(44 GCC profiles) and test-volatile-bitfield-access-v60a.py.
The unchanged v59z volatile scope probe now passes PDP6 O2.
The new semantic fixture fails assertion 1 with the saved previous
compiler and passes all 44 installed profiles. It covers independent
physical-word readback, signed and unsigned assignment truncation,
single-evaluation subscripts, compound writes, neighboring fields
and records, volatile individual members, and 16-bit middle fields.

The permanent assembly audit checks six functions at O2 on PDP6,
KA10, KI10, KS10: simple assignments zero explicit reads/one DPB;
two separate reads remain two; discarded expression one read;
compound update one explicit read/one DPB; 16-bit assignment and
readback one read/one DPB. DPB itself performs the containing-word
read/modify/write. Both deliberately altered assembly controls
(eliminated repeated read and additional assignment readback)
are rejected. Independent PDP6 initial RTL inspection finds the
corresponding volatile word MEM counts 1,1,2,1,2,2.
For the simple unsigned assignment the optimized PDP6 code is
five instructions (LSH, DPB, ANDI, MOVE, POPJ), no stack frame.
This is a specific code-generation measurement, not a global timing
or memory benchmark.

All 680 v59z matrix guards have identical classifications:
648 PASS, 32 existing run_fail. Combined with 44 new passes:
692 PASS, 32 existing run_fail in 724 distinct selected cells.
BLT/XBLT host RTL audit 4/4 remains green. No new regression.
The compiler builds successfully with legacy GCC host warnings.
The original 404 link-failure cohort remains 259 genuine runtime
passes, 48 corrected registrations and 97 unresolved.

## Scope and next work

Native nonextended direct volatile bitfields in word-stride records
are repaired within the original v59z width and memory-backed root
restrictions. No claim for general volatile packed or indirect
references, wide fields, native self-host or extended-section runtime.
Signed 52/53-bit field operations and packed-pointer arrays retain
their 16 existing failures each. The wide-field arithmetic/storage
boundary in expmed.c is the next compiler investigation.
No full-matrix sweep, DAIMOS kernel changes or runtime archive changes.

Future new target runtime functions are to be hand-written PDP-10
assembly, with RAM consumption first and execution speed second,
rather than adding C runtime implementations. No new target
function was required by v60a.

## Evidence

`$HOME/tmp/gcc-volatile-fields-v60a.jhxFzHIp`:
probe-v1, volatile-v1, baseline, guards-v1, guard-comparison-v60a.tsv,
access-counts.log, rtl-audit, build-v1.log.
Installed compiler-only install-v1/local.tar.gz SHA-256:
`1c9e3ab7ebabfbfe2f64ad82e691999cfe2664224a3dc1b5bef6474e3ca7480c`.
