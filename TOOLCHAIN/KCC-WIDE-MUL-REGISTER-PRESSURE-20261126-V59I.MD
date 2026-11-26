# KCC modular wide multiply with constrained accumulators (v59i)

KCC 9f8c05e; testkit d88d5a16. All 80 remaining multiplication
allocation ICEs from the November 25 sweep are eliminated:
MUL.c (18), smulsi3_highpart.c (18), 930921-1.c (22), and
921118-1.c (22). GDB confirmed that the latter two scratch
diagnostics also originate in gdimodemul. The old code required
four operand ACs, an adjacent product pair and one scratch AC.
Incoming ABI reservations can leave too few volatile ACs even
after spilling every unrelated expression value.

For radix B=2^35, the product modulo 2^71 is
alo*blo + B*(ahi*blo + bhi*alo + B*ahi*bhi). Save ahi parity in
the unused sign bit of alo, incorporating the last term into
the second cross-product. Reuse the two dying operand pairs
for all three MUL instructions. Reconstruct the full modular
low36 cross-product from the low35 word and bit 0 of the high
word. IMUL's overflow result has the mathematical product's
sign, so it cannot substitute for this extraction. Restore
the canonical original alo before the final low-word product.

Only one saved word is needed outside the two operand pairs:
a free volatile AC when available, otherwise one balanced
transient stack slot. The original ABI masks and preserved
register protection remain; no new runtime helper or permanent
target storage. The full modular sequence uses 20 emitted
instructions with an AC, 21 with the stack slot, versus 16 in
the old incorrect general sequence (excluding allocation
spills). Target cycle counts have not been measured.

The first experimental scratch-free IMUL candidate passed the
150-cell historical selection but failed every profile of the
new high-word boundary regression. It was rejected. The final
full-MUL candidate passed all 150 historical/adjacent cells,
including the original 80 ICEs and the v59h divider regression.
Of those 80, 22 are actual runtime passes (930921-1.c) and 58
registered codegen passes. The new independently calculated
high36:low35 regression passes 22/22, including a verified
stack-slot path with three incoming AC reservations. Its
initial fixture reproduces the baseline allocation ICE.
Final acceptance: 172/172 selected cells, no internal errors.

Evidence: `$HOME/tmp/kcc-mul-pressure-v59i.qrIdmgCe`, final
replay-v2 and regression-v3. Installed bin/kcc SHA256:
`9be0dd00c470b5d65eed3e355d8f15ed459daecc067b5b84359f81942bc98c86`.
Incremental `install-v2/local.tar.gz` SHA256:
`403d818674d5a153c2657a1adaa4b9855f196e7b164dc93e870efdd7849ec307`.
Original sweep is unchanged; no full-matrix, extended addressing,
self-host compiler, or DAIMOS source change is claimed.
