# KCORE v1582: source-linked real BLOCKSET VOLUME mapper verified

Date: 2026-11-15. Actual native PDP-6 integration of three real
production assembly components; this is NOT the full physical mapped
VOLUME/controller acceptance. P0-A and operator-trust P0-B remain
CLOSED; P1-C remains OPEN. VERSION 0.5, 32K/optimization HOLD.

`daimos-testkit/tests/validation/daimos-p1c-volume-realmap-v1582.sh`
links the unmodified `system/kernel/storage/blockset_dispatch.s`
INTERLEAVE mapper, the real fixed `fs_mres.s` backing gateway and
the actual v1515 critical-media HALT dispatch. A three-member
INTERLEAVE root descriptor is loaded into the original BSET MRES
storage, and the linker-resolved real `blockset_map_block` service
is installed in `blockset_runtime_service_call` just like MINIT.
The test controls only the bottom physical controller completion.

Member units 1/2/3 with distinct physical bases map the first six
tail sectors in exact sequence (1,0300), (2,0400), (3,0500),
(1,0301), (2,0401), (3,0501). The native fixture asserts the unit,
block, DSK/DRM controller selection, transfer direction, 0200-word
buffer stride and retained ownership after every transfer. Thirteen
full native cases PASS, including DSK/DRM multiblock transfers,
second/third-block fatal failures, disabled-but-owned fail-stop,
disabled-and-unowned ordinary error, missing mapper active/inactive,
range overflow/end and zero-length transfers. Fatal HALT must be
the linked reason-2 BACKSTORE address, never the ROOT reason. Eight
actually executed unsafe source mutations are all rejected: wrong
physical block, member, data offset, DRM route, missing fatal,
missing-mapper bypass, ignored owned-block gate, or a 0100-word
instead of 0200-word transfer-buffer stride. Every physical
completion compares its original buffer pointer to the expected
per-sector address.

Unlike older controlled fake-mapper tests, v1582 uses the actual
production BSET mapper; however, it does NOT run a physical
multi-member PDP-6 guest or observe natural DSK/DRM PI completions.
P1-C remains OPEN for those real physical mapped-VOLUME faults,
independent PI-driven swap-in during MEMFS snapshot shutdown,
and precise recoverable owner/BCACHE/MM rollback. P1-D alternate
physical boot inventories and P1-E uncompressed PURE are also OPEN.

No production code, fixed RAM, ABI or disk-format changes; KCORE
low end remains `024364` octal. Full engineering evidence:
DAIMOS `design/kcore-p1c-real-blockset-map-v1582.md`, testkit
`tests/validation/daimos-p1c-volume-realmap-v1582.md`.
