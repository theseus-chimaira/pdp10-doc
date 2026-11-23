# Recovered manual and D6CODEC map checks, v1820-v1821

Both changes are testkit-only and preserve current DAIMOS production
code. See the testkit's versioned test readmes for executable details.

## Native SIXMD inventory, v1820

The historical v1073 manual test asserted old packaging and formatter
paths which no longer exist. The updated audit derives the public
program names and standalone DXR targets from the current
`userland/Makefile`, checks that `PUBLIC_MANUAL_NAMES` is identical to
the public executable inventory, then validates each SIXMD page with
the installed `sixmd-check` tool. It additionally verifies private
MOUNTALL and DSHCOMP manuals and 26 optional assembler utility pages.

Observed PASS: 77 public manuals, 2 private helper manuals and
26 installed optional assembler utility manuals. Native MKSWAP is
documented as planned, but is not installed. The old host MKSWAP
binary and host-only manual are absent from current production;
`mkauxstore` and `d6swap` have distinct interfaces, so the old
host v1071 formatter test was archived rather than misrepresented
as current coverage. External compiler-phase packaging is not
within the new audit's scope.

Command: `make test-native-manual-coverage-v1820` in daimos-testkit.
Testkit commit: `e4242ba`. The former v1071-v1073 detached files
are archived in
`~/backups/daimos-mkswap-manual-v1071-v1073-v1820-20261123/`, with
SHA256 verification and a retained tracked-file patch. That
historical worktree is clean.

## D6CODEC fixed KCORE map, v1821

The historical v1459 linker-map diagnostic enforced the physical
placement of the D6CODEC fixed core. This assertion complements
the already committed v1459 byte-exact KCORE/Stage-1 comparison and
its real-image cold-boot test. The recovered `lowcore-map-v1821.py`
loads the source-matched linked `kcore.map` and checks:

- Decoder core begins at octal 000060.
- The fixed image is exactly 53 PDP-6 36-bit words.
- VFS and direct-buffer adapters lie beyond the fixed image.
- Retired v1452 decoder symbols are not present.
- KCORE's permanent low boundary contains the image.

The current KCORE map passed: decoder start 000060, VFS 001110,
direct-buffer 001156, resident low boundary 024330. All five
in-memory forged symbol-map controls were rejected.

Command: `make test-d6codec-lowcore-map-v1821` in daimos-testkit,
after a complete DAIMOS source-matched KCORE build. Testkit commit:
`1cbce75`. Four detached old v1459 diagnostic scripts are saved in
`~/backups/daimos-d6codec-v1459-detached-v1821-20261123/` with
verified SHA256 and retired from the detached worktree.

**Not claimed:** fresh DTC/MTC boot success, physical media rewrite,
new decoder optimization, or complete kernel memory acceptance.
