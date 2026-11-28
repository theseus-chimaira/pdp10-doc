# DOBJ1 relocation boundary, KCC indexed address and fold safety (v61q/r/s)

Date: 2026-11-28.

## Changes

- daimos-tools db3f791 (v61q): the host DOBJ1 reader rejects relocation kinds outside 1..4; the writer checks every kind before emitting the object header. The native object parser performs a sequential validation pass over relocation triples, reusing its existing header buffer. The direct native relocation decoder also rejects undefined kinds. This is relocation-kind validation, not a complete structural validator.
- kcc 4a542ff (v61r): in cccode.c, foldxref's indexed ADD path must retain the result accumulator when foldidx changes the input index. Remove the assignment of q->Pindex to s. The subsequent code40 call uses the computed address, not the input index. This retains the optimization and removes a redundant address computation in the reproducer.
- daimos-tools 87c1529 (v61s): jump_foldable_block rejects a donor immediately preceded by an instruction recognized by may_skip_next. An implicit skip entry must not bypass the replacement JRST and enter the removed tail.
- pdp10-testkit 574fca13, 910e6c35: independent compiler regression, host relocation consumer/writer and native reader tests, registration and temporary-file handling.

## Verification

Evidence root: $HOME/tmp/dobj-relocation-boundary-v61q.3aBo3Q.

The host consumer test mutates relocation kinds 0, 5, 6 and 7 at TEXT and DATA sites. Both the installed baseline and original source-tree baseline accepted 28 invalid consumer/record combinations. All 40 invalid direct combinations and eight selected invalid archive-member cases are rejected after the change. Five positive consumer checks pass (53 total). Direct writer tests accept kinds 1..4 and reject six invalid values before emitting bytes. Valid archive and fixed-base link output is byte-identical in the tested old/new cases.

The strengthened native reader fixture contains valid symbols and relocation records at a nonzero archive-member offset. It tests all eight kind values, both sections and record positions, full and partial reads, and truncation/restoration. All eight CPU/optimization configurations pass with the corrected compiler. The original native reader separately fails the invalid-kind control when built with that compiler. These are native instruction executions with test-only file I/O, not complete DAIMOS guest runs.

The stronger fixture exposed the KCC bug: an incorrect SETZB indexed by AC2 (holding octal 17) cleared AC17, causing timeout. The independent kcc-foldxref-index-result-v61r regression uses index 0100, canaries, zero/all-one stores and a call boundary. Baseline: all 11 optimized cells fail and all 11 unoptimized cells pass. Candidate: 22/22 pass. No optimization was disabled.

The source-tree p10fold baseline failed the existing xskip-donor guard although the installed baseline passed. The v61s fix restores the complete established fold/link contract. Existing DOBJ1/DARC1, objdump, super-CFG, super-rewrite and DAS/DOBJ/DARC/DLINK integration tests pass. Native darc, dlink and objdump programs cross-compile and link; full guest execution remains outstanding.

Native text sizes (36-bit words): cccode 7307 -> 7304 (-3), DOBJ reader 1142 -> 1193 (+51). Data and BSS unchanged. Reader validation requires no additional buffer allocation; its extra sequential I/O cost has not been measured. No native compiler self-host, peak RAM or execution timing measurement was performed.

Historical exact replay: 1042 cells, 892 pass, 84 compile_fail, 66 run_fail; all 150 failure keys unchanged from v61p. Broad replay: 3444 cells, 3205 pass, 180 compile_only, 44 run_fail, 2 link_fail, 13 policy_fail. The 22 new cells pass and all 59 prior failure keys are unchanged. Strict exit remains nonzero because known failures are retained. This is not a full matrix run.

## Installation and boundaries

Six matched host binaries (kcc, darc, dlink, pdp10-objdump, p10fold, p10super) installed into PDP10_PREFIX from install-v61qrs/local.tar.gz, SHA-256 6aee3e0cba476cec7070a573bacf018eb6db71d2ef01679ac377f29f26a63f14. Previous installed binaries retained in install-v61qrs/previous; all installed files compared byte-for-byte with the tested candidate. The original installed dlink, objdump and p10fold differed from the repository builds; the tested positive output comparisons include the installed baseline. Native utility images were not installed. DAS and GCC unchanged.

No new relocation format has been introduced. DOBJ1 has a three-bit kind field with defined RH/LH kinds 1..4; the DAS host DOBJ2 path uses distinct magic and kinds 5/6 for 30-bit addresses. The daimos-tools checkout implements DOBJ1, not the independent DOBJ2 host-tools implementation. DAIMOS DXR has a one-bit RH relocation map; MRES uses two-bit none/RH/LH/both-independent-half relocation semantics. A full-word carry/borrow relocation must be specified coherently across assembler, linker and loader, not silently mapped to RH18 or both-half relocation. See KCC-EXTERNAL-VOID-20261128-V61P.MD. The 44 symbolic integer initializer cells remain open. No encoding has been selected. DAIMOS has existing local modifications and was not changed; its upstream PROJECT-STATE.MD was inspected. The historical remaining-failure manifest from v61o remains authoritative.
