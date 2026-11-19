# PDP-6 v1657: real 48K PURE PACKED source bitmap density

Date: 2026-11-17. An original source-built PDP-6
48K/no-AUX TSFS-root KCORE with genuine secondary
DTC3 DTFS compressed PURE executable was run
UNMODIFIED through all original PIPE/EXIT-057/WAIT
and normal HALT. A read-only simulator capture
at the authentic six-owner PACKED checkpoint
examined every one of 20,736 original 36-bit
retained words, not a fabricated BSS pattern.

The six source MM owners were derived exactly
from the separately accepted v1654 two-word
physical MM extent table. Each RAW source has
006600 octal / 3,456 decimal words, 65 nonzero
literals and 3,391 zero words. Under the already
native-tested v1451 bitmap codec, exactly 96
36-bit masks cover each 3,456-word original.
Masks plus 65 literals total **161 encoded
words**, compared with the real 3,456-word RAW.

| Original physical source | RAW | Bitmap candidate | Gross reduction |
| --- | ---: | ---: | ---: |
| Each of slots2 through 7 | 3,456 | 161 | 3,295 |
| All six owners | 20,736 | 966 | 19,770 |

The independent testkit auditor checks every
original 36-bit physical word/address, performs
exact presence-mask encoding and complete
word-for-word reconstruction, and authenticates
the two observed private-data content hashes.
Slot2 has three words different from slots3..7.
Three deliberately falsified witnesses are
rejected: changed private word, missing memory
address, and forged final HALT.

Testkit branch:
test-kcore-pure-raw-density-v1657-20261117,
commit 5cbe205.
Physical test entry:
tests/diagnostics/pure-suffix-entropy-v1657/run-v1657.sh.
Full DAIMOS report:
design/kcore-p1e-physical-suffix-density-v1657.md.

Evidence:
$TMPDIR/daimos-kcore-v1657-evidence-20261117.tar.gz
SHA256 c12efb2caa5d489b3638d8d34f95fec6ca4ecf92fd0327ca10e798a90a24e3f7.

**19,770 is potential data-only reduction, NOT
an achieved KCORE allocation or physical
peak-RAM saving.** No encoded MM extents, mode3
owners or compression transaction have been
activated. Source and destination must remain
separate for the v1451 codec and be owned
failure-atomically. The tested 30-TEXT-word
codec, MM metadata, transient disjoint output,
decode to full APR, MRES and BCACHE scratch
must all enter any actual net-memory report.

The data SUPPORTS developing and testing an
optional failure-atomic bitmap MM prototype
instead of prematurely selecting NO COMPRESSION.
Do not enable the feature before native/physical
fault, PI6, lifetime, restore, owner accounting
and full peak/ON-OFF net measurement pass.
Strict 32K boot/size acceptance remains DEFERRED;
PURE is still opt-in and OFF by default.
