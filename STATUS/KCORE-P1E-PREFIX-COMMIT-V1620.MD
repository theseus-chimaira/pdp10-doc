# P1-E v1620: physical PURE prefix commit and scheduler selection

2026-11-16. Original 48K PDP-6, TSFS root,
no AUXST1, actual opt-in PURE DXR2 process.
Physical MM and scheduler milestone accepted;
full user-mode and RAM benefit remain open.

Three replays against the same preserved guest
image and cloned DECtape media prove:

1. Real VFS prefix loader success followed by
   original v1384 commit entry at PC 034614,
   owner 2, APR base 116000, PURE prefix 001000,
   full APR 010000 octal words.
2. Actual MM descriptor writes at 024307:
   000014000002 (PACKED pin1) becomes
   000011000002 (PROCESS pin1), then
   000001000002 (PROCESS pin0).
3. Real v1612 PURE service sets scheduler
   cursor 001002 (READY_ONCE|slot2) at
   PC 032477; fixed KCORE then writes
   proc_current_slot=2 at PC 016770.
   The owner stays PROCESS and unpinned.

The testkit checks the exact linked original
PURE MRES map/DXR instructions, call argument
registers and physical owner descriptors.
Unsafe mutations to owner, pin, READY_ONCE
or current-slot state are rejected. There is
no modified kernel image or new permanent
process metadata, KCORE or MRES allocation.

The source-backed cost gate is still open:
three remaining omitted 001000-word prefixes
total 1536 decimal gross words while the
installed GCC PURE MRES occupies 1758 words,
not counting other transient allocations.

Need actual user-context resume,
PIPE/WAIT/EXIT and source lifetime with a
fitting memory-pressure layout; then
precise net/peak physical RAM acceptance.
No default P1-E enablement is authorized.

DAIMOS:
design/kcore-p1e-physical-prefix-commit-v1620.md.
