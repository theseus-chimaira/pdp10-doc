# DSH bounded SIXBIT indexing with current KCC v1746

Date: 2026-11-21. DAIMOS commit `cbcd7a7e`, testkit `825c5ac`.

The installed KCC after pointer/ABI correctness repairs (current source
`551f406`) produced an increased DSH image under the unchanged two-word
`p10fold` floor. The KCC v1735 executable produced `035751` octal;
the current executable produced `036017` octal, exceeding the strict
`036000` EXEC limit by `017` octal, or 15 decimal words. The change was
isolated to `dsh_s6.s`: C unsigned division and modulo by six acquired
longer target-correct sequences, while other DSH translation units were
unchanged. The compiler correctness fixes were retained.

`userland/dsh/dsh_s6.c` now obtains SIXBIT word and character shift
using bounded repeated subtraction of `DSH_S6_CHARS_PER_WORD` (6),
rather than general unsigned divide and modulo. Both append and get
operate on validated positions in the fixed 36-character buffer. Get
also refuses a corrupt length that would index beyond the six-word
array. The loop executes at most five iterations for valid positions;
this avoids general signed/unsigned division helper code and favors
RAM/image constraints over the small per-character extra computation.

The resulting production KCC DSH image is **`035775` octal**, three
decimal words below the strict `036000` cap, with unchanged BSS
`001763`. The reduction from the current-KCC baseline is `022` octal
(18 decimal words). No shell feature, buffer, or reservation was removed.
The separate native PDP-6 test `daimos-dsh-sixbit-36-v1746.sh`
verifies all 36 character positions, append at capacity, counted-word
packing/unpacking, copying, text comparison, and invalid index
handling; an incorrect subtract-by-five mutant is rejected. The
strict DSH image, resource-contract and command-length gates pass.
The full KCC 256K D6FS-root guest passed DCS0 ROOT login, DSH ECHO,
external TRUE, orderly HALT and CLEAN postflight.

Alternatives evaluated and rejected: a signed division cast increases
the image to `036026`; a reciprocal multiply/shift yields `036002`;
constant-threshold decomposition yields `036032`; and unrolling six
clear stores yields `036001`. The retained bounded subtraction is the
smallest tested source implementation with the current KCC.

The three-word margin is real but narrow. Future compiler changes must
continue to run the exact image gate rather than assuming the earlier
v1735 headroom still applies.
