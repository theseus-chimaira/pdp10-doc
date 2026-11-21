# KCORE K04: pre-VM EXIT owner diagnostics v1754

DAIMOS kernel revision v1754 adds opt-in visibility for a process
whose EXIT has relinquished its private executive stack, but whose
pre-VM slot-zero finalizer has not yet released its virtual memory.
This is an operational diagnostic, not an instruction to free memory.
K04 remains OPEN pending physical failure/recovery verification.

## Diagnostic protocol

`SYS_PROCCTL_GETTEARDOWN` is the existing UUO-077 process-control
operation 041 octal. Its original slot query and 0400 detail query
are unchanged. In particular, the legacy slot query can still return
stage 0 during the pre-VM EXIT window.

Bit 01000 octal in the argument opts in to the expanded view:

* `GETTEARDOWN(pid | 01000)` checks for a stable u-area, process
  TRANSITION, still-owned VM, and both FINALIZE and PREVM marks.
* A matching EXIT owner returns `(retry_ticks << 2) | 3 | 01000000`
  (octal). The low stage is 3; bit 18 (`01000000`) distinguishes
  EXIT PREVM from a failed unpublished RUN rollback, which still
  returns stage 3 without that high bit.
* Both 0400 and 01000 request bits may be combined. FILE and post-VM
  MM diagnostics retain their existing formats and semantics.
* `SYS_TEARDOWN_RETRY_TICKS()` masks bit 18 before extracting the
  shared slot-zero retry delay. It is not a per-owner attempt count,
  error code or guaranteed time to successful reclamation.

The userland `PS` command requests this expanded view. Its `TD`
column retains the low two stage bits. `PVM=1` identifies the
pending EXIT pre-VM owner; `PVM=0` with `TD=3` identifies an
unpublished RUN rollback. Its original `DETAIL` column still uses
the legacy read-only request and is zero for a PREVM EXIT owner.

## Ownership and recovery rules

| Observation | Retained owner | Permissible response |
| --- | --- | --- |
| TD 1 (FILE) | Fatal private kernel continuation and remaining FD(s) | Repair refusing FILE/provider; let the original owner retry |
| TD 3, PVM 1 | EXIT transition, VM and stable u-area | Repair actual VM/pin/backing cause; leave finalization to slot zero |
| TD 3, PVM 0 | Unpublished RUN child awaiting VM discard | Keep child unpublished; allow checked rollback retry |
| TD 2 (MM) | Post-VM stable u-area | Inspect 0400 typed MM detail; repair ownership/pin cause before retry |
| TD 0 | No recognized teardown stage | Not proof that an arbitrary external provider lease is free |

A failed release never authorizes manual `mm_free`, unpin, process
descriptor clearing, PID reuse, forced session HUP or destruction of
a sleeping fatal FILE owner's stack. Repair the originating provider,
verify its real completion and let the existing owning continuation
finish. If the state cannot be proved, keep the owner quarantined
and use controlled offline recovery or system restart rather than
a simulator memory DEPOSIT masquerading as successful cleanup.

## Validation and limitations

The v1754 production PDP-6 process assembly and the target KCC `PS`
translation unit compile and assemble successfully in isolated
`$TMPDIR` outputs. This does not constitute a booted guest recovery
test or a new K01/K02 acceptance. No testkit source was edited while
the external sweep was running. Tests for refused pre-VM destruction,
clock-wrap fairness across multiple finalizers, WAIT and descriptor
reuse remain required before K04 can close. Automatic OOM killing
(K03) remains disabled.

Source detail: `DAIMOS/design/kcore-k04-prevm-diagnostics-v1754.md`.

## 2026-11-21 v1755: diagnostic precedence and first retry epoch

Two source-path correctness defects were corrected in DAIMOS
`8b32e91e` without changing ownership, memory allocation or the
legacy query contract:

1. A fatal APR/post-commit EXEC owner retains `PROC_FATAL_CLOSE_BIT`
   after its FILE/session close completes and EXIT publishes
   `FINALIZE|PREVM`. If the subsequent VM destroy refuses, the
   opt-in query must report stage 3 with PVM=1, not a stale FILE
   stage. The PREVM check now precedes the FILE bit for opt-in calls
   only. Unmarked private FILE retries remain stage 1; legacy
   queries retain their previous precedence.
2. A newly refused SIDL rollback formerly initialized its shared
   020-octal-tick retry interval using an earlier owner's stale
   epoch. It now records the current masked `proc_timer_clock`,
   so its first delay starts at the actual refusal. Already-pending
   owners preserve their shared backoff.

Isolated native PDP-6 DAS assembly PASS; physical fault injection
and the ongoing testkit sweep remain separate. K04 stays OPEN.
See `DAIMOS/design/kcore-k04-retry-diagnostic-corrections-v1755.md`.

## 2026-11-21 v1756: invalid stable u-area address

The slot-zero deferred finalizer now rejects a zero u-area base
before reading the per-process FINALIZE marker. A set u-area flag
does not prove a nonzero base. The invalid owner retains the shared
pending state and bounded retry instead of reading low memory or
being silently treated as completed. Valid owners are unchanged.
DAIMOS `6c1b3478` assembled with the native PDP-6 DAS tool; physical
fault injection remains outstanding. K04 remains OPEN.

## 2026-11-21 v1757: pre-VM zero-base guard

The v1756 guard covered the VM_NONE finalizer path, but the
non-NONE VM PREVM path still dereferenced `PROC_FINALIZE_OFFSET`
before reaching that check. DAIMOS `0f7a783f` now rejects a zero
stable-u-area base before the PREVM marker read, preserving the
shared pending owner and bounded retry. This is defensive descriptor
validation, not a claim that a valid execution creates a zero base.
Isolated PDP-6 DAS assembly PASS; physical acceptance outstanding.
