# KCORE v1550: native DTVOL2 validator and explicit host format

Date: 2026-11-13. P0-B staged implementation; P0-B OPEN.

The experimental six-word RBN0 `DTVOL2` record now has a common
format header and actual PDP-6 assembly validator, independent of
the existing DTVOL1 mount code. It checks exact 36-bit magic,
both nonzero 36-bit identity halves, identity XOR-complement,
CLEAN/WRITE_INTENT state, a second state+salt complement, and
122 zero padding words. Input is a physical 128-word RBN0 buffer;
return is `0` CLEAN, `1` DIRTY, or `-1` INVALID. This helper
does not write media, retain owners or allocate permanent RAM.

The production host `mkdtfs` supports the new record **only**
with `--durable-v1550`. The default `mkdtfs -o ...` remains
unchanged DTVOL1, with the same native directory/owner layout.
The Makefile tracks the new header dependencies. No DTVOL2
record is silently produced by existing build scripts.

Testkit runs actual host formatter output through real PDP-6
native `p10run` code: 13 records (new clean, dirty, independent
identity, unsupported legacy and torn/corrupt cases) PASS and
seven wrong-source parser mutations are rejected. A harness
bug initially treated large 36-bit octal constants as decimal;
leading zeros now preserve exact host-image words, and the
unaltered native validator PASSes. Full v1520 host DTVOL1,
native mount and target MKFS compatibility regression PASS.

**Deliberately not active in production mounted DTFS:** there
is no DTVOL2 admission, durable dirty-before-write readback,
checked clean unmount, physical Type-551 partial-write or
cold-boot recovery acceptance yet. Experimental DTVOL2 media
MUST NOT be treated as supported mounted filesystems. P0-B
remains OPEN; P0-A is complete; P1-C remains independent.

DAIMOS `design/kcore-dtfs-intent-source-v1550.md`; testkit
`tests/validation/daimos-dtfs-intent-native-v1550.md`.
VERSION 0.5, strict 32K/optimization HOLD.
