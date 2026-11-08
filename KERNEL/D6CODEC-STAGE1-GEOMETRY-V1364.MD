# v1364: PDP-6 boot Stage-1 physical-memory geometry checks

Date: 2026-11-08. Status: accepted test-only grouped Stage-1
implementation; production boot media and D6LZ36 remain unchanged.

## Configuration and admission rule

DAIMOS already defines `BOOT_CORE_WORDS` and
`KINIT_STACK_RESERVE_WORDS` during PDP-6 image production. Default
physical profiles are 32K (`0100000` octal) and 256K (`01000000`
octal), with `0400` words of KINIT stack reserve. The host image
builder checks these limits, but a Stage-1 reader also needs to
reject corrupt or forged lengths **before writing the payload**.

For a decoded KINIT length U, compressed length C, stack reserve S,
and installed memory size M, octal addresses and word counts must
satisfy both:

    030000 + U + S <= M
    030000 + U + C <= M

The first rule leaves at least the configured KINIT stack headroom;
the second guarantees the complete compressed input can reside
directly after the decoded output. Both are full-word PDP-6 checks;
individual 18-bit header lengths are insufficient. `C=0` supports
the existing raw/TSFS-hybrid boot paths. The old header magic,
entry-offset and media-mode checks remain independent.

The staged native assembler helper at
`system/stand/pdp6/common/stage1-geometry-v1364.inc` accepts
AC1=U, AC2=C, AC3=M and AC4=S, returning AC0=0 on acceptance
and -1 on refusal. Input ACs remain unchanged; there is no static
storage. All five generated Stage-1 alternatives call it before
the first payload copy. PTR also checks Tape 1's first segment
against the declared total compressed length *before* that copy.

## Validation and sizes

The testkit v1364 checks the native helper's exact/one-word-short
physical bounds, zero decoded length and register preservation. It
also executes the actual generated pre-write guard from each of five
loaders under both standard profiles (10 native tests), plus DTC
and PTR with 64K core and nondefault `02000`-word reserve (2 more).
All 12 generated native guards and the standalone helper PASS.

Relative to the previously tested v1363 grouped Stage-1 variants,
the v1364 preflight adds transient text words but **no BSS**:

| Loader | +text, 32K | +text, 256K | Added BSS |
|---|---:|---:|---:|
| DSK | 20 | 21 | 0 |
| DRM | 20 | 21 | 0 |
| DTC | 22 | 23 | 0 |
| MTC | 22 | 23 | 0 |
| PTR | 24 | 25 | 0 |

The test generator takes physical core words and stack reserve as
explicit octal parameters. Production integration must propagate
the build's configured values; it must not hard-code the profiles.

The generated Stage-1 images still fit below the 020000 stack origin
and are test artifacts only. There has been no full physical-device
boot using grouped KINIT, no production codec migration, and no
addition to permanently resident KCORE. Further acceptance requires
device read failures, actual boot/rollback tests, and the atomic
KINIT/VFS/EXEC/PURE/TSFS/host-producer format conversion.

## v1365: subsequent actual device boots

The staged grouped v1364 guarded loader and real linked full KINIT
image now PASS all five PDP-6 Stage-0/Stage-1 virtual-controller
boots to KINIT entry. The Type-136 tests also verify refusal of
an oversized header and malformed grouped control without entering
KINIT or writing 030000. See
`KERNEL/D6CODEC-STAGED-BOOT-ACCEPTANCE-V1365.MD` for these results.
This is not production runtime activation.
