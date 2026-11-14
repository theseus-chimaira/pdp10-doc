# KCORE v1575: P0-B closed under operator-trust DTVOL2

Date: 2026-11-14. VERSION 0.5; 32K optimization HOLD.

The DAIMOS release policy now explicitly trusts the operator not to
remove or exchange a DECtape reel during any writable VFS mount. A
poisoned mount must first be abandoned through root's destructive
owner-checked administrative discard. This is NOT a mechanical reel
interlock: v1571's silent-exchange negative hardware witness remains
valid, but no longer blocks the accepted operator-trust profile.

Production V2/CLEAN RW was enabled using the existing genuine
WRITE_INTENT-before-DATA barrier, saved 72-bit identity, MM-pinned
scratch, physical per-write RBN0 verification and quiesced final
DIRTY->CLEAN seal/readback. Physical PDP-6/Type-551 guest and a
second full production cold boot PASS. A genuine one-word partial
DATA write remains sticky WRITE_UNCERTAIN and refuses existing
FILE/cwd/UNMOUNT; a further cold boot rejects the same physical
DIRTY reel both RO and RW. Another guest physically restored an
armed RW/DIRTY tape after an observed read error and safely CLEANed.

The previous v1531/v1570 different-reel replacement verifier is
removed. The same private provider operation 026 now performs only
serialized **DISCARD_DRAIN** plus BCACHE/directory tag invalidation.
The root-only syscall ABI remains unchanged; its old REPLACE_SLOT
spelling now means discard the poisoned published owner, not validate
or repair another reel. The VFS still checks every FILE, cwd, PIPE,
swap and retained MM owner both before and after the possible provider
lease wait. It retires slot, media, both saved ID halves and FILE
unlock together only on success. An independent normal new MOUNT
performs fresh native directory/RBN0 checks.

Real PDP-6 physical partial-DATA guest discarded the owner while
the exact same original DIRTY reel was physically attached. After
discard, both ordinary RW/RO remounts of that reel refused. Only
after operator replacement did a separately formatted V2/CLEAN reel
mount RO, refuse CREATE, and unmount. Independent complete-image
SHA-256 checks proved both reels unchanged by the discard and an
additional cold boot again refused the old DIRTY reel. Actual VFS/
FILE source-linked all-process owner tests and four unsafe mutations
PASS/reject. The new provider drain source-linked test and two
unsafe cache mutation executables PASS/reject.

Same-profile optional DTFS MRES output shrank 23,760 -> 23,048 bytes,
89 packed 36-bit words. Fixed KCORE `__kcore_low_end` is `024364`
octal unchanged. No new permanent RAM, API, MM slab, format or root
driver. Earlier v1574 fixed the genuine Type-136 reverse-write overrun
independently (+1 optional tape MRES instruction).

**P0-B CLOSED only under the documented operator-trust DTVOL2 policy.**
No automatic media continuity detection, no physical interlock, no
automatic repair of DIRTY reels and no durable rejection of an
abandoned legacy DTVOL1/V1 reel without operator quarantine. P0-A
closed, P1-C/D/E remain open. Authoritative DAIMOS design:
`design/kcore-dtfs-operator-discard-v1575.md`.
