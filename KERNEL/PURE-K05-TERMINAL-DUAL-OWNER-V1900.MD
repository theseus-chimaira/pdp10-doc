# PDP-6 K05 v1900: exact two-owner RAW PURE terminal cleanup

**Historical policy:** v1920 supersedes direct dual-owner terminal
release. The current contract requires deferred ownership reconciliation
before terminal destruction; see PURE-K06-TERMINAL-IDENTITY-V1920.MD.
The results below describe the original v1900 source, not current
production acceptance.

Date: 2026-11-23. Scoped **original PDP-6 native terminal/MM/
FINALIZE+PREVM integration PASS**. The production source correction
is entirely in the opt-in removable PURE terminal MRES. K05-K09
remain OPEN; RAW PURE and mode3 OFF by default.

## Defect discovered in original terminal code

An interrupted or failed original FULL restore can leave TWO
authoritative physical MM descriptors for one process slot:
the PACKED RAW mutable suffix and a complete temporary PROCESS
APR allocated for the unsuccessful restore. The original v1609
RAW terminal leaf accepts a single PACKED descriptor only and
thus refused such teardown forever even after all external pins
were released. The actual original PDP-6 native VM/terminal
guest reproduced the exact valid two-owner failure before the
correction at octal assertion `000000001001`.

The original source-free refusal v1895 and target-free refusal
v1896 physical experiments separately proved these retained
two-owner states can arise from actual allocator outcomes, not
merely testkit constructions. The normal finalizer must not
discard the authoritative PACKED owner, bypass native pin
validation, publish RESIDENT or reuse the process slot while
either extent remains uncertain.

## Correction, no new persistent metadata

Only `system/kernel/proc/proc_pure_packed_detach_v1609.s`
changes. Existing terminal preflight continues to check exact
PACKED storage mode, TRANSITION, stable u-area, saved executable
backing identity, absent live VM APR and original 02000-octal
APR alignment. Its native MM owner scan now executes with the
global PI gate disabled through BOTH possible free calls and
checked result publication. It accepts ONE exact unpinned
PACKED RAW suffix, plus optionally ONE complete, aligned,
unpinned PROCESS target of original logical APR length.

Original MM count must be positive, within permanent descriptor
capacity 136 decimal and within the actual runtime
`mm_extent_capacity`. Duplicate, extra, malformed, truncated,
overpinned or reserved process-MM ownership is refused before
any release. MODULE and KERNEL_DYNAMIC keep their independent
owner namespaces. No new process field, registry or resident
MM descriptor has been introduced.

The provisional PROCESS target is retired FIRST with the real
native `mm_free(PROCESS)`. An actual refusal keeps the last
PACKED RAW source intact. If target deletion committed despite
an error reply, the existing PREVM retry later finds the sole
authoritative PACKED owner and safely continues. Next, the
native `mm_free(PACKED_PROCESS)` releases the source. If this
last deletion committed but reported a false error, a second
PI-excluded scan must prove the original source base is absent
and no unexpected slot-owned PROCESS or PACKED extent survives
before publishing RESIDENT. A true refusal or uncertain owner
retains PACKED mode and original backing for finalizer retry.

The existing idle-stack `FINALIZE+PREVM` retries and shared
exponential backoff remain unchanged. There is no sleeping
operation, new resident kernel state or unconditional MM scrub.

## Measured memory effects

| Region | Original v1899 | Corrected v1900 | Difference |
|---|---:|---:|---:|
| Permanent fixed KCORE | unchanged | unchanged | 0 words |
| PURE terminal text | 90 words | 156 words | +66 |
| Installed opt-in PURE MRES | 2071 words | 2137 words | +66 |
| New permanent BSS/process/MM fields | 0 | 0 | 0 |

Unchanged fixed-KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Original optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.
Rebuilt optional PURE MRES DXR SHA256:
`09bcb71e4cbb172222f6389153c27293353fdaf6d3210eeef4630ed06de934f2`.
The original reclaim JRST confirms physical installed PURE base
`032417` octal on the updated real 48K PDP-6.

## Actual tests and limits

