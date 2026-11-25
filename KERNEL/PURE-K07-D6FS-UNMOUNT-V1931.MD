# D6FS quiescence and verified mount retirement, v1931

Normal D6FS unmount now shares the existing DTFS owner-admission and
quiescence infrastructure. Its provider-disjoint mount media word uses
020000 octal as the temporary retirement barrier. The initial VFS owner
scan precedes acquisition without yielding. Ordinary D6FS provider
requests, raw recovery inspection and new FILE/cwd/PURE attachments
are denied while quiescing; only the VFS PREPARE operation is admitted.

PREPARE drains the serialized provider lease and verifies the original
selected physical superblock and UUID. A writable mount publishes CLEAN.
The last owner scan now occurs inside PREPARE, after physical I/O and
before the non-sleeping reader free. Failure retains the live reader and
identity; successful CLEAN followed by owner/free refusal leaves a live
read-only mount. Successful reader deletion retains the barrier until
VFS unpublishes the root. Abort clears only temporary quiescence, not
sticky write uncertainty. Ordinary unmount of a poisoned D6FS mount
is refused; no administrative discard or transactional repair implied.

REMOUNT and PREPARE construct the next superblock from the verified
selected header, not the potentially damaged alternate copy. Full
workspace scratch is cleared before physical proof. State publication
writes the opposite copy using the backing descriptor directly; the
private authorized state operation does not weaken the ordinary RO file
write gate. A failed transfer retains the selected sequence/copy/policy
and marks uncertainty; the existing root write failure classifier is
preserved. No additional physical readback guarantee is asserted.

Implementation 9175444e, testkit c7a950d. The extended v1927 native gate
links actual VFS, D6FS proof, CLEAN publication, admission and retirement
leaves against controlled physical callbacks, provider/lease completion,
MM and owner boundaries. Initial and late owner refusal, foreign UUID,
failed free, failed CLEAN, late refusal after successful CLEAN and final
retirement pass; 17 unsafe variants rejected. Extended v1562 rejects
eight unsafe variants. Earlier v1514, v1523, v1562 complete unmount,
v1566 DTFS seal, v1563 attachment and D6FS RW/RO regressions pass.
The old isolated remount test models selected-header proof and raw write;
the integrated native gate executes the real CLEAN publication leaf.

Folded lowmem tape KCC D6FS MRES 2378 -> 2392 words (+14); PURE 3879
unchanged. Fixed KCORE low end 025174 -> 025241 octal (+37 words);
BSS unchanged. No additional persistent mount or process data.

This is source-linked native transaction validation, not complete-guest
controller, physical media exchange or PI6 acceptance. K07 remains open
for executable mutation/admission lifetime, poisoned administrative
retirement and the remaining integrated lifetime audit. RAW PURE and
mode3 remain disabled by default.
