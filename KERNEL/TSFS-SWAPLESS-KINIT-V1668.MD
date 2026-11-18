# v1668: dedicated TSFS KINIT excludes the disk-swap MRES candidate

Date: 2026-11-18. Exact DAIMOS parent v1667:
`9d9d310526a363caf43430e18a84b5720cc08872`.

## Memory architecture and selector contract

DAIMOS v1665 chooses its actual root before installing a single
memory-management MRES, and v1666 puts the terminal disk/RAW
owner logic in that selected MRES rather than in fixed KCORE.
General ROOT=auto requires both possible candidates in transient
KINIT storage until the real root and validated AUXST1 BACKSTORE
are known. A machine configured strictly as TSFS and without
swap, however, never needs the disk-swap memory-management
package, so carrying it wastes bootstrap image words.

v1668 adds explicit build-only `DAIMOS_SWAPLESS_ONLY_V1668=1`
for `ROOT=tape DAIMOS_PURE_MRES_ENABLED=1`. Incompatible
ROOT=auto, disk, drum or disabled PURE builds abort Make with
diagnostics. The flag defaults OFF, preserving all normal
root/disk and auto-root behavior. The original v1667 bitmap PURE
stage remains an independent optional switch (default OFF),
not a requirement to use swapless-only KINIT.

Strict KINIT's module table gives `proc_swap_minit_v481` a zero
MRES package pointer, instead of referencing
`proc_swap_mres_package_v481`. The top-level linker does not
receive `swapv481-mres.dobj` in either GCC lowmem or KCC full
profiles. The existing `proc_swap_reclaim`,
`proc_swap_service_one`, `proc_swap_term_v1666` and default
backstore gateway remain in KCORE unchanged. Only the existing
PURE MRES is installed permanently. DSK/DRM hardware services
are kept for independent LOGSTORE and other auxiliary uses;
removing DISK SWAP does not remove the physical controllers.

At boot, actual root selection MUST report DTC/TSFS. The
existing read-only AUXST1 discovery validates the complete
descriptor and the BACKSTORE size. Absent/invalid/zero usable
BACKSTORE causes the unchanged PURE MRES install and its three
KINIT-patched service exports. A positively validated nonzero
BACKSTORE HALTs with SWAP FAIL BEFORE any PURE installation or
disk swap publication. A wrong/unresolved root also fails
closed. In the full KINIT profile, AUXSTORE is rediscovered
later for LOGSTORE setup; a positive BACKSTORE detected during
this second probe causes the same fatal refusal. A real
LOGSTORE-only descriptor without BACKSTORE remains supported.

No disk-swap ABI stubs are made successful artificially; the
KINIT image simply omits the package it cannot legitimately
use. No permanent dual-implementation compatibility branch,
MM owner registry or new per-process data is introduced.

## Exact source-matched GCC lowmem measurements

| Linked measure | General ROOT=tape, PURE | Strict ROOT=tape, PURE |
| --- | ---: | ---: |
| Fixed `__kcore_low_end`, octal | 024076 | 024076 |
| Installed PURE MRES | 1863 words | 1863 words |
| DISK SWAP MRES candidate | 1837 words | not linked |
| KINIT image end, octal | 113732 | 110205 |

The dedicated image saves **1877 transient boot-image words**:
1837 from the unnecessary disk MRES package and 40 net words
from other KINIT code/table/packaging changes (not separately
attributed). This is verified from two
independently linked builds of the same production source using
the testkit's `run-size-v1668.sh`, not from a compiler guess.
Fixed KCORE and installed PURE memory are exactly unchanged;
the 1877 words are NOT claimed as net steady-state free core
or a measured physical transient high-water improvement.

The independent KCC FULL source-matched tape-root link pair
passes the same ownership/size validation: fixed KCORE remains
octal `024353`, installed PURE remains **2307** words, general
KINIT end `131475` -> strict `125775` octal, **1856 transient
words saved**. Both strict GCC and KCC maps entirely omit the
DISK SWAP MRES package symbol; ordinary profiles retain it.

## Source-linked, link-map and PDP-6 physical evidence

The v1668 testkit extracts the real `proc_swap_minit_v481` and
later `auxstore_minit` from `module_minit.c` into strict host
C89 tests. It verifies exact optional PURE installation,
three terminal/scheduler patches, all non-DTC and positive
BACKSTORE fatal refusals, and no disk MRES installation. It
executes and rejects two unsafe source mutations: bypassing
the first fatal and bypassing full-profile rediscovery fatal.
Top-level Makefile candidate selection is verified for strict
tape and ordinary tape/auto/disk/drum; invalid flag combinations
are refused, and v1665/v1613 root-selector regressions pass.

The real source-built 64K GCC lowmem PDP-6 with three TSFS
DECtape root members and no attached auxiliary disk/drum boots
to `MEM 64 K`, `DTC OK`, `PURE LOADED`, `INIT V1`. With the
**same** kernel image and the genuine generated DRM0/AUXST1
positive BACKSTORE descriptor attached, the console instead
prints `SWAP FAIL` and the CPU HALTs (observed octal PC 065774)
before PURE or INIT. This is an actual media-discovery refusal,
not a stubbed positive BACKSTORE test.

## Remaining work

Continuous whole-machine 48K/64K physical KINIT/BCACHE/MM
peak RAM accounting, identical process workloads comparing
PURE-off/RAW/bitmap-on, physical general ROOT=auto and DSK/DRM
regressions, and strict physical 32K acceptance are still open.
The v1667 bitmap stage does not link its compression frontend
to live reclaim and fixed VM teardown refuses mode3. No real
compressed-process RUN/EXIT/WAIT/PI6 lifecycle or net RAM
benefit is claimed. Preserve the normal disk/drum swap backend
and keep PURE/bitmap mode3 OFF by default.