The first native fixture source-links the ACTUAL PDP-6 terminal,
actual VM-destroy instructions, compiled KCORE swap detach and
storage-mode machine code. A testkit allocator fixture drives
exact two-owner successful release, repeated source/target
pin refusal and recovery, malformed/duplicate/foreign owner
cases, genuine native-free refusal and retry, and falsely lost
success replies. FOUR separately executed unsafe terminal ASM
mutants are rejected at exact octal assertion results `001103`,
`001203`, `001305` and `001714`.

A second stronger native fixture links the ORIGINAL PDP-6
allocator `mm_pin`, `mm_unpin`, `mm_free`, `mm_delete` and
`mm_find_base`, real VM teardown, original process finalizer
and `proc_finish_service_v881` idle-stack PREVM retry/backoff,
with BOTH slot-owned extents. The first true source pin refuses
teardown; PREVM remains marked after original idle retry.
Native unpin releases that source; the original allocator
deletes both extents. The real finalizer then completes VM_NONE,
u-area release and zombie publication, without repeated VM
release or double-free. The historical v1609, v1680, v1681
terminal native suites and v1855 FULL native tests also pass.

The full-system 48K test links the updated MRES with unchanged
fixed KCORE and preserves the original five-child ROOT/secondary
DTC3 compressed guest, including independent user parent HBs.
Its real physical original `mm_free(PACKED)` -5/persistent pin
path **PASSes** original PURE FULL fail-closed compatibility:
MM count18 with authoritative RAW PACKED `111200..120000` pin1
and temporary full PROCESS `120000..130000` pin0 remain through
actual native source free refusal, original FULL service and
TWO new parent HBs. Slot3 retains TRANSITION/VMbase0, MM and
selected TSFS ROOT remain unchanged and actual PCLK advances
`021574` -> `021674` octal between parent user HBs.

The installed PURE MRES grows by 66 words without changing fixed
KCORE, so the original dynamic `proc_table` moves from `044712`
to **`045014`** octal. Physical runtime pointer and actual slot3
meta `045025=046133600401`/VM `045026=010000000000` are checked
at all snapshots, not presumed to remain at old lowcore addresses.
Independent physical acceptance and TEN executed unsafe physical
owner/pin/process/PCLK/user transcript controls PASS. All seven
earlier v1893-v1899 physical cases also re-PASS independent audits
and their 74 executed unsafe transcript negatives.

Original normalized v1900 physical PDP-6 trace SHA256:
`91d9cacd28c8f6de77640340f30a431e6e74f23959eabc411fcdcc033aa45a55`.

One test-only late relink regenerated the base TSFS tapes and
temporarily omitted compressed `/SYSTEM/COMP` (real guest `!3`).
Restoring the exact compressed test root tapes after linking
resolved that test setup error with both kernel binaries
unchanged. The accepted physical preflight checks the actual
root tapes and the REAL updated process table pointer.

This work does **NOT** yet prove real
physical foreign SIGTERM or EXIT on the still-quarantined
two-owner child followed by a real PI6-scheduled idle finalizer
and eventual pin release. That remains a separate K05 gate.

Executable versioned testkit:
`tests/validation/pure-k05-terminal-dual-owner-v1900/`.
DAIMOS design:
`design/kcore-k05-terminal-dual-owner-v1900.md`.
The accepted 48K original physical evidence is in the versioned
`$TMPDIR/daimos-k05-terminal-physical-v1900-real-v3/` run.

K05-K09 remain OPEN. RAW PURE/mode3 stay disabled by default.

**v1901 physical safety follow-up (2026-11-23):** A real user
`SYS_PROCCTL_EVENT_PID`/`SYS_EVENT_TERM` sent AFTER original
dual-owner source `mm_free(PACKED)` refusal is correctly refused:
slot3 still holds TRANSITION, VMbase0, and original saved
executive `CTX_PC=340000016664` with the USER-return bit CLEAR.
Forcing foreign finalization would free its live private stack.
The next step must reconcile the provisional PROCESS owner
from a safe slot-0 context after pins release, preserving the
executive continuation, not bypass the event guard. NINETEEN
executed unsafe physical controls PASS. See
`KERNEL/PURE-K05-TRANSITION-TERM-V1901.MD`.
