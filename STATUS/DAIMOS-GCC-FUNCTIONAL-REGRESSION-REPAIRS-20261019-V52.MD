# DAIMOS GCC 256K Functional Regression Repairs (2026-10-19, V52)

## Scope

The earlier 30-group SIMH disk sweep excluded 32K and size acceptance.
This record covers **focused** confirmation of its failed groups, not a new
complete acceptance run. The corrected GCC remains isolated in the test
environment and does not overwrite the installed cross-compiler.

## Confirmed restored checks

- Group **004**, boot/codegen: five boot origins (DSK, DRM, DTC, MTC, PTR),
  four-member disk-root discovery, Stage1 KINIT entry, SIXBIT lifetime,
  and PDP-6 stack-adjustment codegen all pass.  Exit 0, 23:22:59 CEST;
  `$TMPDIR/daimos-gcc-v47/group004-v47.log`.
- Group **031**, DSH conformance: 115/115 pass, including HELP ECHO and
  HELP DAS, with the corrected GCC local-char-array codegen. Exit 0;
  `$TMPDIR/daimos-gcc-v46/dsh-conformance-v46.log`. See companion toolchain
  documentation `GCC-PDP6-LOCAL-CHAR-ARRAY-20261019-V45.MD`.
- Group **023**, storage integration: the three native D6FS test oracles
  now define KINIT's four-slot `vfs_mount_limit`; those tests pass. The
  MonitorFS online-device test had omitted detachment of DTC3 while trying
  to disable the DECtape controller. SIMH rejected `set dtc disabled` and
  the guest correctly found a DECtape device. Detaching the fourth unit
  fixes the test; direct GCC retest passes, exit 0 at 23:28:03 CEST.
  The combined group has not yet been rerun in one invocation.
- Group **032**, PDP-10 host tools: the old `d6fsck-repair-v2` test claimed
  PASS before its power-loss ordering subtest. The checker now writes only
  three changed blocks (DIRTY publication, changed summary, CLEAN
  publication). Cuts 1 and 2 correctly leave DIRTY metadata for recovery;
  cut 3 occurs after CLEAN publication, and no fourth durable write exists.
  The obsolete fixture expected four interrupts and three DIRTY states.
  Testkit `11feecb` corrects the expectations and checks readback, recovery,
  and unchanged media after ambiguous corruption. Full group-032 sequence
  passes all 12 host-tool scripts, exit 0 at 23:39:20 CEST;
  `$TMPDIR/daimos-tools-v52/group032-v52.log`.

## Outstanding separate failures

- GCC group **017** multiprocess scheduler remains failed with the corrected
  compiler: `<aAbB2cCxxxxxxxxdDeEyyyyyyyy314D5`. Investigate the actual
  hardware-clock quantum before adjusting workload or assertions. The
  boot diagnostic sees `HZ 60 LINE` but does not prove that four clock
  interrupts occurred before the first process checkpoint.
- Group **027** has old TSFS mount fixtures that boot a D6FS root and then
  demand a resident secondary DTFS/TSFS service. Current `dtfs_minit()`
  installs that shared package only for a DECtape-selected root, by design.
  Avoid adding permanent MRES memory merely to make an unsupported mixed
  profile appear supported. Tape persistence and alternative tests remain
  to be designed/retested.
- Automatic root priority (006), MonitorFS live state (009), and other
  failed groups still require targeted investigation; do not infer that
  the full 30-group GCC/KCC suite now passes.

All verified fixes above are testkit or compiler corrections; DAIMOS kernel
production source remains unchanged.  The simulator runs used 256K with the
diagnostic DSH image ceiling `037000`, not strict 32K acceptance.
