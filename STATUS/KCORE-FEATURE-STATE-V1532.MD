# KCORE v1532: FSINFO runtime capacity accepted

Date: 2026-11-13. Feature correctness only, no optimization.

KINIT already sizes and publishes six MM-backed mount arrays from
validated controller and selected-root inventory. The v1492
read-only FSINFO extension is now accepted and integrated:
AC2=-1 returns the published positive capacity without mapping
AC3; ordinary slot results remain 1 occupied, 0 empty and
-1 error. MOUNTS and DF query once and scan the complete capacity
instead of a fixed four mounts. Errors are not confused with holes.

Native PDP-6 source-linked FSINFO tests PASS, including zero
capacity, an empty last slot, the first invalid slot and
the 26-slot representational boundary. The intentionally broken
capacity-comparison mutation is rejected. Host C89 tests use
the actual MOUNTS/DF function bodies and PASS for capacities
1, 4, 8, 17, 26, noncontiguous occupied slots and provider errors.

The fresh 256K KCC PDP-6 TSFS-root image physically boots with
three selected DTC0-DTC2 root members, five other DTC positions
and MEMFS capability, publishing **7 actual mount slots**.
The real guest `FSINFOCAP V1532` obtains this count with
a null result pointer, verifies TSFS root provider 7,
mounts stamped DTC3 native provider 5 in public ID 2,
scans all slots 0..6 (including empty 2..6),
rejects out-of-range calls, unmounts DTC3,
and verifies slot retirement without capacity loss.
Guest and full host runner PASS.

This is not proof of a live 26-volume configuration or a
physical mount at index 4 or above. Those are separate P1-D
hardware/full-guest acceptance requirements.

The v1532 fresh KCORE map has identical low-init/end
addresses to the compared locally built v1531 image:
`023341` and `023756` octal. The DTFS MRES
reports 2095 initialized and two BSS words on both builds.
This same-size comparison is not a controlled isolated
feature-size attribution; no new permanent mount table is added.

Details: `DAIMOS/design/kcore-fsinfo-runtime-capacity-v1532.md`
and `daimos-testkit/tests/validation/
daimos-fsinfo-capacity-physical-v1532.sh`.

Remaining: P0-A qualified permanent root/BACKSTORE loss;
P0-B durable discarded-volume admission; P1-C mixed-provider
errors; P1-D high physical mount occupancy and cross-inventory;
P1-E swapless PURE/MM/APR; final feature regression.
VERSION 0.5 and strict 32K/optimization gate HOLD.
