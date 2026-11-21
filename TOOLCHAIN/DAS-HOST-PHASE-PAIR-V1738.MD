# Host DAS/DAS2 phase compatibility v1738

Date: 2026-11-21.

Host `das` emits a private DASIR2 intermediate stream and launches
`das2` from the same directory to consume it. The two binaries must
be built from compatible assembler source. A current `das` installed
on 2026-11-21 was paired with an installed `das2` dated 2026-10-16.
Even the standard PDP-6 disk stage-1 bootstrap then failed with
`das2: malformed private DASIR2 phase state` before guest execution.

Both host executables were rebuilt from the same `das.c` revision
using `make -B das das2` and installed together in `$PDP10_PREFIX/bin`.
The old executables remain under the versioned `-pre-v1738` names.
The corrected pair assembles `system/stand/pdp6/dsk/stage1.s` to a
valid DXR image: image `000553`, BSS `001557`, entry `000000`,
relocation `000013` (all octal). The old `das2` combined with the
new `das` still fails, confirming the installation mismatch.

The regression `pdp10-testkit/tests/isa/das-port/
test-das-phase-pair-v1738.sh` exercises the real installed pair and
checks the exact DXR shape. Testkit commit: `ba625b5`.

This is a host tool installation/phase compatibility correction,
not a DAIMOS kernel or filesystem code change.
