# KCORE v1543: physical DRM236 D6FS root HALT accepted

Date: 2026-11-13. Source-backed feature acceptance, not optimization.

The pre-existing KINIT root selector supports class `KINIT_ROOT_DRM`
and the existing D6FS blockset boot path maps four genuine Type-236
drum units. The new daimos-testkit v1543 physical test builds a
complete 256K KCC `ROOT=drum BOOT=drm` PDP-6 image, with four
populated D6FS drum media produced by the normal production tools.
SIMH switch octal 3 selects this drum root, independently of the
also attached Type-270 disk units.

After successful `/SYSTEM/INIT`, `/SYSTEM` and `/CONFIG` root stat
checks in real PID1, the simulator physically DETACHes all selected
DR0..DR3 members. A required fresh `/SYSTEM/INIT` data read on the
now-lost root HALTs at the exact fixed v1515 root PC **003743**
octal. The linked fatal entry is 003740; its BACKSTORE HALT is
003747. The test refuses wrong HALT PC, userland return or absence
of a real root-data request. The matching DSK270 disk-root v1540
physical test was already accepted at the same root HALT PC.

**v1543 PASS**: no production KCORE code, resident RAM, ABI or
on-media change; only versioned testkit configuration and docs.
Sources: `DAIMOS/design/kcore-drm-root-outage-v1543.md` and
`daimos-testkit/tests/validation/daimos-drm-root-outage-v1543.md`.

P0-A remains open for a *failing actual transfer* on unrelated
secondary DSK/DRM mounted filesystems without stopping the selected
root, and for real concurrent or shared DEVICE/VOLUME ROOT/BACKSTORE
failure and owner classification. Merely detaching an unused disk
does not demonstrate secondary mount error handling. KCORE features
only; VERSION 0.5; strict 32K/optimization HOLD.
