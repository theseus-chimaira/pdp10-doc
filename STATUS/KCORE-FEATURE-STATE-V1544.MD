# KCORE v1544: native D6FS secondary DSK270/DRM236 isolation

Date: 2026-11-13. Feature acceptance only; no optimization.

Actual PDP-6 native code now links complete source from D6FS
file read/write-through, `fs_backing.s` physical DSK270/DRM236
direct adapter and interleave-set mapper, the selected-root
provider/mount classifier, and the existing v1515 fatal leaf.
The final physical driver outcome is test-controlled; the shared
cache is a small test double. The current full KCC tape-root
fixed KCORE and D6FS MRES production links pass.

All **24** native cases pass: DSK and DRM root/secondary read/write
success/failure, secondary three-member interleaved read/write
error, and logical out-of-range rejection for both root and
secondary. The test checks actual Type-270/236 selector, physical
unit and block, I/O counts, BCACHE handling, and root-only
logical D6SET accounting. Six actual source mutations are rejected
(lost root gate, false owner match, wrong controller, false root
accounting, wrong HALT reason and wrong interleave member).

Historical v1513 secondary media return, v1515 distinct critical
HALT and direct-backend v1 native regressions all PASS.
Only stale positive-permanent-media-loss comments in production
`fs_backing.s` and `proc_storage_runtime_v30.s` were updated.
**Zero executable words, RAM words, ABI or media format changes.**

The native cases are not an actual mounted-secondary D6FS
full-guest hardware-detachment test. The runtime mount syscall
expects a previously validated 14-word transient handoff; no
production userland handoff scanner currently exists. A full
physical negative test must mount a real validated secondary
D6FS, force a failed device request and prove independent root
progress without a fatal HALT. Combined ROOT/BACKSTORE
DEVICE/VOLUME fault acceptance is also pending.

Sources: `DAIMOS/design/kcore-secondary-d6fs-direct-isolation-v1544.md`
and daimos-testkit `daimos-secondary-d6fs-direct-error-v1544.md`.
VERSION 0.5; strict 32K and optimization gate HOLD.
