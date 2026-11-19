# v1661 compressed PURE terminal MM release, test-only

Parent: DAIMOS v1660 commit `6a78fb79fa2fa19b4e986a7e085fb2d4c3776134`.

## Exact ownership

Mode3 is the reserved final two-bit state for compressed PURE process
memory. A single unpinned `MM_TYPE_PACKED_PROCESS` extent belonging
to the process slot already records both its physical base and its
actual compressed word count. Therefore no second registry or extra
per-process descriptor is required for terminal deletion.

The v1661 opt-in C89 terminal leaf validates TRANSITION, stable
u-area, matching saved immutable PURE backing, absent user APR,
original APR alignment, valid omitted prefix, exact mode3, and
one (only one) packed-process MM owner. The encoded extent must
cover at least one presence bitmap per 36 RAW words, must be
strictly smaller than RAW, unpinned, inside 18-bit physical core.
Even one duplicate PROCESS owner fails closed.

Under PI exclusion, the original PDP-6 `mm_free` verifies physical
type/owner/pins again and removes the packed descriptor. If it
refuses, the two-bit mode and MM descriptor remain unchanged.
After success the same PI-disabled critical section only clears
the exact two-bit mode3 pair, which cannot fail. The terminal
caller still owns the swap record, u-area and TRANSITION until
subsequent successful finalization; this leaf does not clear them.
No executable vnode or surviving mount is required for terminal
release of an inactive process. Terminal discard need not decode
the dying image; restore must separately verify every input word.

## PDP-6 validation

The new native test links the actual source-extracted PDP-6 MM-free
assembler, not a C simulation of MM ownership. A 161-word packed
extent corresponding to 3,456 RAW words is released successfully;
only mode3 is cleared and all non-MM process records retained.
Eighteen invalid state cases retain MM/mode, including a late
injected MM-free refusal and an illegal reserved-type extra owner.
An unrelated MODULE owner with the same numeric ID remains owned.
An extent ending exactly at exclusive
`01000000` octal succeeds while an overflow fails. An independently
compiled source mutant disabling duplicate-owner checks is run and
rejected. A controlled PI fixture models exclusion; physical PI6
and full terminal/wait integration have not been exercised yet.

The prototype compiles to 314 TEXT words and zero DATA/BSS; not
installed cost. The unlinked v1660 decoder also corrects its
exclusive-end geometry to accept the physically legal final word.
Original v1660/v1659 bitmap/MM regression remains PASS.

## Remaining blockers

Permanent storage-mode checks still reject mode3, and the current
`pdp6_pure_detach_v1609` is strictly RAW-only. A production
compressed owner requires full fail-atomic RAW retirement and
bitmap publication, bounded FULL restoration, terminal dispatch,
scheduler PI6 failure policy and matched 48K/64K physical tests.
No physical compressed RAM saving or strict 32K acceptance claimed;
PURE default OFF.
