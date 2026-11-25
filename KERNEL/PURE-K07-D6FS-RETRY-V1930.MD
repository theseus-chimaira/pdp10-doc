# D6FS ordinary secondary identity recovery, v1930

The ordinary D6FS register dispatcher examines the existing mount
media word under the serialized provider lease. Healthy requests
continue without physical identity I/O. READ_SUSPECT invokes the
v1927 selected-superblock physical identity verifier. The six
request registers AC1..6 are saved on the existing kernel stack
across potentially sleeping verification. Only success permits the
original request to proceed. A matching original UUID and admitted
header retires read suspicion; foreign or inaccessible media retains
the flag and refuses ordinary dispatch. This also permits PURE DROP
eligibility to return when every affected executable owner is resident.

WRITE_UNCERTAIN, combined or unknown state refuses ordinary dispatch,
including SYNC, REMOUNT and PREPARE_UNMOUNT. Matching identity is
insufficient to prove that an earlier failed write completed safely.
Privileged raw recovery inspection retains its existing separate
syscall authorization and does not clear uncertainty. The private
PURE proof also refuses unknown state. The existing selected-root
ordinary transfer fail-stop classifier is unchanged.

Implementation 4975f130; testkit 24e0812. The native source-linked
v1927/v1929 gate now verifies foreign-media refusal, ordinary exact
retry, preserved request arguments, healthy no-proof path, poisoned
operation refusal and raw inspection. Thirteen unsafe variants are
rejected. Historical selector, multimount, reader-slab and RW/RO
transition regressions pass. Physical transfer and vector destinations
are controlled boundaries; no whole-guest media exchange or integrated
unmount/PI6 acceptance. The test's independent reader retirement
exercise is not a write-recovery procedure.

Folded lowmem tape KCC D6FS MRES 2354 -> 2378 words (+24); PURE MRES
3879 unchanged, fixed KCORE byte-identical at 025174 octal. No extra
persistent or per-process data; six temporary stack words for the
suspect request. RAW PURE and mode3 remain default OFF.

K07 remains open for integrated quiescence, mutation/owner admission
and retirement. D6FS PREPARE currently releases the reader before
VFS's final owner check; this ordering needs an integrated fix.
Uncertain writes remain quarantined rather than transactionally
repaired, and safe administrative retirement still needs validation.
