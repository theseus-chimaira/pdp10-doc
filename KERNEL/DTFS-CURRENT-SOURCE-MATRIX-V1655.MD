# PDP-6 KCORE v1655: current-source DTFS identity/intent regression

Date: 2026-11-17. Source-linked native PDP-6 V1/V2
recovery, physical DIRTY validation, exact MM mount
intent, and poisoned-owner drain PASS. No production
KCORE/MRES/MM/VFS changes.

Historical source-sensitive v1518 and v1522 testkit
fixtures predate the original v1561 write-intent
owner gate and v1572 operator-trusted writable
V2/CLEAN format policy. They do not presently
extract the source correctly and are NOT newly
accepted as historical standalone current-tree tests.

The more recent v1556 native fixture was modernized:
an unarmed V2/CLEAN file source can be writable before
its original physical first-write DIRTY barrier;
a promotion from RO to the original RW CLEAN state
does not by itself prove a replaced volume. The
source-linked test retains strict physical RBN0,
saved high/low ID, malformed seal, changed owner,
WRITE_UNCERTAIN, and stale BCACHE tag refusal.
An additional native case rejects a genuinely armed
DIRTY record under RO mount policy. Its corresponding
actual invalid-source mutation is executed and fails.

The current versioned testkit suite runs four original
production-source-linked native PDP-6 branches:

| Coverage | Cases | Unsafe mutants rejected |
| --- | ---: | ---: |
| v1556 physical V1/V2 recovery | 30 | 14 |
| v1554 physical DIRTY verification | 17 | 10 |
| v1561 mounted physical-unit intent gate | 35 | 13 |
| v1575 poisoned-owner cache drain | Original leaf | 2 |

All **82 enumerated native cases PASS** and all **39
independent unsafe source mutations are rejected**
by their native test oracle. These are real original
instructions, with controller completions supplied by
explicit test fixtures. No simulated helper predicate
or test shim is added to the production kernel.

Testkit branch:
test-kcore-dtfs-current-matrix-v1655-20261117,
commit a048ab3.
Run: tests/validation/dtfs-modern-v1655/run-v1655.sh.
DAIMOS detailed report:
design/kcore-dtfs-current-source-matrix-v1655.md.

Evidence:
$TMPDIR/daimos-kcore-v1655-evidence-20261117.tar.gz
SHA256 f39b620a6d9462d62e3dcd5d15b9d7ad928afb4ebd291bf18b898ce0e81dc9db.

This matrix does not independently establish the exact
original combined raw DTC read/write-cache + mounted
identity test from v1518/v1522 under the new gate.
That source integration still needs modernization or
equivalent actual evidence before declaring full KCORE
feature freeze. v1575's physical writable removable
DTFS contract requires trusted operator reel retention;
no unavailable hardware interlock is claimed.

Strict 32K boot/size acceptance remains DEFERRED.
Remaining KCORE functional freeze gates: complete peak
RAM, permanently uncertain MM ownership policy, optional
mutable RAM compression measured yes/no, stale source
integration fixture, and final matched-source physical
and native release matrix.
