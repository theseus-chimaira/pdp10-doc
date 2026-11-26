# KCC full-word indexed displacement range (v59j)

KCC a9add7c; testkit 5dec8402. The November 25 sweep's last
44 original ICEs came from SFset.c, SIcmp.c, SIset.c and USIcmp.c
(11 optimized profiles per source). GDB confirmed outaddress,
called during final PCODE emission, diagnosed offset 01036421.
Despite the historical diagnostic wording, the failing address
was not necessarily stack-relative.

PCODE PTV_IMMED can describe a full-word literal, while the
indexed operand's displacement is limited to 18 bits. code40
and foldxref must not consume an ADD/SUB of a large constant
when forming an indexed operand. Retain the arithmetic and
the following zero-displacement access for out-of-range values;
keep the existing small displacement optimization. No change
to the final emitter's consistency diagnostic. In foldxref,
validate before negating a subtraction operand.

All 88 original-source codegen cells pass in replay-v2. The
new kcc-index-offset-v59j fixture reproduces the baseline ICE
and passes all 22 native local-address execution profiles.
It deliberately tests local PDP-10 effective-address wrapping,
not portable ISO C pointer arithmetic or extended addressing.
It covers positive/negative wide offsets, the 0777777 boundary,
small offsets, stores and adjacent sentinel words. Five existing
related sources contribute 36 runtime passes and 54 compile_only
successes. Total 200 selected cells, 146 pass and 54 compile_only,
no failures. Codegen pass is not a runtime assertion.

Evidence: `$HOME/tmp/kcc-index-offset-v59j.04Xx7d7q`.
Installed bin/kcc-only local.tar.gz SHA256:
`c2a033f3e21555df3ba20965b342bddd13a926d99cfab25bc932aa0dc4f9ad0c`.
The initial replay invocation used shell brace expansion unsupported
by the remote shell; the explicit source list and replay-v2 are
the successful evidence. The full-sweep baseline is unchanged.
All original compiler ICE clusters have now been individually
replayed, but no fresh whole-matrix result is claimed.
