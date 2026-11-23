# PDP-10 GCC Installed Sweep Repair (2026-11-23, v16)

## Completed scope

The interrupted `pdp10-testkit` compiler sweep was recovered without
resetting previously completed cells. The installed GCC compiler was
then brought up to the already-fixed source state, eliminating the two
observed O0 ICEs and the 32 compilation failures in the 9-bit shift
oracles. The original guest tests were retained unchanged.

## Cause and correction

The build-time GCC source already contained the `instantiate_virtual_regs_1`
repair and the historical `__pdp10__` target predefine. The older
installed compiler under `$PDP10_PREFIX` did not include those fixes.
Its `cc1` searched a previous private build prefix for standard
headers. That older `limits.h` selected 8-bit host-character limits
because the old driver did not define lowercase `__pdp10__`.

A fresh out-of-tree `gcc-pdp10/build.sh --no-regen` build set both
`PDP10_PREFIX` and `PREFIX` to `$HOME/cross`, and installed successfully.
The installed `limits.h` was not edited; its existing PDP-10 branch
already defines `CHAR_BIT` as 9. The rebuilt compiler now searches
`$HOME/cross/lib/gcc-lib/pdp10-dec-none/3.2/include/limits.h`.

Compiler identity after installation:

- `cc1` SHA256: `b4f6fe1e12f87030ba52c70689a8a5b0485204e45fd11d86aa47ab2c596bcf1f`
- `limits.h` SHA256: `6627c97182848c8909483507607ecda6f3f8f509b86406c78c6ede0e786db233`
- `CHAR_BIT == 9`, `__PDP10__ == 1`, `__pdp10__ == 1`

## Verified target execution

The official restartable compiler matrix runner produced:

| Exact source / operation | Outcome |
| --- | --- |
| Interrupted struct-return test, 4 `running` and 10 missing cells | 14 new PASS; 54/54 total |
| GNU DEC signbit initializer, GCC O0 PDP-6 and KA10 | 2/2 PASS; 16/16 source total |
| 9/18/36-bit shift reference, all 16 GCC profiles | 16/16 PASS |
| Signed right-shift reference, all 16 GCC profiles | 16/16 PASS |

The test runner's restart/result-accounting self-check also passed.
The resulting **partial** checkpoint census is 6,978 cells:
2,829 PASS, 3,090 compile-only, 1,059 runtime failures, with zero
ICE, compilation failure or incomplete checkpoints.

This is not a full 123,576-cell result. The 1,059 runtime failures
remain recorded and have not been reclassified or discarded. No
compiler-source or test-oracle change was necessary in this repair.

Detailed checkpoint evidence is maintained in
`pdp10-testkit/history/PDP10-FULL-SWEEP-CHECKPOINT-RECOVERY-20261123-v16.md`.
