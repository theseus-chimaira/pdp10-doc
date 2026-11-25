# Persistent PURE executable provider identity (v1924)

The resident backing record still holds index:18, low mount-slot:4,
PURE text:14. Stable credential word 066 keeps mount slot bit 4 at
bit 27 and original provider at bits 28..30. Provider values 4..7
retain their normal VFS meanings; zero is not a valid PURE provider.
No credential payload bit changes. SIDL stages the mount bit at
scheduler RH bit 0 and provider at bits 2..4; bit 1 remains the
unpublished VM discard marker. RUN copies these fields separately
from inherited UID/GID/UMASK. EXEC saves/restores the complete
four-bit identity with each old/new backing record, including before
old or staged VM destruction. Detach retires the saved identity.

RAW admission and the shared backing decoder independently require
the live mount provider to match the original provider. A different
provider reusing the mount slot is refused before reclaim or restore
can change the authoritative image. The v1923 transaction root check
still validates the complete binding during provider I/O. The existing
mount/backing owner scanners remain conservative and protect matching
mount/index regardless of a later provider mismatch.

Implementation 5466c1e6, daimos-testkit e950900. Nine substitution
cases, 16 RUN publication cases, three unsafe executed controls,
extended v1563 attachment and eight-case v200 EXEC commit/rollback
pass. Prior ownership, arena and terminal regressions pass. The new
native tests use controlled MM/VFS and actual source-linked leaves;
whole-guest PI6/concurrency acceptance is not established.

Folded KCC lowmem tape PURE MRES 3673 -> 3694 words (+21), fixed
KCORE +22 words (low end 025146 -> 025174 octal), BSS unchanged.
No permanent data or per-owner storage added. Same-provider media
identity, mutation/admission and unmount lifetime remain K07 work.
RAW PURE and mode3 default OFF.
