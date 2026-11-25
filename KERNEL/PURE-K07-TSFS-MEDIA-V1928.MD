# TSFS mounted physical set identity, v1928

The internal boot handoff now contains eight words: the existing
three locator/member-map words and five admitted identity words
(set ID high/low, generation, table-directory locator and checksum).
The optional tape MRES stores these five words once for the TSFS
boot root. No new VFS mount-array or per-process storage. KINIT's
extra five handoff words are reclaimed with its transient image.

The serialized private operation 030 verifies the physical descriptor
copies of every configured member without using clean block cache hits.
It clears the transfer scratch before each read, validates the full
128-word descriptor seal and compares the admitted identity, geometry
and expected logical member ordinal. An unreadable/unsealed copy is
tolerated if the other matches; a sealed malformed or contradictory
copy causes refusal. The existing member map carries the suspect bit;
failure retains the identity and prevents further PURE DROP, and
exact successful retry clears that bit. The shared PURE bridge calls
the operation before and after reconstruction, retaining RAW on refusal.
MEMFS requires no physical proof.

Implementation 2e096e30 and testkit 2348091. Forty native descriptor/map
cases and seven executed unsafe controls pass. The boot handoff gate
covers one/two/eight members; the shared bridge and DROP, stored
cross-member/extent, parent and root failure gates pass. Historical
stored-extent runners link the actual invariant leaf. The obsolete
v1 compressed-extent runner still references removed d6lz_pdp6.s and
is not counted as passed. Tests control raw transfers and VFS/MM
boundaries; they do not establish full-guest media exchange or PI6
acceptance. The proof authenticates descriptor identity against saved
metadata, not the actual table or payload; same-ID clones and physical
swap-and-return between checks are outside its guarantee.

Folded lowmem tape KCC DTFS/TSFS MRES 2836 -> 2985 words (+149,
five additional BSS); PURE 3878 -> 3879 (+1). Fixed KCORE byte-identical
at 025174 octal, no additional process words. Actual physical I/O
cost rises by up to 32 descriptor reads per full reconstruction
(two copies on eight members, before and after loading); no speed
measurement claimed. K07 remains open for provider-loss, executable
mutation/admission and unmount lifetime. RAW PURE and mode3 default OFF.
