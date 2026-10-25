# File-runtime lock mount identity (v321)

The resident DAIMOS file runtime checks FLOCK conflicts using the
canonical vnode identity, descriptor owner, regular-file lock state,
and the two high mount-identity bits stored in the u-area extension.
The high mount bits must participate in both the conflict scan and
state-update scan. Otherwise distinct mounts with matching low vnode
fields could interfere with one another.

As of DAIMOS commit ac8e5ab (2026-10-25), the two scanning paths use
one private assembly leaf, `file_lock_mount_ext_v319`. It takes the
current descriptor-pair address in AC2, returns the two-bit mount
extension in AC0, and preserves AC4, which holds the packed descriptor
word. The public descriptor layout, vnode representation, lock ABI,
provider dispatch and lock semantics do not change. No new resident
storage or provider interface is introduced.

Full GCC lowmem disk-root links comparing committed pre-change source
with the new source: KCORE 024764 -> 024756 octal (10740 -> 10734
decimal words); KCORE+MRES 036520 -> 036512 octal (15696 -> 15690
decimal words); PERMANENT_LAST 036577 -> 036571 octal. Total saved:
six permanent words. MRES component sizes are unchanged.

The daimos-testkit GCC fixtures `target-file-pdp6`,
`target-file-walk-pdp6`, `test-daimos-vfs-stat-frame`, and
`test-daimos-pre-dsh-fs-closure` passed. Lock fixture coverage includes
shared/exclusive conflicts and unlock operations. A dedicated test
for two distinct mounts whose low vnode fields collide is a further
regression opportunity. Strict 32K acceptance remains deferred.
