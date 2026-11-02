# DAIMOS PDP-6 kernel and standalone regression group 028 (v123)

Date: 2026-10-21

## Diagnosed test mismatch

The previous kernel smoke Make target used `tools/watch-output.sh` to
require the production DAIMON CTY banner followed by `LOGIN:` on
that same CTY stream within three seconds. The current DAIMOS kernel
reports its boot and device-initialization diagnostics on CTY, but
interactive LOGIN and DSH use the DCS0 serial transport. In the
v75 non-size matrix the old smoke test timed out under both KCC and
GCC and caused group 028 to fail, even though the separate stage1
bootloader suite completed.

## Correction

The versioned `daimos-testkit/tests/system/kernel/daimos-kernel-boot-dcs-v121.sh`
smoke test now builds the standard production PDP-6 disk image with an
isolated DCS0/GE0 port pair. It checks CTY's DAIMON version banner and
the `BSET LOADED` initialization report. It then uses the established
testkit PTY/TCP harness to require DCS0 ROOT login, a DSH prompt, and
the outputs of external ECHO and PWD programs. Both command probes
require return to a fresh DSH prompt. No host-side synthetic success
markers or new kernel behavior were introduced.

The production build is used with functional 256K memory and the
explicit diagnostic `DSH_IMAGE_LIMIT=037000`; strict 32K boot and
resident-size acceptance remain separate and deferred.

## Results

| Regression | KCC | GCC |
| --- | --- | --- |
| Production PDP-6 kernel boot banner and BSET | PASS | PASS |
| DCS0 ROOT login and DSH | PASS | PASS |
| External ECHO | PASS | PASS |
| External PWD | PASS | PASS |

The independent standalone PDP-6 loader suite was also rebuilt and
rerun with the corrected SIMH, passing **26/26** DSK, PTR, MTC and DTC
test cases.

Logs: `$TMPDIR/daimos-kernel-group028-kcc-v121.log`,
`$TMPDIR/daimos-kernel-group028-gcc-v122.log` and
`$TMPDIR/daimos-stand-group028-v122.txt`.

Group 028 is accepted independently for both compilers. Group 027
(DECtape/TSFS) is explicitly deferred; groups 029 and 030 and the
complete serialized non-size matrix are not yet accepted.
