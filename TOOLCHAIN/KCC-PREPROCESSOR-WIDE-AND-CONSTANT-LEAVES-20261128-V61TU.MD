# KCC native-width preprocessing and constant leaf traversal (v61t/u)

Date: 2026-11-28.

## Changes

- KCC 7b3b0f4 (v61u): ABI query marking and scratch-register analysis recognize terminal expression nodes instead of interpreting their payload unions as child pointers. A narrowed wide literal can retain its original high word in the union even after the wide flag is cleared. The scratch-register and ABI passes formerly followed that word as an address (including address 1 on the host). Both crashes were reproduced during the in-place evaluator experiment and traced independently. No optimization is disabled.
- KCC 9b124ae (v61t): preprocessor expressions use target precision, 36 bits for strict C89 and 71 bits in C99/GNU modes. PPVAL holds 36 high bits and 35 low bits plus unsignedness in the spare bit. Arithmetic uses unsigned word operations, carry/borrow propagation, bounded shift/add multiplication and restoring division, with a single-word division fast path. Literal accumulation avoids host-width dependence. Logical and conditional expressions retain short-circuit evaluation. The maximum signed long-long macro is provided in modes supporting that type.
- The hosted wide constant-folding left shift by exactly 35 now preserves the low bit of the high limb. This defect independently affected ordinary C constant expressions, not only preprocessing.
- pdp10-testkit 3a42a1b7 registers the independent preprocessing oracle, the all-shift-count runtime regression and the constant-leaf regression including a nested conditional that triggers ABI query analysis.

## Verification

Evidence root: $HOME/tmp/kcc-pp-wide-v61t.PhNfQm.

The registered test kcc-pp-wide-v61t passes 760 host arithmetic checks, 14 rejection checks and 127 native checks in each of eight PDP-6, KA10, KI10 and KS10 optimized/unoptimized configurations (1016 native checks total). The native fixture extracts the current evaluator from ccpp.c and uses the actual compiler token structures. These are native instruction executions, not a complete native KCPP guest run or compiler self-host.

The independent shift regression checks folded constants and variable shifts against Python-generated results for every shift count from 0 to 70. All 22 baseline configurations fail and all 22 candidate configurations pass. The strengthened leaf regression reproduces 11 optimized compiler ICEs on the baseline, with the 11 unoptimized cells passing.

Both revisions of each measured compilation unit were compiled with the final candidate and measured in native 36-bit words:

| Unit | Before text | After text | Delta | Data/BSS before and after |
|---|---:|---:|---:|---|
| ccpp | 12740 | 13610 | +870 | 55 / 777 |
| cceval | 7390 | 7390 | 0 | 0 / 2 |
| ccgen | 5471 | 5471 | 0 | 0 / 27 |
| ccdecl | 14304 | 14304 | 0 | 1 / 13 |

The earlier value-return preprocessor measured 13951 text words. The final caller-owned in-place representation measures 13610, 341 fewer words (the experiments used slightly different compiler revisions). No persistent storage is added. This is a compilation-unit measurement; peak stack, whole compiler memory and execution time have not been measured. A native scalar alternative was larger than the corresponding portable implementation, so was not retained.

The exact 1042-cell historical replay reports 892 pass, 84 compile_fail, 66 run_fail; all 150 failure identities and classes unchanged. The two imported shift tests ashldi-1.c and ashrdi-1.c now select their intended 71-bit branch, but still contain integer literals exceeding that width. Their four failures are retained; overflow diagnostics are not weakened. Likewise the historical conflicting function declarations remain unresolved rather than accepted by removing type checks. Full-word relocation, GCC work and DAIMOS changes are outside this batch. The historical v61o manifest remains authoritative.

The final 4388-cell expanded replay reports 3980 pass, 224 compile_only, 4 compile_fail, 44 run_fail, 2 link_fail and 134 policy_fail. The matched baseline consists of the previous 3444-cell replay, 922 extra baseline cells and 22 strengthened leaf baseline cells. Its 217 failure identities shrink to 184, with precisely 22 shift runtime failures and 11 leaf compiler ICEs removed and no added failures. The remaining 184 failures are unchanged, including the expanded policy cohort. All 44 cells of the two new regression sources pass. This is not a full matrix sweep.

## Installation

The compiler-only install-v61tu/local.tar.gz has SHA-256 b0648002ec34fbca995c13a8a1126b087433521711e19f51a51fc358c46e545f. Installed into PDP10_PREFIX, the kcc executable matches the tested candidate byte-for-byte (SHA-256 91f19bf1192fb2392d09a6ebf1025526aafc54f7f7036eea4ff7c181d7c73e42). The previous v61r compiler is preserved at install-v61tu/previous-kcc-v61r. No native compiler images, linker, assembler or GCC binaries were replaced.
