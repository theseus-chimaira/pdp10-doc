# P1-E v1611: opt-in PDP-6 PURE MRES

2026-11-16. Verdict: native transaction and link verified;
physical guest and net-RAM acceptance OPEN. Default OFF.

DAIMOS now includes a distinct PDP-6 source-selected
swapless MRES with the existing process reclaim and
slot-0 wakeup exports. It uses v1381 zero-copy prefix
omission for SLEEP/STOP processes, v1384 bounded
same-APR reconstruction, v1391 bounded FULL fallback,
and original MM/source identity/transition gates.

For dedicated TSFS roots, KINIT can select this PURE
package only when no AUXST1 backing exists; otherwise
the original disk SWAP module is installed. This
experimental selection requires
`DAIMOS_PURE_MRES_ENABLED=1`; the default remains 0.
It does not change resident KCORE or any process
descriptor, MM descriptor, backing format or machine
backend interface.

Source-linked native PDP-6 tests of actual SLEEP DROP,
RUN slot-0 prefix/FULL wake, intact mutable suffix,
invalid source/BACKSTORE and failed restore PASS.
The default KINIT does not link PURE. Experimental
KCC KINIT and PURE MRES build and link, but no full
physical no-AUX guest has been accepted. The
fixed `__kcore_low_end` remains `024534` octal.

PURE MRES installed text: **2275 KCC / 1864 GCC
words**, no BSS or extra process state. KCC full
aggregate single-root 16K report: default 4814
words over limit, PURE included 7089 words over.
The report counts both mutually exclusive storage
modules and is not a substitute for actual physical
installed-RAM accounting. At a modest 256-word
omitted prefix, nine inactive owners would only
break even with the KCC MRES cost before peak
restore allocations. No profitable workload
is accepted yet.

The Makefile KINIT version-stamp default-order bug
was also corrected; historical v490 MINIT GCC
test passes. Full real PI6 scheduler, source
immutability/unmount, EXEC/EXIT/WAIT/BRK and
MM/physical fault matrices remain open.
No compression or new CPU support is enabled.

See DAIMOS
`design/kcore-p1e-swapless-service-v1611.md`,
`PROJECT-STATE.MD`, and `TODO.MD`.
