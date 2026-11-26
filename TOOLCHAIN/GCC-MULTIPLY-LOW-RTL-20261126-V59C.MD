# Multiply-low RTL input and destination constraints (v59c)

GCC b6371de6, on top of the v59b native record fix. The November
25 sweep contained 31 `extract_insn` ICEs with UNSPEC_MULSI_LOW:
22 in `960801-1.c` and 9 in `arith-ll.c`. Optimizer substitution
of the low-word input exposed the incorrect `match_dup 0` in the
canonicalize_mulsi_low pattern. The input and destination need
not have identical RTL before reload.

Represent the low-word input explicitly and tie it to the output
with the matching constraint `0`. Update both the mulsi3 and
truncdisi2 expanders. The high-word input remains independent.
The three-instruction TRNE/TLOA/TLZ output is unchanged; no new
target helper or ABI change is introduced.

Installed-compiler strict replay: 104 cells, zero ICEs. All 44
`960801-1.c` profiles pass, and all 16 registered profiles of
`pdp10-wide-multiply-36bit-v1.c` pass. All 44 `arith-ll.c` cells
compile but cannot link because the freestanding test harness
does not provide printf. This includes 9 former ICE cells and
35 previously recorded link failures. No semantic pass is
claimed for arith-ll.c; no test source or oracle was modified.

Thus all 64 original GCC ICE cells are eliminated by v59b/v59c
targeted replay, with 55 now passing and 9 blocked at link.
The original sweep is untouched. No full-matrix or extended
addressing acceptance run is claimed for this batch.

Evidence: `$HOME/tmp/gcc-mul-rtl-v59c.cLnJEw7f`.
Installed incremental cc1-only `local.tar.gz` SHA256:
`bbeedff54f473d3497306dac76c337be41dceff0c5ea81f933c15241b2263ba8`.
