# E13 v1276: exact-source dual-host physical OOM release profile

## Accepted first OOM profile

The frozen S07/S08 first release is 256K PDP-6 Open SIMH V4.1-0,
KCC-built full kernel, DSK Stage 1 with four-member D6FS/DISK root,
60 Hz real-time PCLK, CTY headless and configured DSK/drum AUXSTORE.
Alternate boot/root media, other simulator families, OpenBSD and the
strict 32K/DSH profile are outside this frozen first scope, not
implicitly tested or enabled.

The same exact DAIMOS production source `226e7a5f` and testkit
`ee9d92b` were independently built on Android aarch64 and Linux
x86_64. Each host executed both normal requester-OOM-disabled and
disposable requester-OOM-enabled release-media full guests. The
normal production switch remained OFF in every build. Both policies
and both hosts PASS real physical BRK exhaustion, original FILE/FS
lease ownership, protected INIT/witness/gateway/spinner, and exact
WAIT, u-area, VM and MM cleanup. Only requester PID 3 is selected
for fatal retirement when test-enabled; disabled mode returns -1.

## Independent binary, build and source evidence

| Property | Android aarch64 | Linux x86_64 |
| --- | --- | --- |
| DAIMOS | `226e7a5f` | `226e7a5f` |
| Testkit | `ee9d92b` | `ee9d92b` |
| Simulator SHA256 | `601b6e9ec9979a7b457ab1d866072f4e167bd14929b6de26877c6620fcc36ea0` | `fdf2efcad69bc4fa7c14fa97fa0031923123af184832af6ca986302432b3d72c` |
| OOM-disabled KCORE low end (octal) | `023503` | `023503` |
| Disposable OOM-enabled low end (octal) | `023521` | `023521` |
| Disabled and enabled guests | PASS/PASS | PASS/PASS |

The v1163 independent-host verifier checks the different simulator
binaries, exact profile and source provenance, actual OOM/protected
process counters, loaded D6FS, boot media and identical linked
endpoints. Complete original build logs, SIMH console, memory/map,
host-tool, profile, media and counter evidence are retained.

Android artifact:
`$TMPDIR/daimos-e13-release-media-v1160.dYyc6O.tar.gz`

SHA256:
`c87799ba1ecda40f07bbbf718d5ca20e0d5be896549df3be80d075ed0c8990ef`

Linux artifact copied and rehashed on Android:
`$TMPDIR/daimos-e13-linux-x86_64-v1272-r1.tar.gz`

SHA256:
`a59d9b90338a0c23fd084fb7153160bd4068fd4233eabfcf7cd0205fb8b6a720`

## Exact release build diagnostics

The v1190 F10 audit rejects unexpected KCC notes, compiler/assembler
warnings and tool errors. All four complete build logs pass with 12
identical approved tool-note classes and zero unexplained messages.
Linux-built legacy D6FSR2 media emits the already-approved optional
reserved-range note. Android's modern explicit-zero media correctly
omits it. Testkit v1275 excludes only this approved optional *media*
warning from cross-host diagnostic equality; all compiler/assembler
notes remain strict. Three unexpected diagnostic mutants still fail.

## Scope and activation boundary

E13 is **CLOSED only for the stated first S07/S08 release profile**.
This does not claim successful alternate emulator, OpenBSD host,
TSFS/tape boot, raw XIP process launch, full 32K KCORE/DSH or
Type-340/interactive sessions. Those are separate release items.
The independently passed F07 64-slot/descriptor test is documented in
`OOM-F07-64SLOT-ACCEPTANCE-V1274.MD`.

F11 final exact-candidate acceptance remains OPEN. In particular an
extra current-source Linux v1112 failed-RUN regression halted before
its expected marker, whereas historical DAIMOS and Android current
source passed. This host-dependent fault is being bisected; F11 must
not be signed off until it has been diagnosed and retested. F12/F13
production-enable and rollback testing have not started. Requester
OOM remains disabled in production. The scope-adjusted OOM checklist
counts 61/64 mandatory gates complete.
