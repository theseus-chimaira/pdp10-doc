# P1-E v1652: original PDP-6 64K nine-PACKED-owner acceptance

Date: 2026-11-17. Original 64K source-built PDP-6
TSFS-root / native secondary DTFS compressed PURE
guest lifecycle and separate physical MM replay PASS.
No production code/ABI/MM/MRES/RAM change.

The original v1643 C89 nine-child source expansion
RUNs one plain root and eight compressed secondary
DXR2 children; source-built three-member TSFS and
real DTC3 native DTFS media are used without changing
user syscalls, executable format, or MM allocation.
The first nine RUNs succeed, the tenth is refused
under real memory pressure; all original private
writable sentinels, PIPE exit057 and exact nine
PID/WAIT assertions pass, with one normal PC000000
HALT. A preliminary TEN-child expected-admission
test rightly failed when only nine were admitted,
not a KCORE bug.

The independently replayed original simulator MM
arena and sorted physical descriptors verify nine
distinct pin0 PACKED_PROCESS owners, process slots
2..10, each RAW 006600 octal (3,456 decimal)
words. The 64K managed arena is physical octal
042601..200000, exactly 47,743 decimal words.

| Original checkpoint | PACKED owners | Free decimal | Largest gap |
| --- | ---: | ---: | ---: |
| B | 9 | 9404 | 4155 |
| A | 9 | 9404 | 4155 |
| Z | 0 | 42769 | 42769 |

All nine B/A suffixes are unchanged. At Z, the
parent survives, all child PROCESS/PACKED owners
are released, and exact userland WAITs finish.
The testkit MM oracle rejects three independently
forged copies of the physical trace: incorrect
PACKED type, duplicate owner, and forged final
WAIT marker.

DAIMOS report:
design/kcore-p1e-64k-nine-owner-physical-v1652.md.
Testkit: tests/diagnostics/pure-64k-owners-v1652/.

P1-E still OPEN: this is not matched 64K PURE ON/OFF
net savings, complete continuous FULL transient
peak, 32K original physical acceptance or KA10 VM.
Unknown permanently pinned MM rollback remains open.
PURE default OFF, strict 32K optimization HOLD.
