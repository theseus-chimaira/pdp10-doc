# GCC block-transfer memory dependencies (v59y)

## Defect and repair

GCC `931c7543` corrects the RTL description of BLT and XBLT. In the
native computed-goto companion, the aggregate copy was an
UNSPEC_VOLATILE with a control-register clobber and a USE of the final
destination address, but no memory write. In the pre-repair
`10.addressof` dump, instruction-field extracts follow the BLT inside
the dispatch loop. The `11.gcse` dump is the first incorrect pass:
it moves those extracts and a word load above the dispatch label.
The final assembly then repeatedly dispatches using stale instruction
data, exhausting the simulator step limit in nine GCC O2 cells.

Add the standard `(clobber (mem:BLK (scratch)))` RTL effect to both
block transfer patterns. GCC's alias analysis explicitly defines this
form as conflicting with every memory reference. The existing
UNSPEC_VOLATILE protects opaque reads; the memory clobber exposes the
hidden writes to GCSE, store motion and other RTL optimizers.
The BLT end address alone does not identify the written range. This
conservative representation avoids inventing precise memory extents
or requiring source and destination addresses to remain live in
additional accumulators. It can inhibit unrelated memory optimizations
across a block transfer; it does not disable any optimization pass
or add target instructions, runtime storage or registers. Both native
transfer instructions remain in use.

All matching/replacement BLT peepholes preserve the clobber; both
late repack recognizers accept the four-element PARALLEL, keeping
the original end operand at element 2. Removed an obsolete MOVEI
peephole that omitted the control AC clobber and could not match
the former actual three-element BLT. The repaired GCSE dump keeps
the field loads after the copy; final assembly does likewise.

## Acceptance

Testkit `4497ab1e`: native typed-page computed-goto test passes
GCC 44/44 and KCC 22/22. New independent
`gcc-blt-memory-effects-v59y.c` uses raw initialized words, ordinary
nonvolatile fields, repeated aggregate copies and memset fills.
The previous GCC fails its first assertion on PDP6 O2; the repaired
compiler passes all 44 profiles. Real BLT remains in the emitted code.
Four compile-time RTL/assembly controls verify PDP6 BLT and extended
KL10 XBLT at O0/O2; XBLT runtime execution is not established here.

The 864-cell GCC guard run has 689 runtime passes, 72 compile-only
cells and 103 unchanged run failures. Comparison against the original
sweep: 601 prior passes preserved, 72 prior compile-only preserved,
103 prior run_fail unchanged, 88 newer cells pass (44 independent
BLT oracles and 44 v59v struct-return oracles). No new failure.
The 66 native dispatch replays are separate from these 864 cells.

The initial independent probe using indexed bitfield stores failed
both before and after the BLT repair: generated code incorrectly
steps a word address with IBP. Its source, RTL and assembly are
retained separately. Raw-word initialization isolates the block-copy
dependency rather than claiming that indexed bitfield stores are fixed.
Existing signed/array bitfield defects remain open.

## Historical test admission and outstanding scope

The v59x linked-image audit established that all 24 original
`tests/todo/generic/comp-goto-1.c` cells access outside their
allocated payload due to flat byte-address integer assumptions.
The historical source SHA remains
`c874cfc465c32b0361a952505b4a11fcb605441e3ee8ae5da785324ef32c8bae`.
Its 20 active native matrix columns (24 actual cells) are now
unsupported. The complete native companion remains registered and
verified. A source-pinned audit checks both admissions and four
negative controls. This is corrected test scope, not 24 original
runtime passes. The six old P1 admission auditors already reject
the preceding live matrix at a PDP6 GCC optimization admission;
both before/after checks reproduce that existing problem. Their
frozen ledgers and auditors remain unchanged; reconciling historical
and current admission policy is outstanding.

Of 404 original link failures: 259 confirmed runtime passes,
48 corrected registrations, 97 unresolved (94 console-dependent,
three capacity-limited allocations). New regression passes are not
added to those original-cohort counts. The original full sweep was
not overwritten or rerun.

## Evidence and installation

Evidence: `$HOME/tmp/gcc-dispatch-v59y.S5pulYQd`.
Original and repaired RTL dumps in the root and `rtl-after`;
`oracle-before-v2`, `oracle-after-v2`, `indexed-field-store-probe-v1.c`,
`oracle-before`, `oracle-after`, `dispatch-replay-v1`,
`kcc-dispatch-v1`, `guards-v1`, `guard-comparison-v59y.tsv`,
`blt-rtl-v59y.log` and `xblt-compile-v1`.
Installed compiler-only `install-v1/local.tar.gz` SHA-256:
`e9e217aa7222108eeffdb3e07b11d92f712f3948091f7479824b02b0277a43a7`.
No runtime archive, DAIMOS kernel code or historical source changed.
Native self-host and extended-section runtime are not newly verified.
