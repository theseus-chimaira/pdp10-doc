# MM allocation publication and disk PI continuity v1742

Date: 2026-11-21. DAIMOS commit `03dcb0e9`.

`mm_alloc_aligned_noreclaim` performs its free-gap search and
descriptor insertion as one atomic MM ownership transaction. The
older implementation used `CONO PI,0400` to disable global priority
interrupts during that entire operation. A full 256K PDP-6 KCC
D6FS-root login reproducibly failed with Type 270 data-request-late
(DRL, status bit `000020`): the DCT needed PI3 service during a
roughly 1360-tick global PI-disabled interval. The failed read was
logical D6FS block `0743`; the root-specific fatal HALT was correctly
reached at octal PC `004270`.

The v1742 implementation inhibits scheduler PI6 only (`CONO PI,1002`)
while searching and publishing the MM descriptor, leaving disk PI3
and PI5 enabled. It saves the original PI6 enable bit from CONI PI
and restores PI6 (`CONO PI,2002`) only if previously enabled.
Calls entered with PI6 already inhibited retain that state.
The modification changes no instruction count and adds no static,
heap, or per-process storage. The fatal-root error policy is unchanged.

The native PDP-6 v1742 allocation test passes and rejects three
unsafe mutants: absent PI6 exclusion, overwritten saved AC16, and
global PI masking. A fresh full 256K KCC disk-root guest passes
DCS0 ROOT login, DSH ECHO, external TRUE with status zero, orderly
HALT and CLEAN D6FS postflight. See testkit report
`daimos-d6fs-root-pi6-accept-v1742.md`, testkit commit `eb1a66f`.

The separate 32K KCORE memory budget and a complete post-change
K03 physical OOM rerun remain outside this acceptance.
