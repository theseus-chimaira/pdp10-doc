# Alpine x86-64 host toolchain installation, 2026-11-25, v1

The DAIMOS host installation was rebuilt on Alpine Linux with host GCC
15.2.0. The shared prefix is `/home/theseus/cross`. Login and interactive
shell initialization must export PDP10_PREFIX and put its bin directory
before system directories: an older DAS in /usr/local/bin otherwise
shadows the current assembler.

Build order: daimos-tools (provides sixmd-check), DAS, KCC with runtime
and headers, PDP-10 GCC with its source-tree build.sh, then DAIMOS host
utilities and libc. The GCC helper is run out of tree. For the DAIMOS
libc installation under GNU make, explicitly select the target compiler,
assembler and archiver (CC, DAS, DARC). ImageMagick 7 development headers
are required by the DAIMOS C img2dpic implementation. The separate
daimos-tools d6lz utility is retained when installing both tool sets.

The installed source revisions were:

| Repository | Revision |
| --- | --- |
| DAIMOS | 417a05d9 |
| daimos-tools | 48fc3df |
| DAS | eca12aa |
| KCC | edff620 |
| PDP-10 GCC | f2a25976 |
| SIMH | fe97912 |
| KLH10 | 7f740f8 |
| KX10 | a723bf0 |
| TS10 | 671909a |
| PDP6 | 0b9128c |

KLH10's three configurations (kn10-kl, kn10-ks, kn10-ks-its) coexist in
the prefix after the cumulative-install fix. KX10's host configure and
single-owner global definitions were repaired. TS10's PDP-10, PDP-11,
and VAX components build together with the original GNU inline semantics;
its standalone dump10 has an independent non-debug tape object. The
original PDP6 emulator uses an explicit GNU C99 dialect. It is installed
as pdp6-aap, with pdp6_s, pdp6_n and pdp6_headless_v1 alongside SIMH pdp6.
The separate DAIMOS pdp6-aap-v2 executable is not supplied by this build.

Validation: installed KCC and GCC compiled a C function, DAS assembled
both outputs, and DARC archived a target object. SIMH PDP6, KA10, KI10,
KL10 and KS10 register sanity checks passed. TS10 clean rebuild and
interactive startup/quit passed. KLH10 cumulative installation passed.
The daimos-testkit pure-service-v1612 native PDP-6 test passed, including
the unsafe READY negative control. No full guest boot matrix is claimed.
No KCORE implementation or acceptance state changed.
