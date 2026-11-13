# KCORE features v1533: real high physical mount ID

Date: 2026-11-13. KCORE feature acceptance, not optimization.

The v1532 native query and fresh full physical boot established
the actual KINIT seven-slot dynamic mount slab. v1533 now verifies
active ownership **beyond four mounts** on actual hardware.

The full 256K KCC PDP-6 guest boots from TSFS root DTC0-DTC2
and mounts **four independent native DTFS volumes** from
separately attached Type-551 DTC3 through DTC6. The host verifies
the four complete nonzero and pairwise-distinct 72-bit DTVOL1
identities. The real guest validates FSINFO provider/path/ID for
four secondary mounts with public IDs 2, 3, 4, **5**.

FSINFO sees root plus four DTFS mounts, two empty upper slots
and an out-of-range error. The guest unmounts middle mount ID
3 while keeping high ID 5; its original pathname and ID still
resolve correctly. All remaining secondary mounts retire
successfully and leave the TSFS root and seven-slot capacity
unchanged.

Guest reports `FSINFOHI V1533 MOUNTED5`,
`SCANNED7`, `HOLE_STABLE`, `RETIRED` and
`PASS`. Host runner exits zero. Independent physical v1532
seven-slot/single-DTC mount regression also PASSes after the
test-harness extension.

The additional physical tapes, SIMH attachment commands and
target acceptance source live **only in daimos-testkit**.
The production KCORE, installed MRES, original KCC image,
on-media format and permanent RAM are unchanged.

The 26-mount representational/planner ceiling does not mean
the current DTC-root guest physically provides 26 devices.
Cross-inventory acceptance remains outstanding.

See `DAIMOS/design/kcore-fsinfo-high-physical-v1533.md`
and `daimos-testkit/tests/validation/
daimos-fsinfo-high-mount-v1533.md`.

P0-A confirmed critical media loss, P0-B durable quarantine,
P1-C mixed faults, P1-D additional real inventory profiles,
P1-E swapless PURE/MM/APR and final feature freeze remain open.
VERSION 0.5; strict 32K and optimization HOLD.
