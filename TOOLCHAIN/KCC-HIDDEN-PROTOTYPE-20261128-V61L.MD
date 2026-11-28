# KCC hidden old-style parameter lists (v61l)

KCC 15e9b87, testkit 042e3be6. In parglist, the hidden prototype
recorded for an old-style function definition is diagnostic information,
not an explicit C prototype. Excess arguments incorrectly caused a hard
error even though the other hidden-list count checks issued advisories.
Use the existing warnf distinction for that remaining branch. Explicit
prototypes retain their argument constraints. Default promotions and
native ABI argument placement are unchanged; no new runtime mechanism
or persistent metadata is required. Incorrect argument counts in calls
without prototypes remain undefined behavior when executed. The
historical source is compile-only, not a defined runtime oracle.

## Verification

Evidence: TMPDIR/kcc-hidden-prototype-v61l.dtd75O.
- Original torture-unsorted/pass.c: 22 compile failures recovered.
- Native argument ABI regression kcc-oldstyle-argument-abi-v61l:
  22/22 pass on both revisions. Covers valid direct and indirect calls,
  default promotions, 36-bit values and multiword stack arguments.
- Combined focused suite: baseline 22/44, candidate 44/44 pass.
- Registered diagnostic checks: 320/320 pass across C89/C99/GNU89/
  GNU99, four CPUs and opt/noopt. Incorrect calls are not executed.
  Real prototype violations still fail, including a prototype before
  or after an old-style definition, and function pointer calls.
- Broader 444-cell argument suite: 366 -> 388 pass; remaining 54
  compile failures and 2 link failures identical by exact keys.
- Fresh historical replay: 848/1042 pass, 128 compile_fail,
  66 run_fail. Exactly 22 cells recovered; no new failure keys.
  Manifests PDP10-KCC-COMPILE-REMAINING-20261128-v61l.tsv and
  PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61l.tsv.

The broader replay exposes an integration defect already present in
v61k: support/kcc/insns.h defines __builtin_va_list and old variadic
macros despite KCC v61j providing native builtins. The duplicate
typedef causes 54 compile failures (argument-offset, function-pointer,
varargs). The stale compatibility shim was reconciled with native builtins
in v61m; see KCC-NATIVE-VA-INTEGRATION-20261128-V61M.MD. The other two failures are unresolved
vprintf linkage in va-arg-21.c. None was excluded or reclassified.

## Native footprint and installation

Same baseline cross-compiler builds both ccstmt revisions for PDP-6
-Pgnu99 -O -m=gas -DHOST_DAIMOS=1 -DHOST_UNIX=0; DAS DOBJ headers:
text 9360 -> 9377 (+17 words), data 2 -> 2, BSS 23 -> 23.
No linked phase, native self-host, peak dynamic RAM or speed measurement.
No GCC, assembler or DAIMOS source changes.

Compiler-only install-v61l/local.tar.gz installed into PDP10_PREFIX
and verified identical to the tested candidate; previous binary at
install-v61l/previous-kcc-v61k.

- bin/kcc SHA256: f1f01ac47de106f831d9c5037890bc980b5dac899ad18958b171f0f4b9c29924
- local.tar.gz SHA256: 0b1603a2086c353c97c80fcec5fc946e97a1ab94c0dc7534ab94c7d7fc5e1e0f
