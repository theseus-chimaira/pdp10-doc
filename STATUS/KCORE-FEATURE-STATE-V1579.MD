# KCORE v1579: native P1-C snapshot/BACKSTORE owner interleaving

Date: 2026-11-14. Accepted source-linked native PDP-6 transaction
test. Overall P1-C remains OPEN. P0-A and operator-trust P0-B remain
CLOSED. VERSION 0.5; 32K/optimization HOLD.

The versioned daimos-testkit native runner compiles the actual
production `memfs_snapshot_pdp6.s` and `backstore.s` sources. It
uses test-only MEMFS metadata, a 512-block synthetic backing medium,
and read/write callbacks. A preexisting process owns block zero;
the original snapshot mount reserves physical blocks 474..511.

The original snapshot shutdown passes through invalid-header,
metadata and final-header writes. While processing the first two
write callbacks, the runner invokes genuine BACKSTORE allocation
for process runs of 90 and 18 sectors. All 512 bitmap positions,
physical write arguments, commit order, version/geometry and exact
used-block counts are verified. All 147 blocks remain allocated
before temporary process release, and 39 remain after; the original
process and entire MEMFS snapshot reservation are preserved.

Three independent executed native source mutations are rejected:
clearing snapshot reservation bits, skipping the final header, and
ignoring allocated bits in the process allocator. Both the passing
baseline and rejected mutations use exact `__test_exit` acceptance,
not a script's impression of successful assembly or static grep.

No production KCORE/MRES, MM, ABI or persistent-storage format change.
`__kcore_low_end` remains `024364` octal. The I/O endpoint is a
test-controlled native stand-in: there is no actual physical DMA,
independently scheduled swap, natural controller fault or simultaneous
process swap and MEMFS HALT. **P1-C remains OPEN** for these physical
concurrency and DEVICE/VOLUME/PI cases. See DAIMOS
`design/kcore-p1c-snapshot-interleave-v1579.md` and testkit
`tests/validation/daimos-p1c-snapshot-interleave-v1579.md`.
