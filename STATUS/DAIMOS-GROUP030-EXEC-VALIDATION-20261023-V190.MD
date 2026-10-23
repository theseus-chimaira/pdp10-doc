# DAIMOS GROUP 030: EXEC/RUN NEGATIVE-LAUNCH VALIDATION (V190)

## 2026-10-23 VM growth allocation exhaustion with RUN pressure (V198)

The new daimos-testkit target `test-daimos-run-vm-oom` extends the
existing v197 native RUN fixture on the standard 256K PDP-6. Three
children remain alive while the caller requests a well-formed 18-bit
`BRK(0776000)` four times. A near-256K resident user VM cannot fit
beside the kernel and other active process allocations. All four
requests fail as expected in the VM/MM allocation path. Each returns
without changing the original break, the caller's process extent
length or the sentinel data; process slot counts remain stable.
Reported process-resident words do not increase after failure (reclaim
may swap out an eligible sleeping child, reducing that counter).

The test then completes v197's 16 late RUN descriptor-map failure
checks, waits for the three children to exit, and verifies recovery
with an additional valid RUN/WAIT. The fresh-image native simulator
test **PASSed**. Evidence:
`$TMPDIR/daimos-run-vm-oom-v198-native.log`. The original v197 target
still cross-compiles with the v198 checks disabled.

This establishes a real unallocatable-VM growth and rollback case; it
does not yet demonstrate direct ENOMEM inside `exec_load_process` or
`vm_space_create` during RUN/EXEC. It does not individually enumerate
dynamic-MM descriptors. No DAIMOS resident changes were made, and the
linked KCORE exclusive end remains `025657` octal. The 32K gate is
explicitly deferred, and strict full-profile size and timing limits
have not been changed.

## 2026-10-23 RUN late-failure rollback stress (V197)

`daimos-testkit` v197 adds the native PDP-6 fixture
`test-daimos-run-late-rollback`. It keeps three children alive while
executing 16 invalid RUN requests whose fourth explicit descriptor
mapping names a nonexistent parent FD 15. Earlier validation and
executable/VM/startup/u-area setup succeed; the invalid mapping is
detected in the late RUN file-descriptor publication loop. All sixteen
RUN calls return `-1`. After each failure, the fixture confirms that
`SYS_MEMINFO` process-slot and process-word totals still equal their
pre-failure baseline. The waiting children are then unblocked and
reaped, accounting returns to the initial baseline, and an ordinary
RUN/WAIT succeeds afterward.

Native 256K PDP-6 fresh-image test: **PASS**. Evidence:
`$TMPDIR/daimos-run-late-rollback-v197-native-b.log`. The first
prototype failed an invalid test assumption that initial slot usage
must be exactly one; the corrected fixture uses the observed nonzero
baseline and passes. No production kernel modifications, new resident
memory or stack changes resulted. `__kcore_low_end=025657` octal.

The test demonstrates RUN late descriptor-map rollback and recovery,
not deliberate full MM exhaustion, a complete audit of dynamic-MM
allocation leakage or post-commit EXEC failure. The 32K-specific
image/boot/DSH errors remain **deferred by project direction**, not
accepted. Full-profile permanent-memory acceptance remains separate.

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
