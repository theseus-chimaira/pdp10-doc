# KCORE v1547: shared Type-236 root/BACKSTORE physical acceptance

Date: 2026-11-13. Actual full-guest PDP-6 hardware-simulator
error path, not optimization or physical removal detection.

A complete KCC 256K PDP-6 with `ROOT=drum BOOT=drm` boots from
the selected four-member D6FS DR0..DR3 root. BACKSTORE is a
real mapped BLOCKSET VOLUME on those same physical Type-236 units.
A live owned BACKSTORE WRITE request completes at the original
Type-236 PI2 path; a disposable selected-request observer routes
the completion into the real driver error handler. The production
v1541 fail-stop gateway then HALTs at the **swap** PC `003747`,
not root PC `003743`.

Linked post-HALT physical words confirm one selected PI2 error,
BACKSTORE enabled 1, owned blocks octal `0320`, physical selector
`400000000000` (DRM), volume data blocks octal `075774` and
selected root `060101000000`. Boot switch 3 selects the DRM root;
the physical DR0 image is attached to that root. Four clean
pre-error samples and nine corrupted-evidence mutations PASS.
No KCORE code, RAM, ABI or on-media format changes were needed.

This proves **BACKSTORE-first** physical error escalation with
shared ROOT media. It is a selected **injected PI2 error branch**,
not evidence of spontaneous drum disappearance. The opposite
**ROOT-first physical failure with live swap owners** remains P0-A.
Native v1546 covers both logical orders but not physical DMA
arbitration. References: DAIMOS
`design/kcore-shared-drm-root-backstore-v1547.md`, testkit
`tests/validation/daimos-shared-drm-root-backstore-v1547.md`.
VERSION 0.5, strict 32K/optimization HOLD.
