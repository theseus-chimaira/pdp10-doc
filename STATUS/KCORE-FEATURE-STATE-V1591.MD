# KCORE P1-D v1591: two-member Type-551 TSFS root and eight live slots

Date: 2026-11-15. Full physical 256K KCC PDP-6 acceptance using the
production KINIT/FSINFO dynamic mount slab, original TSFS discovery
and original Type-551 DTFS provider. No KCORE/MRES/ABI/media change.

The existing physical acceptance v1532/v1533 used a three-member
TSFS root and a seven-slot mount inventory. v1591 separately formats
the actual TSFS root using **two** full tape members on DTC0/DTC1.
It explicitly attaches an empty DTC2 transport, not an old third
root member. The normal boot-time selected-root mask now leaves
six available Type-551 transports. KINIT publishes the real
eight-slot VFS capacity (`1 root + 6 spare tapes + 1 MEMFS`).

Four independent correctly stamped native DTVOL1 volumes are
physically mounted on DTC3..DTC6, each with distinct 72-bit media
identities. Original PID1 FSINFO verifies root TSFS provider 7,
capacity 8, native DTFS mount IDs 2..5, their exact pathnames,
upper empty slots 5..7, out-of-range 8 and 9, and stable high
mount ID 5 after the middle DTC4 mount is retired. It then retires
all remaining secondary mounts and verifies that the root and
capacity remain correct.

The complete physical guest returns PASS with all six required
markers `START`, `CAPACITY8`, `MOUNTED5`, `HOLE_STABLE`, `RETIRED`,
`PASS`. The first test-only target compilation included an unused
local warning, removed immediately after that run; an independent
fresh KCC source compile of the corrected probe is warning-free.

Source/runner and physical transcript:
`daimos-testkit/tests/validation/daimos-fsinfo-two-member-root-physical-v1591.md`.
DAIMOS state:
`design/kcore-p1d-two-member-tsfs-v1591.md`.

P1-D is not fully closed: distinct D6FS DSK/DRM physical root and
available-provider failure inventories remain open. The 26-slot
limit is a validated representation, not a requirement to simulate
26 real devices. P1-C mixed owner/provider fault tests remain open.

P1-E is explicitly paused for a later KA10-aware architecture
review. The previously planned PDP-6 APR-specific swapless PURE
drop/restore implementation is not approved for production or for
reuse on KA10 unchanged. Existing raw disk/drum swap remains intact;
no mutable-RAM compression implementation is authorized yet.

VERSION 0.5; production fixed low end `024364` octal; strict
32K optimization and placement remain on HOLD.
