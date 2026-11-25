# D6FS mounted volume identity for PURE reconstruction (v1927)

D6FS uses the existing provider-disjoint mount slab: shape and extent
words retain the 72-bit mounted UUID, and the media word records an
unsuccessful physical proof. The 16-word reader and secondary mount
handoff ABI are unchanged. KINIT publishes the selected root UUID;
secondary mount reads the selected physical superblock before public
mount publication, verifying the admitted sequence, geometry and
CLEAN state. The old handoff has no UUID: this establishes identity
at mount publication, not at the earlier userspace scanner read.
Failed reader free retains identity; successful retirement clears
only the retired slot.

Private operation 030 (octal), serialized by the existing provider
lease, invalidates clean cache tags and clears the workspace header
before an uncached selected-superblock read. It verifies both saved
UUID halves, format, sequence, reader geometry and appropriate media
state. PURE calls it before and after executable reconstruction.
Failure preserves the old UUID, sets the mount proof-failure flag and
prevents new RAW DROP; restoration rollback retains the RAW suffix.
A successful matching proof clears the flag. An all-zero UUID cannot
authorize DROP. Ordinary filesystem I/O and terminal cleanup retain
their existing contracts.

The host mkd6fs formatter now acquires a fresh random 72-bit UUID
before the first formatting write and stores it in both superblocks.
It no longer derives IDs from names and volume size. This changes
bitwise reproducibility of new filesystem images but not the disk
format. Existing media are not silently modified.

Implementation 618f8ce1; testkit 83061a0. Twelve native physical-read
fault/retry cases, mount/reader lifecycle and four unsafe controls
pass. Two independently formatted equal-layout host volumes pass
fsck and have different UUIDs with matching A/B copies. The extended
v1925 bridge, v1926 DROP, earlier PURE admission/recovery and D6FS
mount/remount gates pass. Native tests use controlled raw read/MM/VFS
boundaries; actual disk exchange and PI6 acceptance remain open.

Folded lowmem tape KCC D6FS MRES 2184 -> 2320 words (+136); PURE MRES
3865 -> 3878 (+13); fixed KCORE byte-identical at 025174 octal. No new
persistent data or per-process storage. This is selected-copy identity
proof, not a fresh full A/B election or file-content authentication.
Same-ID clones, legacy deterministic collisions, and swap-and-return
between checks remain possible. TSFS set identity/generation, ordinary
provider loss, mutation/admission and integrated unmount remain K07
work. RAW PURE/mode3 default OFF.
