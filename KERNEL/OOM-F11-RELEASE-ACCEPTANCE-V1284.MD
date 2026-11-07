# Requester-only BRK OOM: F11 exact-source release acceptance v1284

## Frozen source and release scope

* DAIMOS source `3978b50ac341e27c92e86162ab2da73093b58cf5`;
  current main `f110be06` differs only in documentation, verified
  no diff under `system/kernel` or `system/boot`.
* Testkit frozen at `69830c9f647f576eb0d60f564ef03ce8fae367c1`.
  The later v1282 provider-quarantine source guard and all its unsafe
  mutants pass independently on the identical production kernel.
* S07/S08: 256K PDP-6, KCC, DSK Stage 1, four-member D6FS disk root,
  PCLK60, headless CTY, normal writable EXEC/RUN. No raw XIP admission.

## Prerequisite evidence

* F07 both policy modes, 63 user slots, 127/136 descriptors, exact real
  APR/BRK and physical OOM under foreign provider contention, 57 child
  RUN/TERM/WAIT: disabled SHA256
  `68c719c5901e3874acaeabe2cef6f309df0566bf99619d89dae2f53380994cd7`;
  enabled SHA256
  `bb0173c726d4e243edec0a09c977b0e4bb68147a4117f90ab932df2e0936b253`.
* E13 exact-source Android and Linux disabled/enabled release-media
  guests, distinct simulator binaries and matching KCORE low endpoints
  `023503` / `023521` octal. Android archive SHA256
  `c87799ba1ecda40f07bbbf718d5ca20e0d5be896549df3be80d075ed0c8990ef`;
  Linux SHA256
  `a59d9b90338a0c23fd084fb7153160bd4068fd4233eabfcf7cd0205fb8b6a720`.
* F10 four full builds zero unexplained compiler/assembler diagnostics,
  v1190 check PASS. v1087 fatal-release transitive no-new-MM/backstore
  graph PASS; v1092/v1282 provider quarantine 16 unsafe mutations PASS.
* F04 real VM/BRK cause -2/-3, v1260 physical arena feasibility/no
  side effects and bad mutation controls PASS. v1123 normal writable
  RUN/EXEC/no XIP and default-OFF six negative mutants PASS.
* F03 all 14 Android full guests exit0 and audit PASS; SHA256
  `c167a89c44ef2480c0c9d91f92e588b15d1a4c9e928a75113c6c2bca1f62e19e`.
  All 14 Linux guests PASS, SHA256
  `0252a5900c7e9850e4c6251e7d52a01898c3b9e1dff5f26e1af5b88d832bb88b`.
* Linux full P0-D matrix was externally interrupted at v1121 simulator
  exit137, NOT reported as an uninterrupted PASS. The first nine
  physical guests had finished; each of v1121 and the other seven
  guests was subsequently rerun independently with exit0 and PASS.
  All native stages PASS. Combined evidence SHA256
  `901326d12c562d7bdb313c05009b370b33219cd45312ffea1defae7c5b47685d`.

## Uninterrupted Android P0-D final acceptance

Android `P0D_FULL=1` completed all 18 named source-linked/native
stages and 17 separately booted physical fault-injection guests
without an interruption. The v1087 transitive release graph creates
no MM/backstore allocation; every physical guest section contains a
PASS and completed before the next stage. An independent log audit
rejected any missing, duplicate or reordered guest-stage boundary.
All 17 guest calls and the complete runner exited successfully.

Archive `$TMPDIR/daimos-f11-android-p0d-v1284-r1.tar.gz`, SHA256
`0565d7e879b432e728ea3262e3cd65774126fda129fe27c6b38ef1233a5819c9`.
The complete simulator output, full driver-stage log, frozen source
revisions, simulator/linker tool identities and SHA256 evidence
manifest are preserved. There is no suppression of error messages.

**F11 is CLOSED on this exact candidate.** The active checklist now
records 62/64 mandatory OOM gates as complete. This is NOT a claim
that production OOM is active. The source switch remains 0 pending
F12; F13 requires actual post-enable negative and positive guests,
and any failure requires reverting the switch.

## Non-OOM limitations

Runtime MRES is boot-installed and not OOM-reclaimable. Unreturning
foreign-provider owner may require controlled restart. Separate
16K KCORE+MRES footprint remains above its independent target.
32K low-memory, other boot media/emulators and raw-XIP RUN/EXEC are
excluded from the first scoped OOM activation, not silently certified.
