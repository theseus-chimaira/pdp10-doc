# KCC terminal labels (v61e)

Date: 2026-11-28.

KCC 81ae951, testkit b7e2d294. GNU/KCC mode accepts ordinary,
case and default labels immediately before a closing brace.
The parser keeps the existing labeled null-statement representation
and leaves the closing brace to the enclosing scope parser. No new
node type, symbol field, persistent table or target runtime support.
Strict C89 and C99 continue to reject the extension.

## Verification

The original v61d compiler and the candidate were rebuilt and tested
on charybdis with the same installed runtime and simulator tools, in
separate versioned directories and with no cached cells. The new
semantic regression exercises terminal goto targets, chained case
labels, default dispatch, and nested labels. The original source
20000211-3.c and the new semantic test failed all 44 cells before the
fix; all 44 pass after it. Explicit -Pc89 and -Pc99 reject the new
source; -Pgnu89 and -Pgnu99 accept it.

The complete 1042-cell active KCC compilation replay gives:

| Revision | PASS | compile_fail |
| --- | ---: | ---: |
| v61d | 694 | 348 |
| v61e | 738 | 304 |

The local v61d failure set exactly matches its committed manifest.
The candidate retains every prior pass and recovers all 22 cells
each of 20000211-3.c and 950922-1.c. No new failing cells, ICE,
link failures, run failures or timeouts in this replay. The remaining
19 sources (304 cells) are recorded in the testkit manifest
PDP10-KCC-COMPILE-REMAINING-20261128-v61e.tsv. No new scope
exclusions: of the original 2196 compilation failures, 1154 remain
documented unsupported, 738 are recovered and 304 remain active.

## Native footprint and installation

Both ccstmt.c versions were cross-compiled by the same rebuilt v61d
compiler for PDP-6, -Pgnu99 -O -m=gas, HOST_DAIMOS=1, HOST_UNIX=0,
then assembled with DAS -F -C. DOBJ sections, in words:

| Section | Before | After |
| --- | ---: | ---: |
| Text | 8876 | 8885 |
| Data | 2 | 2 |
| BSS | 23 | 23 |

Nine extra text words; no persistent RAM data growth. Native phase
execution and linked KPARSE size were not measured. The test only
addresses terminal labels, not general GNU label-as-value extensions.

Evidence relative to TMPDIR: kcc-terminal-label-v61e.GXZqfM
(baseline-results, candidate-results, strict, native, full-baseline,
full-replay, install-v1/local.tar.gz). The archive contains only
bin/kcc and was installed in PDP10_PREFIX. Installed binary was
compared byte-for-byte with the tested candidate. Previous binary
preserved in install-v1/previous-kcc-v1.

Installed bin/kcc SHA256:
24e5a3e6c8283d4242a55d79f811463813bfa73892f716b8ca166f513a8944fd

Compiler-only local.tar.gz SHA256:
06682e3fe099637e1d5a743d4e174c4115279a3b9e074616fc7b1b277b263de3
