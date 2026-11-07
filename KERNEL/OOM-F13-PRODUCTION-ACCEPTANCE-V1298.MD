# DAIMOS requester-only BRK OOM F12/F13 production acceptance v1298

Date: 2026-11-07. Scope: 256K PDP-6, KCC, DSK Stage 1, four-member
D6FS root, 60 Hz PCLK, headless CTY, ordinary writable RUN/EXEC.
The post-enable production policy is ON, restricted to actual BRK
physical memory exhaustion. All 64/64 mandatory OOM release gates
are accepted for this profile, completing F12 and F13.

## Source provenance and functional scope

The exact independent-host post-activation release source is candidate
`78259106e267636130a4ab41f01d020981fe8c41`, parent
`e6993a764d9fc82f2421c8b71862c1aa80e2fa14` (accepted F11).
Its entire production source change is the one-word assembly policy
declaration `PROC_OOM_BRK_ENABLED_V1000` from `0` to `1` in
`system/kernel/proc/syscall_dispatch_pdp6.s`. The candidate source
from the original v1285 Git bundle was verified byte-for-byte against
the v1298 working source. F11's previously accepted 62 checks remain
valid for the same production implementation.

This switch does not extend OOM selection to MM descriptor exhaustion,
fragmentation, invalid BRK requests, failed RUN, backing exhaustion,
or nonrequester processes. Foreign filesystem-provider ownership,
allocation-free fatal release and fail-closed quarantine semantics
remain in place. Unreturning provider operations can still require
controlled restart; this release does not promise an automatic
termination bound for every failure.

## Functional and endurance evidence

* Exact-source F03: all 14 booted disabled/enabled guest combinations
  pass, covering v1075, v1110, v1112, v1114, v1115, v1116 and v1120.
* P0-D: all 18 native release/ownership stages and all 17 individual
  booted physical owner/fault cases pass. An initial monolithic run
  was interrupted after six physical guests; the remaining eleven
  passed in a separate sequential continuation. This is complete
  individual coverage, not a claim of one uninterrupted full run.
* F07: both fresh full 64-slot policy guests pass with 63 real users,
  127/136 MM descriptors, eight swapped images, 239576 allocated MM
  words, 31642 APR-rounding words, 180/206 private-stack peak and
  63/256 idle-stack peak. Only the requester is selected when enabled.
* Source/native tests: exact BRK cause/AC2, actual MM -2/-3, policy
  mutation guards, restricted raw-XIP exclusion, provider retry and
  independent scheduler progress all pass.

## Independent post-activation host/media acceptance

The original v1285 archives were recovered and independently checked
in the v1298 release review, not rerun over the unavailable LAN.
Android aarch64 and Linux x86_64 both ran the SAME v1285 source and
testkit `74baec8b9be739d20ba55780491410556b8448f6` with
production_oom=ON. Each host records both guest policies, four
complete KCC/DSK/D6FS release-media builds in total. Every guest
has a valid internal SHA256 evidence manifest, reports a successful
256K physical-OOM provider-held boot and correct requester/witness
policy, and retains its complete SIMH/build/media diagnostic logs.

The corresponding per-policy KCORE maps are byte-identical between
Android and Linux; their PDP-6 simulator binary hashes are different.
Strict v1190 F10 diagnostic audit passes all four builds with zero
unexplained compiler, assembler or media diagnostics (12 approved
classes on Android, 13 on Linux owing to the approved legacy D6FSR2
layout note). The independent-host archive SHA256 values are:

* Android: `29df5d087439b23643652aa3bb8cb8bd9bb78161aafe5bd281569e54ece746fc`.
* Linux: `bf77f0009a6598dcb172a57595214d8c4825c49f15f09b2ceacc7a7750c6ddcc`.

## Release evidence and exclusions

The combined source bundle and verified Android/Linux/local evidence
are retained in `$TMPDIR/daimos-f13-release-evidence-v1298-r1.tar.gz`,
SHA256 `5dcb4d75ea6ea2291e5ec01f54874ce8d704713c1608fff9dcd64ab8aab83221`.
The local v1296 evidence archive SHA256 is
`13411b8dbbf548a425567a6ed2a460e57cf3e8d2155a9ad32e4bbf84ba2a6e07`.

F12 and F13 are CLOSED only for the defined first production release
profile. This does not certify restricted raw MEMFS XIP admission,
full 32K DSH boot, other boot media or hardware, runtime MRES
unloading, or the separate 16K permanent-resident/KCORE size targets.
DTFS/TSFS implementation tests remain excluded pending the rewrite.
The next KCORE functional freeze work is D6CODEC streaming integration,
shared BSTORE consumer migration, restricted MEMFS XIP RUN/EXEC, and
executable-opt-in compressed process swap. Optimization is deferred.
