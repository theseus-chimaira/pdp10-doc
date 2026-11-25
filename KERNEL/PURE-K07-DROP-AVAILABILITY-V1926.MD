# PURE DROP backing availability, v1926

RAW DROP must not discard the only resident executable text when the
kernel already knows the saved source is unavailable. The final
PI-excluded rebase admission checks the original provider and public
mount ID again, then checks its published service JRST and rejects
zero, malformed or no-service fallback bindings.

DTFS requires the mounted metadata slabs and rejects READ_SUSPECT,
WRITE_UNCERTAIN and QUIESCING. Saved stamped identity must be a complete
pair, and V2 must have one. An armed V2 must remain writable; legacy
media cannot carry the V2 arm bit. Valid legacy, stamped and armed
writable mounts remain admissible. D6FS requires a published mount-owned
reader, TSFS requires its published file/extent/member metadata. These
checks are non-sleeping and do not dispatch provider I/O. Refusal
preserves the entire resident image and original ownership; terminal
cleanup does not depend on backing availability. Already nonresident
restoration can still attempt the existing provider recovery path.

Implementation d6da62cb, daimos-testkit 122bb8f. Native 108 cases on
mount IDs 1 and 18 exercise all four providers, refusal and valid
admission, including controlled changes at the PI boundary. Five
unsafe variants fail native assertions. The existing admission,
recovery, backing-identity and DTFS bridge gates pass. No full-guest
controller/PI6 acceptance is claimed.

Folded lowmem tape KCC PURE MRES: 3707 -> 3865 words (+158), DTFS
MRES unchanged 2836, fixed KCORE byte-identical at 025174 octal.
No new permanent data or per-process storage. RAW PURE/mode3 default OFF.

Physical identity for D6FS/TSFS remains open. D6FS superblock validation
knows the 72-bit UUID but the runtime reader deliberately omits it.
TSFS boot validates set ID and generation, but the three-word runtime
handoff retains only file/extent locators and member map. A healthy
publication is therefore not evidence of actual physical continuity.
The availability check must not be described as media authentication.
