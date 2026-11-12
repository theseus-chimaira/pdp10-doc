# DAIMOS KCORE v1519: physical native DTFS mount validation

Date: 2026-11-12. Completed optional DTFS KCORE feature fix,
source-linked PDP-6 and full isolated KCC/GCC tape-root builds.
This is format-validation admission, NOT checked replacement-media
recovery, unique volume identity, or hardware media-removal detection.

## What changed

The normal `MOUNT.DTFS` userspace command already invokes transient
`DTFSPROBE`, but `SYS_DTFS_MOUNT` can be called without that helper.
The former kernel `dtfs_mount_unit` checked unit and type/RO flags,
then called `vfs_mount_prevalidated` without verifying that the
physical directory block still contained an accepted native format.
Even the normal userspace probe could be invalidated by a subsequent
tape change. Cached DTC block tags are not physical-media identity.

`dtfs_mount_unit_media_ready` now performs these steps for the
native (currently supported) DTFS personality, **before** any
VFS or MM-backed media owner publication:

1. Invalidate both clean BCACHE identities and clear the shared
   `dtfs_cache_mount` directory identity without MM free.
2. Physically read DTC directory block `0144` using the existing
   MINIT-patched raw `dtfs_dtc_read_jump` into `fs_block_workspace`.
   This deliberately bypasses older `dtfs_dtc_read` cache hits.
3. Refuse failed physical I/O or a directory with invalid native
   magic (`0446446632021`) at word `0177`.
4. Verify reserved allocation owners `036` for physical blocks
   zero and `0144`, and three terminal native owner tags `035`.
5. Only then permit `vfs_mount_prevalidated` and publishing the
   existing MM media slot/owner and root vnode.

The checks match native `DTFSPROBE`, host `mkdtfs` and userland
`MKFS.DTFS` existing format constants. A bad tape returns error
without creating a VFS mount or falsely publishing media identity.
No new runtime buffer, filesystem ABI, on-tape format, process
state or data word has been introduced. Optional TENEX/ITS
personalities remain disabled in the supported production build.

## Native tests, regressions and KCORE budget

Testkit `daimos-dtfs-mount-native-verify-v1519.{s,sh}` source-links
actual PDP-6 `dtfs_mount_unit`, its original register-restoration
epilogue and the production BCACHE invalidation leaf. Test-only
physical DTC and VFS publication callbacks verify I/O ABI,
publication ordering and absence of stale cached writes.

Baseline PASS tests a valid physical native mount, stale cached
directory containing the old magic while new physical media is
invalid, bad block-zero owner, bad directory-block owner, bad
terminal native tags, negative physical I/O, and success after
the same original native medium returns. Every failed admission
leaves the caller's root output, VFS publisher, media patching
and cached clean identities unchanged or invalid as required.

Six **executed** unsafe source mutations were rejected: bypassed
physical read, bypassed BCACHE invalidation, wrong magic check,
skipped block-zero owner, skipped directory owner, and skipped
terminal owner tags. All seven native runs PASS. v1518 DTFS
I/O error/cache, v1490 BCACHE MM refusal/raw DTC ABI, DTFS
chain/create/unlink/rename/directory-field regressions PASS.

Both isolated **full 256K KCC** tape-root and **low-memory 32K GCC**
tape-root KINIT/DTFS-TSFS MRES assemble and LINK successfully.
The strict **32K fixed-KCORE size limit remains FAILED**:
`__kcore_low_end=023241` octal versus required `<020000` octal.
Do not represent a successful image link as 32K size acceptance.

Controlled old/new DTFS/TSFS optional-MRES relink using identical
fixed-KCORE map, other objects and independent folding plans shows
**+27 optional MRES instruction words, +0 permanent data words,
+0 fixed KCORE words**. Existing VFS slot capacity and native
on-media layout are unchanged.

Testkit revision `af2a57a`; DAIMOS revision `ad062984`.
Full detail: `DAIMOS/design/kcore-dtfs-native-mount-validation-v1519.md`
and `daimos-testkit/tests/validation/daimos-dtfs-mount-native-verify-v1519.md`.

## What is still OPEN

The native DTFS directory contains generic magic and allocation
markers, **not a unique volume ID**. Two individually formatted
valid tapes can pass these checks. No existing open vnode, dirty
write, FILE or namespace mount may be silently reattached to a
*different* physical tape based only on a successful native read.
Volume identity, format-version compatibility, checked revalidation,
dirty/open FILE lifetime and actual physical replacement guest
tests remain P1 non-root DTFS feature gates. A silently detached
medium with only cache hits still has no authoritative Type-551
removal indication, so immediate offline detection remains absent.

Confirmed TSFS root-member loss must stop via the separately
accepted v1515 terminal HALT, but real Type-551 loss classification
and its fatal dispatch remain P0 OPEN. Mixed storage providers,
F1 swapless PURE/MM/APR and F3 dynamic FSINFO are still KCORE
feature gates. Compression/size optimization is deferred until
feature completion; VERSION 0.5/32K HOLD unchanged.
