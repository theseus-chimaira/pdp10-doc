# D6FS ordinary backing failure publication, v1929

The ordinary D6FS block reader and write-through commit path notify
PURE admission of an exhausted backing transfer. The existing mount
media word now uses bit 0 for READ_SUSPECT and bit 1 for
WRITE_UNCERTAIN. The failure notification checks the exact reader
pointer in the public mount slot, rather than inferring ownership
from the controller or mount number alone. It invalidates the shared
workspace and a private clean tag belonging to that source; an
independent private clean entry survives. No allocation or extra
persistent data. The original v1538 selected-root fail-stop and
secondary normal error policies are unchanged.

Private physical identity proof sets and clears only READ_SUSPECT.
A matching selected superblock and UUID cannot prove that a failed
write was atomic, so WRITE_UNCERTAIN refuses proof before physical
I/O. Both flags block further PURE DROP. A refused reconstruction
keeps the original authoritative RAW image. Terminal ownership cleanup
remains independent of media availability. Successful ordinary reads
are not themselves UUID proofs. No automatic all-resident secondary
recovery or general transactional write recovery is implemented here;
those lifetime/retirement policies require separate verification.

DTFS already records read suspicion/write uncertainty in its ordinary
transfer paths. TSFS is boot-only with the existing v1538 critical
root fail-stop policy and v1928 private identity proof. Neither
policy is altered by this change.

Implementation d63e1f98, testkit 9e762e5. Six new fault-publication
assertions, twelve original physical identity fault/retry cases and
ten unsafe controls pass. Extended DROP gate passes 118 cases and
rejects six unsafe controls. Native v1513/v1538/v1544/v1546/v1602
regressions pass with source-linked failure notification. These are
controlled I/O completion/MM/VFS fixtures, not full-guest controller
loss or PI6 acceptance.

Folded KCC lowmem tape D6FS MRES 2320 -> 2354 words (+34), PURE
MRES 3879 unchanged, fixed KCORE byte-identical at 025174 octal.
No BSS or per-process growth. RAW PURE/mode3 remain default OFF.
K07 remains open for secondary retry, write recovery policy,
executable mutation/admission and integrated unmount verification.
