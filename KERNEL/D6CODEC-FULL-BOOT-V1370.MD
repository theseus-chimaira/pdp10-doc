# v1370: actual grouped PDP-6 cold boot through LOGIN, DSH and RUN

Date: 2026-11-08. Acceptance: **isolated native full grouped-format
minimal-system boot PASS**. Production D6LZ36, KCORE and media are
unchanged. This is not yet an approved full-distribution conversion.

## Why this milestone matters

Prior grouped tests reached only KINIT entry or exercised real EXEC,
PURE and TSFS routines against fake VFS/VM providers. This stage
demonstrates an actual grouped kernel running the real D6FS and VFS
providers, loading grouped executable files, entering the shell and
executing a new grouped user process. No further resident-code size
optimization was attempted before functional proof.

The versioned testkit `d6codec-fullboot-v1370` first produces a
full grouped v1367 PDP-6 KCORE, with the previous 40-word low-core
decoder range reserved but no old executable decoder. It reproduces
the entire original KCORE link and runs the v1368 folding and size
acceptance suite. It then replays all **27 current production MRES
package linker recipes** against the old and grouped KCORE maps.
The old-map outputs match byte-for-byte; the grouped-map outputs
retain exactly the same resident sizes. The control KINIT link also
reproduces the original file byte-for-byte.

The grouped KINIT links the 27 grouped-map modules with a correctly
resized embedded KCORE slot. Entry address: octal `054124`;
KCORE load slot: `030037..053240` (exclusive upper bound). The
unmodified `image/mkimage.sh` inserts the real grouped KCORE stream
and the test checks every installed word. The complete 32926-word
image compresses into 29235 grouped boot words. It is loaded through
the real PDP-6 Type-760 paper-tape Stage-0, guarded grouped Stage-1
and Type-136 DECtape simulation.

## Root filesystem and full native boot test

The normal KCC compiler builds actual PDP-6 userland DXRs into an
isolated test build. The grouped DXR2 producer preserves executable
header/entry/TX2/relocation metadata when producing:

| Program | Decoded words | Grouped stream words |
|---|---:|---:|
| LOGIN | 3131 | 2541 |
| DSH | 15331 | 12385 |
| ECHO | 1126 | 918 |

The test root contains 15 D6FS nodes: uncompressed INIT and
MOUNTALL, grouped LOGIN/DSH/ECHO, and the ordinary boot and login
configuration. Four DSK270 disk member images are initialized using
normal `mkdsk`, `mkd6fs` and `logstore`; `d6fsck` verifies CLEAN
superblocks before the boot. This is a **minimal smoke-test root**,
not the full distributed set of commands and files.

The positive native PDP-6 SIMH run with a dedicated clean disk set
completes the following real guest path:

    Stage0 -> grouped Stage1 -> grouped KINIT -> grouped KCORE
      -> 27 grouped-map MRES -> DSK270/D6FS -> INIT V1
      -> grouped LOGIN (ROOT) -> grouped DSH
      -> grouped ECHO V1370GROUPED -> shell prompt
      -> shell exit -> new LOGIN prompt

The test checks that `V1370GROUPED` appears as an actual separate
program-output line, not merely as an echoed command string, and
that INIT respawns LOGIN after DSH exits. Thus a real grouped RUN
is exercised through the installed VFS, process and terminal code.

An independent negative control uses original D6LZ36 KCORE/KINIT
with a separately cloned clean copy of the grouped-executable root.
It reaches INIT but correctly reports `INIT: RUN FAILED` rather
than LOGIN. This verifies the runtime compressed format change is
effective and the old decoder is not accidentally serving grouped
executables.

All control and positive boot runs clone the initial CLEAN media:
boot can generate a newer DIRTY superblock and another boot may
correctly halt with `?RT` if that disk is reused. This behavior
was investigated and fixed in the test harness through independent
disk sets, never by weakening filesystem consistency checks.

The host Termux environment lacks working optional SDL/EGL and some
console backends. The test uses native PDP-6 SIMH headless mode with
the optional display and WCNSLS/OCNSLS interfaces disabled, keeping
all guest filesystem and decoder code unchanged. The selected root
is DSK270, using 256K physical core.

## Reproduction and remaining gates

From `daimos-testkit`:

    DAIMOS_REPO=/path/to/staged/DAIMOS sh \
      tests/validation/d6codec-fullboot-v1370/run-v1370.sh

The test creates only files under `$TMPDIR/d6codec-fullboot-v1370`
and uses the v1368 KCORE link workspace. The normal production
Makefile, D6LZ36 code, linker objects and root media are not edited.
The grouped candidate has the previously measured +256 KCORE text
words with reserved low-core geometry; no static BSS growth.

This proves a bootable, interactive **minimal grouped system**, not
the entire release. The remaining gates include the full normal
distribution and all grouped compressed commands, TSFS root and
restart media, physical PURE/swap eviction and reload, real nested
filesystem/provider stack watermarking, and injected device/media
failures. The single DXR compressed flag does not identify D6LZ36
versus grouped D6CODEC, so the eventual production media/host/runtime
cutover must be atomic. No permanently installed dual-codec mode.

## v1371 subsequent full standard distribution

The full normal D6FS Makefile root (237 files, 29 directories, 89
DXRs, 71 grouped compressed) now boots the same grouped KCORE to
INIT, LOGIN and DSH, and executes and checks real installed grouped
ECHO, LS, CAT, GREP, WC and STAT before LOGIN respawn. The test
root passes d6fsck CLEAN with 336 FCBs. See
`KERNEL/D6CODEC-FULL-DISTRIBUTION-V1371.MD`.
