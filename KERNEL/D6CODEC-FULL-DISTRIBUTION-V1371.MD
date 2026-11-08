# v1371: full normal DAIMOS D6FS distribution boots with grouped D6CODEC

Date: 2026-11-08. Status: **native PDP-6 full-distribution boot and
representative grouped command execution PASS**. This is test-only;
the production OS still uses D6LZ36.

The v1370 result proved a grouped KCORE, all 27 linked MRES packages,
the real Stage-0/Stage-1 KINIT boot and an interactive LOGIN/DSH/ECHO
run on a small 15-node D6FS test root. The new v1371 test uses the
**full normal D6FS_ROOT_ARGS manifest** from the current DAIMOS
boot Makefile, not a separately transcribed list of applications.

The current normal root has **29 directories, 237 files, 89 DXR
executable files and 148 binword files**. The 71 executable paths
that normally refer to `.d6lz.dxr` are all rebuilt using the
independently validated grouped DXR2 producer from the corresponding
uncompressed linked binaries. Each is decoded independently to
match its original 36-bit words; entry address, TX2 text metadata
and relocations are preserved. The other 18 DXRs (including INIT,
MOUNTALL and native toolchain bootstraps) remain uncompressed.
No D6LZ36 executable payload is allowed to remain in the imported
DXR list. All sources must exist and all filesystem paths must be
unique. A versioned inventory records a SHA-256 checksum for each
imported file.

The test separately stages the real native DAS executables,
KCC-supplied boot binaries and source/ABI tree, manuals, ASMUTILS
sources and configuration through normal Makefile dependencies.
Its auxiliary Makefile is parsed after the original, while all
outputs remain isolated below `$TMPDIR/d6codec-fulldist-v1371`.
The usual `mkdsk`, `mkd6fs`, `logstore` and `d6fsck` host utilities
construct the full four-member DSK270 root. It checks **CLEAN,
336 FCBs**. The 71 grouped executable artifacts occupy 904704
host bytes including their executable metadata.

## Native full-distribution acceptance

The v1371 runner repeats the v1370 full grouped KCORE/MRES cold-boot
test and independent old-kernel negative control before building
the complete root. It boots that root on native PDP-6 SIMH through
the real Type-760 Stage-0, Type-136 guarded grouped Stage-1, KINIT,
the grouped KCORE and all 27 matching MRES packages, and actual
DSK270/D6FS, VFS, INIT, LOGIN and grouped DSH.

Interactive ROOT authentication succeeds. The following normal
installed grouped commands execute, with guest output verified:

| Command | Acceptance evidence |
|---|---|
| ECHO V1371FULL | Standalone `V1371FULL` output line |
| LS /SYSTEM/EXEC | LOGIN, DSH, GREP, WC and MAKE file entries |
| CAT /CONFIG/FSTAB | Correct DAIMOS FSTAB text header |
| GREP MEMFS /CONFIG/FSTAB | `MEMFS:/TEMP:4096` line |
| WC /CONFIG/FSTAB | `3 7 55` result |
| STAT /SYSTEM/EXEC/DSH | Regular executable, mode 0555 |
| LS /OPTION/BASE/EXEC | DAS, DARC, DLINK, OBJDUMP, KCC entries |
| EXIT | INIT respawns LOGIN successfully |

The checks distinguish guest program output from terminal-echoed
input. The host keeps a pristine CLEAN root image and uses a
separate copy for each native boot, so interrupted/failed boot
transactions cannot contaminate subsequent filesystem checks.
All native v1371 command, login and respawn tests **PASS**.

## Reproduction and remaining work

Run from daimos-testkit:

    DAIMOS_REPO=/path/to/staged/DAIMOS sh \
      tests/validation/d6codec-fulldist-v1371/run-v1371.sh

The grouped KCORE remains the v1367 reserved-low-core variant,
previously measured as **+256 linked text words** versus D6LZ36;
no RAM/code-size optimization occurs in v1371. The production
runtime remains unchanged; the single compressed flag in DXR2
does not encode the codec and cannot support mixed formats.

Important remaining blockers: the uncompressed installed `D6LZ`
utility still creates legacy-format DXR streams, so it needs a
grouped-format replacement or removal in any atomic production
conversion. The native KCC/DAS binaries were listed but not
execution-tested by this regression. Real PURE eviction/reload,
swap, TSFS root/restart media, nested VFS/provider stack watermarks,
and injected device/media fault recovery require separate native
end-to-end tests before enabling a permanent grouped-format build.
