# Native KCC compiler FP36 phase-link repair v1751

Date: 2026-11-21. KCC `6492d96`, `fb2210d`; DAIMOS `6ac48272`;
pdp10-testkit `fcd75cc`, `00b81dd`.

The current KCC source compiler correctly emits calls to exact
D-format conversion ABI helpers. A fresh native five-phase build
previously stopped at link with KPARSE undefined `__kcc_fixdfsi`;
after fixing that dependency, KGEN exposed undefined
`__kcc_floatsisf`. Both implementations already exist in KCC's
verified compiler runtime. Neither was a reason to weaken the
compiler, replace a conversion with a stub, or alter KCORE.

KCC `native.mk` now builds a separate `fp36-conversions-v1750.a`
from the existing signed and unsigned D-format PDP-6 assembly
sources. The archive is linked after KPARSE/KGEN/KOPT phase objects
and before DAIMOS libc. Archive selection avoids forcing the full
conversion runtime into phases that do not reference it.

The native `daimos.mk` likewise names `BOOT/FP36.DARC` before libc
for KPARSE, KGEN and KOPT. DAIMOS source staging now includes the
exact built compiler archive and uses current source-matched names
for CRT0, bootstrap and syscall helper objects; the D6FS source
manifest includes `BOOT/FP36.DARC`. This ensures the on-target
MAKE link rules receive the same compiler ABI runtime as the
host-built native phase images.

The isolated source-linked regression
`pdp10-testkit/misc/validation/kcc-native-fp36-link-v1750.sh`
PASS: fresh host compiler and all five native phase links, each
validated with `dxrcheck`, then native source-package staging and
byte-for-byte archive comparison. Measured octal image/BSS:

| Native image | Image | BSS |
| --- | ---: | ---: |
| KCC | 012275 | 001745 |
| KCPP | 063324 | 006035 |
| KPARSE | 150717 | 010721 |
| KGEN | 217712 | 005734 |
| KOPT | 042751 | 011454 |

The independent host constant-folding regression passed both
optimized and unoptimized PDP-6 executions. The DAIMOS native KCC
overflow fixture passed a fresh 256K D6FS guest: zero exit status,
no false constant-expression overflow and emitted assembly.

**Not yet accepted:** the standalone native KPARSE KPT4-to-KIR1
physical execution or native MAKE ALL. Attempting the historical
v1401 16-probe test after source-building all phases exposed a
missing legacy `d6codec-fullboot-v1370` LOGIN DXR prerequisite
before guest startup. This is a fixture dependency failure, not a
passed KPARSE test. Rebuild the fixture from current sources and
rerun the physical phase matrix before closing P0 KPARSE.

The separate KIR graph writer's linear typed-pointer identity
lookups remain a potential quadratic native runtime/RAM issue.
The existing host graph test passes 12000 nodes, 9000 symbols and
5000 types and the KIR format roundtrip passes, but those results
do not establish acceptable native KPARSE performance.
