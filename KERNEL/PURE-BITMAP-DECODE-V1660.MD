# v1660 PDP-6 PURE bitmap FULL decode preflight

Parent: pushed v1659 `ad64849f1bc6c416905533c0e90f2409616f4552`.
This is a test-only decoder admission proof, NOT a compressed owner.

The permanent two-bit storage map currently permits RESIDENT (0),
DISK (1) and uncompressed RAW PACKED (2), rejecting mode3. This is
intentional: terminal `pdp6_pure_detach_v1609` demands the exact RAW
suffix length, fast PREFIX requires in-place RAW, and FULL v1391
copies the uncompressed writable suffix. None can accept compressed
data until its ownership, restore and teardown lifecycle is complete.

The new opt-in C89 decoder preflights the entire bitmap without
writing output: exact source/target geometry, disjoint extents,
presence masks, nonzero literals, final short-mask padding, and
exact stream consumption. Only then does it invoke the original
30-word v1451 PDP-6 decoder. The native call explicitly passes its
four-AC assembly ABI (AC2 unused, AC3 output, AC4 decoded length).
No second decoded-size scratch buffer is allocated.

Source-linked KCC/PDP-6 tests encode and restore 73 RAW words with
three nonzero literals across three mask groups (six encoded words).
All malformed/truncated/overlong/overlapping cases preserve every
destination word. A separately compiled, executed mutant bypassing
exact-consumption verification is rejected. Original v1659 bitmap
and actual MM allocator tests also PASS.

This is NOT MM owner validation. A production caller must prove
exact single compressed MM owner, pinned disjoint APR target, source
admission and TRANSITION/PI6 exclusion. Mode3 publication, terminal
EXIT/WAIT/EXEC and full fault matrix remain unimplemented. No
physical 48K/64K compressed RAM saving or 32K acceptance is claimed.
PURE default remains OFF; 32K optimization deferred.
