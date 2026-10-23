# DAIMOS GROUP 030: EXEC/RUN NEGATIVE-LAUNCH VALIDATION (V190)

## 2026-10-23 RUN lifecycle and scope-ID retest (V196)

The existing daimos-testkit `test-daimos-process-lifecycle` and
`test-daimos-process-id-lifetime` targets both PASS on fresh native
256K PDP-6 images. Verified RUN/GETPID/WAIT, WAIT/nohang, orphan
adoption, zombie reap, explicit descriptor mapping, and preservation
of process-group/session/domain identifiers across creator-slot reuse.
Logs: `$TMPDIR/daimos-run-lifecycle-v196-native.log` and
`$TMPDIR/daimos-run-id-lifetime-v196-native.log`.

The strict 32K boot/KCORE size gate and DSH issues specific to that
profile are **deferred by explicit project direction**, not fixed or
accepted. Neither test changes executable size, permanent RAM, or
KCORE code; the current full-profile KCORE exclusive endpoint remains
`025657` octal. The independent full-profile permanent memory deficit
is recorded in `STATUS/RESIDENT-SIZE.MD` and remains unresolved.

Date: 2026-10-23

The native PDP-6 replacement fixture now performs 28 malformed inline
EXEC V1 / RUN V2 calls before its already accepted normal EXEC
replacement. The additional cases are maintained in daimos-testkit
`tests/validation/daimos-exec-malformed-v190.h`, included from the
existing `daimos-exec-replace-v1-init.c` fixture.

The 14 EXEC and 14 RUN cases cover invalid header versions and declared
word counts; 36-bit scalar counts containing forbidden upper-half bits;
argc, envc and fd-map limits; counted-SIXBIT path and argument record
lengths, including empty/oversize records; and truncated or mismatched
block endpoints. Each malformed call must return `-1`. The fixture
restores each mutated field after rejection and verifies that the
original process survives to perform normal current-image EXEC.
The RUN negative cases cannot legitimately spawn a child.

Testkit commit `9bc72d6`: GCC C99 target cross-compilation PASS and
fresh-image native PDP-6 simulation PASS. Native log:
`$TMPDIR/daimos-exec-malformed-v190-native.log`. The runner uses
the explicit functional-only `DAIMOS_EXEC_OBSERVE_SECONDS=35` and
`DSH_IMAGE_LIMIT=037000`; both production defaults remain unchanged.

These results do **not** establish strict 15-second EXEC timing, the
`036000` DSH image-size gate, real-time-owner admission/transfer/rollback,
or full group-030 acceptance. The resident KCORE image was not changed;
the existing v188 post-p10fold exclusive endpoint is `025657` octal.
Current full permanent-memory measurements and their failing 16K gate
are recorded separately in `STATUS/RESIDENT-SIZE.MD`.
