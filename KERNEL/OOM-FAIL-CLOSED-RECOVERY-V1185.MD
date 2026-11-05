# DAIMOS requester OOM: fail-closed noncompletion and operator recovery (v1185)

Status: reviewed operator policy for the bounded writable-process BRK OOM
release scope. This document defines **safe reactions to noncompletion**,
not a promise that a dead device or damaged MM owner can be recovered
automatically. It does not enable production OOM.

## 1. What the OOM policy does and does not own

The only eligible OOM victim is the process whose ordinary writable
BRK request encountered genuine physical MM_ERR_NOMEM after permitted
reclaim/compaction. A BRK failure caused by an address limit,
MM_ERR_FRAGMENTED (-2), MM_ERR_DESCRIPTORS (-3), MM_ERR_INVAL (-4),
MM_ERR_BUSY (-5), or exhausted backing storage must not retire a
process. Failed child RUN admission is an unpublished child rollback,
never grounds to kill its parent. A protected unrelated process,
original in-flight provider owner, and INIT as a *nonrequester* must
not be selected. INIT is not categorically exempt if it itself makes
the failing BRK request.

Boot-loaded MRES belongs exclusively to KINIT: its installed images
are permanently reserved, never dynamically loaded/unloaded, and never
reclaimed by runtime OOM. A failed MRES install is a boot failure,
not a runtime OOM. Low-core permanent address-budget failure must be
repaired before release, not concealed by a runtime module policy.

## 2. Known refusal classes and safe owner lifetimes

| Failure/condition | Required kernel behavior | Operator action if it persists |
| --- | --- | --- |
| Foreign or in-flight D6FS/DTFS/TSFS provider holds a serialized lease and never returns | Retain original provider owner, FILE reference and private kernel stack. Do not unlock from another process or infer that a timed-out caller released its lease. Owner-side retry may back off forever. | Determine whether the original device/provider can resume. Restore media/controller service if safe; otherwise perform a controlled restart. Do not force-unlock or free the owner's memory. |
| FILE close or session HUP returns BUSY; TTY canonical buffer or PIPE/DPY owner remains pinned | Preserve FILE, session and exact owner; sleep/retry only from an owner-private continuation if a provider can block. PI6/slot-zero must not issue sleeping provider callbacks. | Inspect live owner/session and hardware state. Allow the holder to unpin/return. If the holder cannot return, controlled restart; do not fabricate completion or clear FILE counts. |
| VM or u-area MM_ERR_BUSY (-5) during terminal teardown | Keep PREVM/VM_NONE/FINALIZE and u-area owner metadata; qualified clock/idle reaper retries without an unbounded busy loop. Pending retry delay doubles up to octal 01000 ticks. WAIT/slot reclamation remains refused until actual completion. | Restore the pin owner when possible, observe retries and independent process progress. Persistent pin without a reachable owner requires controlled restart; never forcibly unpin. |
| Wrong MM type/base/owner or permanent MM_ERR_INVAL (-4) on final release | Preserve the exact failing owner and allocation. A finalizer may remain quarantined indefinitely; neither a zero pin count nor elapsed time licenses freeing an allocation with mismatched ownership. | Treat as an integrity defect. Preserve diagnostic evidence, stop destructive writes, repair the underlying defect offline and restart. Never substitute another owner or free by guessed address. |
| Swap/backing bitmap corruption, unallocated-block release or I/O refusal | Fail closed and retain bitmap/backing ownership, VM/WAIT state and uncommitted teardown. Do not reuse a block whose exact release was rejected. | Inspect and repair media/bitmap using appropriate offline filesystem/backing tools before controlled restart. Never mark an unverified block free. |
| Late KINIT/boot MRES/physical allocation cannot fit configured core | Halt/refuse boot; do not enter normal user operation with partial MRES, overwritten KCORE or false MM descriptors. | Change boot profile/hardware to a verified supported configuration, fix memory/media or reduce permanent code during development, then cold restart. This is not a runtime OOM event. |
| Fatal owner context absent or an impossible transition is detected | Stop safely rather than continuing with a borrowed stack, guessed owner or forged WAIT/CHLD completion. | Record simulator/console HALT and counters. Correct software/ownership fault, then controlled restart. |

These cases are not equivalent. MM_BUSY can recover if the original
pin owner returns; permanent wrong-owner/MM corruption must not be
retried as if its ownership were valid. An unavailable original
provider can remain quarantined forever even when unrelated processes
continue to run. **There is no general bounded-reclamation guarantee.**

## 3. Operator diagnostics and decision sequence

1. Confirm whether the failure is genuine physical BRK OOM (-1)
   versus the exact VM/MM cause or backing/provider refusal. OOM may
   retire only its requester, never an arbitrarily larger process.
2. Use the existing user-visible `ps` teardown columns `TD`,
   `RETRY` and `DETAIL` (read-only GETTEARDOWN operation 041).
   `TD=1` denotes a fatal FILE/private-stack owner, and `TD=2`
   denotes an irreversibly terminated process awaiting checked
   post-VM MM finalization; ordinary and completed zombie state
   reports zero. RETRY is a shared reaper clock, not a per-PID
   deadline. DETAIL identifies the remaining FILE/provider class
   or the checked u-area pin/type/owner state; it never supplies
   authority to free that object.
3. Confirm progress of unrelated runnable processes and the qualified
   clock; a retained owner must not busy-spin in the scheduler or
   consume unbounded kernel memory while waiting.
4. If the original holder can resume, allow its normal checked return
   and retry, and verify that FILE, VM, MM, backing and WAIT ownership
   is released exactly once. If original ownership cannot be
   re-established, preserve evidence and perform an orderly restart
   rather than forcing completion.
5. When production requester OOM remains disabled, physical BRK
   exhaustion returns the ordinary error instead of selecting a
   victim. Never interpret tests that used a disposable enabled guest
   as proof that the production default was switched.

## 4. Regression evidence and release boundaries

- v1092 provider contract, including 14 unsafe-mutant controls:
  in-flight provider ownership and fail-closed quarantine.
- v1086/v1087: non-sleeping PI6 slot-zero finalization and
  allocation-free transitive fatal-release graph.
- v1101/v1102/v1125: persistent wrong-owner -4, bounded reaper
  backoff, retained unreaped WAIT and independently runnable
  process under a foreign held provider lease.
- v1104-v1107/v1118-v1124: real PIPE/DPY/TTY/VM/u-area MM_BUSY,
  swap bitmap refusal/repair, HUP and checked retry.
- v1157/v1158: APR/postcommit EXEC and failed RUN under foreign
  provider contention, plus a separate OOM requester/independent
  RUN parent.
- v1181: exact-candidate 14-case F03 matrix, positive and negative
  requester OOM including MM -2, MM -3 and PID reuse.

The bounded release candidate is the 256K PDP-6 KCC full DSK270/
D6FS/CTY profile in `OOM-RELEASE-PROFILE-V1160.MD`. Optional
MEMFS raw-XIP RUN/EXEC, strict 32K GCC/DSH and untested media or
simulators are not silently admitted by this policy. Actual boot
residency and peak physical memory, compiler/assembler diagnostics
and final-source regression signoff are independent F05-F11 gates.
No operator action in this document permits changing
`PROC_OOM_BRK_ENABLED_V1000,0` before all gates pass.
