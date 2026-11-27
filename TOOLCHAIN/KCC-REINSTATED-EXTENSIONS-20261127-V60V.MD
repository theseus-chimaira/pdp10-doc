# KCC reinstated extensions and void returns (v60s-v60v)

Date: 2026-11-27. Extends KCC-EXTENSION-SCOPE-20261127-V60R.MD.
Installed host compiler: 2ed00eb. Testkit: c85b1e8.

## v60s: empty initializers

KCC e826ad6, testkit b1f55a2. In GNU/KCC mode, empty braces
initialize bounded arrays, structures, unions and scalars using
the existing zero-fill and scalar conversion paths. Unknown-bound
empty arrays remain rejected; no zero-length type is introduced.
Compound literals and automatic nested initialization are tested.
Original 960221-1.c: 22/22. Acceptance: 128/128 PASS.

## v60t: scalar-to-union casts

KCC dea7e0f, testkit 3992202. Select a compatible member in GNU
mode, evaluate the scalar once, and use existing aggregate and
expression nodes. Runtime casts materialize an automatic union;
direct static union initializers reuse the selected-member list
with constant checking. Verified integer, char, double, pointer
and wide values, static local and file-scope initializers, return,
member access, discard and side effects. General aggregate-member
casts and nested static initializer forms are not established.
Original 960416-1.c: 22/22 compile/codegen PASS.
Acceptance: 154/154 PASS. The initial matrix selection rejected
a semantic status on this codegen source; restored its original
pending status before the successful runs.

## v60u: addressable conditional lvalues

KCC 3fac374, testkit dd7182e. Lower only when an operator requires
an lvalue. Select the original address, including the omitted-middle
case, reusing the v60q saved-value slot for the address. Ordinary
conditional value expressions keep their existing conversions.
Verified assignment, compound assignment, increments, address-of,
structure members, nesting, volatile and single evaluation.
Branches must be compatible same-type addressable objects with
matching pointer representations. Conditional bitfields and actual
register-backed objects are not supported. Stack-backed register
declarations work in the current build; REGISTER_VARIABLES is off.
Strict C99, const objects and bitfields were checked for rejection.
Original 981022-1.c: 22/22 PASS. Acceptance: 136/136 plus 22/22
on the extended stack-backed register guard.

## v60v: value expression in a void return

KCC 2ed00eb, testkit c85b1e8. GNU compatibility emits a warning
and discards the value while preserving side effects. Strict
C99 continues to reject. Verified calls, increments, comma
expressions, constants, void casts and aggregate-returning calls.
Original 920817-1.c: 22/22 PASS. Acceptance: 88/88 PASS.

## Native object footprint

Before/after source copies were cross-compiled with the same
installed host KCC for each patch, using -Pgnu99 -O -x=pdp6
-m=gas, HOST_DAIMOS=1, HOST_UNIX=0 and the self headers.
DAS -F -C assembled each object. Counts are 36-bit words from
DOBJ text, data and BSS headers, not file byte counts.

Patch | Text delta | Data delta | BSS delta
---|---:|---:|---:
v60s ccdecl | +39 | 0 | 0
v60t ccstmt | +163 | 0 | +1
v60t ccdecl | +78 | 0 | 0
v60u ccstmt | +223 | 0 | 0
v60v ccstmt | +16 | 0 | 0
Total | +519 | 0 | +1

No enlarged permanent node, symbol or type representation.
Runtime union casts can increase generated function frame size.
Actual linked native phase size, native self-host execution and
peak memory remain unmeasured.

## Cumulative status

Original 2,196 KCC compile_fail cells: 1,154 explicitly
unsupported, 518 recovered, 524 active remaining. All 518
recovered cells replayed on the installed v60v: 518 PASS,
zero cached cells and zero failures. Not a full matrix sweep.
The cumulative scope manifest is
manifests/PDP10-KCC-UNSUPPORTED-SCOPE-20261127-v60u.tsv.
The three reinstated sources account for 66 cells. GCC remains
85,799 selected cells; KCC 40,820; total 126,619. No GCC or
DAIMOS source changes.

Evidence under TMPDIR:
kcc-empty-init-v60s.wYJz10/guards-v1,
kcc-union-cast-v60t.CPwKHU/guards-v4,
kcc-query-lvalue-v60u.Q1LtY1/guards-v1 and guards-register-v1,
kcc-void-return-v60v.clhQA8/guards-v1,
kcc-cumulative-v60v.BDo48a/replay-v1.
Each patch directory also contains native-before and native-after
DOBJ files. The cumulative parent contains exact cohort manifests.

Final compiler-only install archive:
kcc-void-return-v60v.clhQA8/install-v1/local.tar.gz
SHA256 55b369f98a076d9ae5cb48fad25fb72cc209c57d5d502012c05d9af5712f6197
Installed bin/kcc SHA256:
4c7a6dca0acb2c7f7de4d27c5407a3b8d4c2e0daada2e14ef44512ae65d8977c
