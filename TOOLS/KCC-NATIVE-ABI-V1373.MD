# v1373: PDP-6 native KCC register arguments across nested calls

Date: 2026-11-08. Status: **isolated native nested-call ABI and
single-function KCC code-generation PASS**; complete self-hosting
is still blocked by separate KIR/global and unoptimized-KGEN issues.

The v1372 native KCC compiler emitted invalid assembler output
for a valid C89 `return 012345` fixture. Instrumentation in KGEN
showed incorrect `P_CVALUE` and `P_DFMP` instructions entering
`NEWCODE`; KIR1 retained the correct integer constant and KOPT
faithfully printed the invalid output. KPCODE serialization was
not responsible for the corruption.

The cross-build compiler itself was miscompiling nested function
arguments. `code1(op, vr, s)` calls `codr1(op, vrtoreal(vr), s)`;
the original generated code evaluated `vrtoreal` first, leaving
its return value in call-volatile AC1, and passed that return value
as `op`. It also failed to preserve the third incoming argument.

KCC `ccgen2.c:gccabi_direct_reg_args()` now snapshots incoming
register-bound arguments in tracked virtual registers **before**
evaluating any side-effecting argument expression. Ordinary call
spilling/reload preserves those snapshots; simple calls without
side effects retain the old direct zero-copy fast path. This is
a general GCC-ABI compiler fix, not a KGEN-specific workaround.

The testkit `kcc-native-refresh-v1373` compiles a corrected host
KCC from the staged source, checks preservation of AC1 in the
generated `code1()` assembler, and rebuilds all five native
KCC/KCPP/KPARSE/KGEN/KOPT DXRs using that host. A pristine native
PDP-6 boot uses the full grouped-format KCORE, all 27 matching MRES
packages, and the normal 237-file D6FS root with only those five
compiler binaries replaced. Nine PDP-6 checks PASS, including:

- Native KCC's packed 36-bit assembler exactly matches host KCC for
  a minimal C89 function returning 012345 octal.
- Native KCPP/KPARSE/KGEN(KPCODE2)/KOPT run independently and the
  resulting assembler stream also matches the host reference.
- A separately host-compiled nested-call executable tests all three
  incoming register parameters under PDP-6 and returns success.

The regression is reproducible in the `daimos-testkit` checkout:

    DAIMOS_REPO=/path/to/staged/DAIMOS \
    KCC_REPO=/path/to/staged/kcc \
      sh tests/validation/kcc-native-refresh-v1373/run-v1373.sh

Remaining separate compiler issues: native KGEN reconstructs an
invalid global symbol class (`gaddress: bad Sclass 0`) when native
KCC compiles a larger multi-function/global-source fixture, and
standalone KGEN `-N` trips a UUO and exits 126. The host-built
multi-function executable runs correctly, but native *compilation*
of it is not yet accepted. Neither issue is hidden by the nine
passing checks. Native KCC -> ordinary S6REC assembler -> DAS/DLINK
-> executable RUN also remains an independent self-hosting gate.

No resident kernel, DAS, production binary or D6LZ36 media changed.
