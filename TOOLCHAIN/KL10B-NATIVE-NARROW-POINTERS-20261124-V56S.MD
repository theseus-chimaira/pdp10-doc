# KL10B native narrow pointer arithmetic (v56s)

GCC a8eb147, testkit d12b60f. The native global byte-pointer difference
algorithm now covers all five architectural OWGBP widths: 6, 7, 8,
9 and 18. For native 6/7/8-bit C arrays, the number of slots per word
is 6/5/4 respectively. Each byte occupies one C storage unit. The
word-coordinate difference fits signed 31 bits; scaling by at most
six fits the target signed 36-bit word. Use a shift for four slots,
shift plus addition for five, and twice (delta + 2*delta) for six.
The signed selector difference completes the result. The 9/18-bit
paths and all local-address paths are unchanged. No new runtime
helper or correction table is required.

## Verification and measured footprint

The dedicated fixture gcc-global-narrow-pointer-v56r.c tests signed
and unsigned arrays, dynamic loads and stores, static pointer offsets,
pre/post increments, positive and negative adjustments, all 361 ordered
pointer pairs per width (1083 per execution), one-past differences,
aligned word-to-byte and byte-to-word round trips, static versus
runtime pointer conversions, and null conversions. Both the v56q
baseline and v56s candidate pass all four optimization profiles on
SIMH and KLH10. The new regression is not a recovered historical
failure. It does not establish arbitrary mixed-width conversions,
packed records, or C arrays spanning a hardware section boundary.

Linked image words (TEXT + DATA + BSS):

| Optimization | v56q | v56s | Saved |
|---|---:|---:|---:|
| O0 | 1004 | 921 | 83 |
| O1 | 739 | 662 | 77 |
| O2 | 736 | 659 | 77 |
| Os | 720 | 643 | 77 |

In this fixture, DATA drops from 95 to 7 words because the three
correction-table members are no longer linked. TEXT increases by
5 words at O0 and 11 at each optimized level. These are image
footprints, not measured peak RAM or execution times. The tables
remain available for other compiler modes; no library removal.

Existing native record and switch tests pass all 16 extended runtime
observations. The previous global execution and archive tests pass
10 observations, and the static-pointer object checks pass four
profiles. All 64 local code-generation comparisons are byte-identical
versus v56q. Inventory audit passes. The canonical compiler matrix
and historical 2124 local failure count remain unchanged.

Evidence: testkit manifests/PDP10-GCC-GLOBAL-NARROW-EXECUTION-20261124-v56s.tsv
contains 16 baseline/candidate observations with image sizes and hashes.
Staged compiler: TMPDIR/pdp10-gcc-global-narrow-v56s/gcc; cc1 SHA256:
576c4824d89d766b31db25ab3e6b522443d759869bd43203bdf4c71c2719f6d4.
Production installation is deferred. No DAIMOS changes.

Exact-width 16/32-bit packed global pointers, independent multiply
verification, extended runtime, multi-TEXT placement and KCC remain
open.
