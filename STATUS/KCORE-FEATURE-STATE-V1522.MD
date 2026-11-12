# DAIMOS KCORE v1522: mounted DTFS identity revalidation after I/O failure

Date: 2026-11-12. This is a narrowly accepted P0-B source-linked
native PDP-6 feature increment. It does **not** establish
authoritative Type-551 root media-loss detection (P0-A) or a
complete physical multi-process replacement/recovery acceptance.
DAIMOS VERSION remains 0.5; strict 32K stays on HOLD.

## Installed behavior

Native DTFS v1520 keeps a physical tape's random 72-bit `DTVOL1`
ID in two otherwise TSFS-only words of its allocated MM mount
slab. Before v1522, a physical DTC error invalidated shared
clean BCACHE and directory cache, but a mounted volume could
be reused later without comparing that retained volume identity.

v1522 uses two spare bits in the *existing* per-mount DTFS
media word to preserve the needed distinction:

| Bit (RH) | Octal | Semantics |
| --- | --- | --- |
| READ_SUSPECT | `0200000` | Next access requires a fresh physical RBN0 identity proof |
| WRITE_UNCERTAIN | `0400000` | Partial write possible; no implicit recovery on ID match |

The actual `dtfs_dtc_read` error branch marks only still-mounted
DTFS slots on the failed physical unit; `dtfs_dtc_write` does
likewise with the sticky write flag. It does not mark unrelated
devices, create another cache or allocate a persistent table.

Before accepting an old directory cache hit or serving a vnode/
FILE operation, the real `dtfs_load` checks the flag. On a
read-suspect mount, it reads physical block zero through the
original direct DTC service, not clean BCACHE, then validates
the *complete* DTVOL1 stamp with the same native parser used
by initial mount admission. Only an exact match of **both**
saved 36-bit ID words clears the read flag and permits a fresh
directory fetch. Changed or corrupt media, unreadable RBN0,
and zero-ID legacy media fail closed after an error without
retiring the original FILE/vnode or mount ownership.

An uncertain physical WRITE may have damaged directory/data;
that flag is never cleared by a mere same-ID read. DTFS SYNC
and FSINFO space accounting use the checked loader, so failed
or uncertain media cannot accidentally be reported synced
or have stale allocation values displayed. The raw reader can
clobber AC4..AC7: the mount-slot AC6 is now explicitly saved
on the transient kernel stack during I/O and validation. No
new persistent per-process or per-mount memory is required.

## Tests and memory impact

New `daimos-testkit/tests/validation/daimos-dtfs-mounted-id-v1522.sh`
executes the exact production wrappers, BCACHE, identity parser,
loader, FSINFO and SYNC on a native PDP-6, with controlled raw
physical I/O and populated mount slabs. It accepts independent
DTFS mounts sharing one DTC unit, same-volume return after an
error, separate high/low replacement-ID rejection, sticky
write failure, unrelated unit/provider, conservative legacy
refusal and ordinary initial access. **Seven executed unsafe
mutations rejected**, including forced raw AC6 clobber.

Earlier v1490/v1518/v1519/v1520 DTFS and BCACHE native
tests, v1516 root-member classification and DTFS create,
unlink, rename, resize, chain and directory fields PASS.
An isolated 256K KCC tape-root KINIT/KCORE/DTFS+TSFS MRES
build assembles/links. The independent strict single-root
resident-address limit still rejects that combined package;
not a compiler/linker error and no 32K guest success.

Controlled old/new matching KCORE map/other DTFS+TSFS objects
and independent fold plans:

| Metric | Before | v1522 | Delta |
| --- | ---: | ---: | ---: |
| Optional DTFS+TSFS package text (DXR image, octal) | `003620` | `003746` | +86 decimal words |
| Package BSS (octal) | `000002` | `000002` | 0 |
| Fixed KCORE, MM mount slab words | same | same | 0 |

An optional installed MRES grows by 86 instruction words.
No new data words; this is not a claim of zero total RAM cost.

## Remains open

- P0-A: original Type-551 selection/status and Type-136/236
  timeout/PI errors cannot by themselves establish permanent
  boot-root or BACKSTORE loss. v1515 HALT remains uncalled.
  Root/BACKSTORE confirmed loss is crash-only, not reconstructed.
- P0-B: full booted guest with actual open FILE/vnode, dirty
  writes, independent root, repeated real tape replacement and
  VFS mount retirement; safe explicit resolution of a sticky
  WRITE_UNCERTAIN and legacy zero-ID mount.
- P1: mixed DEVICE/VOLUME/LOGSTORE/MEMFS faults, committed
  FSINFO F3 and scheduler-integrated uncompressed swapless PURE.

Full source/test rationale: `DAIMOS/design/kcore-dtfs-mounted-identity-v1522.md`.
This supersedes v1521 only on the specific mounted-DTFS
after-error identity gate. It is not final KCORE acceptance.
