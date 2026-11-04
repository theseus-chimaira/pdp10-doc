# DAIMOS PDP-6 requester-only OOM: activation gate v1123

Status: **NOT READY** (2026-11-04). This is the persistent OOM
acceptance specification, not an implementation handoff. The detailed
action-by-action checklist is the top `v1123` section of DAIMOS
`TODO.MD`. That section assigns unique IDs S01-S08, A01-A04,
B01-B07, C01-C04, D01-D16, E01-E13, F01-F13 and X01-X08.
At this revision, 32 of 65 mandatory items are complete. Of the
remaining 33, 31 block the *pre-activation decision*; F12 (enable
the production switch) and F13 (verify/document/commit that
enablement) are necessarily performed later.

## Exact requested policy

Only an already-running user's BRK request which remains impossible
due to **actual aggregate physical MM exhaustion** after the normal
allowed compaction/reclaim may terminate its own process. Production
`syscall_dispatch_pdp6.s` still defines
`PROC_OOM_BRK_ENABLED_V1000,0`; do not enable it on the strength
of isolated successes. BRK returns its existing user-level -1 and
the VM helper carries the actual internal cause in AC2. MM -2 is
fragmentation, -3 descriptor shortage, -4 invalid argument/type/
owner, and -5 busy/pinned ownership; none may be interpreted as
physical exhaustion. A failed child RUN is an unpublished admission
rollback, **not** grounds to terminate its parent. No cross-process
largest-victim selector is permitted.

## Gate disposition and proof required

| Gate | Status | Required release evidence |
| --- | --- | --- |
| S: Scope/classification | Partially closed | Freeze image/profile; verify exact MM code and opt-in victim selection; keep optional XIP off |
| A: Allocation-free release graph | CLOSED v1087 | Rerun graph and unsafe mutations against final exact code |
| B: Provider ownership | CLOSED for *fail-closed safety* v1092 | Preserve in-flight private stack/lease; do not require false bounded liveness; recheck under physical pressure |
| C: PI6/slot-0 finalizers | CLOSED v1086 | No sleeping acquisition or swap-in under PI6; owner retains potentially sleeping pre-VM work |
| D: Error-stage ownership | OPEN | Complete physical TTY, APR/EXEC, mixed locks/VM/FILE and exact-once multi-resource release |
| E: Full system | OPEN | Sustained reproducible real pressure, spinner fairness, parent WAIT/CHLD, PID recycling and negative scenarios |
| F: Activation | OPEN | Full matrix, actual resident RAM budget, signoff, only then production enable and repeat |
| X: Optional raw XIP process launch | EXCLUDED | Not wired into ordinary RUN/EXEC today; adding it requires separate opt-in acceptance |

### A/B/C: what closed actually means

`daimos-oom-transitive-release-v1087.py` checks the real bounded
allocation-free fatal graph across FILE/VFS SYNC, HUP/TTY, VM/XIP,
PIPE/DPY, backing bitmap, u-area and notification. It does **not**
guarantee total elapsed teardown time. The provider lifetime rules
accepted in v1092 retain a live in-flight DTFS/D6FS/TSFS lease and
its original owner; a returning error retains FILE/VM identity for
the next checked owner-side retry. An owner that never returns may
remain quarantined indefinitely; administrative repair or controlled
restart, not forced unlock, is the accepted response. Such a system
is safe against the known use-after-free paths but cannot promise
reclaim or progress from that specific dead provider.

PI6-masked slot-zero cleanup is non-sleeping (v1086), while HUP and
provider calls that can sleep remain on their owner-private stack.
PREVM/POSTVM markers retain the process identity, MM owner and
exit/WAIT status until the checked release truly commits. Never
replace those stages with a PI6-masked blocking callback.

### D: already demonstrated resource combinations

- v1101: physical wrong-owner -4 failure quarantine and bystander.
- v1104/v1105/v1106/v1107: independent physical PIPE, DPY, process VM
  and DISK bitmap release refusal/repair.
- v1108/v1109: source-linked MEMFS MRES XIP unpin and VM destroy;
  **not** a scheduled XIP process.
- v1118: real PIPE+VM MM_BUSY, FS lease, deferred u-area owner and
  independently progressing process with stale WAIT checks.
- v1119: PIPE+DPY on the same delivered session HUP; live PI6 replay.
- v1121: physical DPY MM_BUSY while one process genuinely swaps out
  and then restores; checked bitmap detach was **not** executed.
- v1122: first-member physical PIPE MM_BUSY and separate member's
  actual DISK bitmap corruption/refusal/restore during one session
  teardown, including exact WAIT/TTY and no leak. Standalone v1104
  and v1107 were rebuilt and rerun after harness composition.

Remaining D release cases must use actual production MM/FILE/TTY/
provider error returns on valid owned objects. Synthetic error codes
or test-only fake process descriptors do not demonstrate a physical
fault. Require exact owner, type, base, pin/reference, FILE/VM/
swap backing, scheduler and process-slot counters before refusal,
after refusal, and after successful finalization. A wrong-owner -4
must *remain* quarantined; do not force a success to make the guest
HALT. Include a physical TTY buffer fault, nested APR/EXEC fatal
entry, RUN rollback, and long-held provider contention in a booted
cross-product. All D items in DAIMOS TODO must be checked before the
normal-process-only activation decision.

### E: integrated stress acceptance

The existing 256K real-OOM v1075 guest proves requester-only fatal
cleanup against an unrelated blocked witness. The v1120 guest
further completes 24 sequential RUN/WAIT lifetimes with 23 adjacent
numeric PID reuses; v1118 checks 48 stale WAIT/PID requests and 37
spinner cycles during its held-owner interval. These are valuable
short-form proofs, not sustained liveness under repeated OOM.

Before declaring E complete, decide and document a *measurable*
stress horizon, run multiple true exhaustion/recovery cycles within
that horizon, and retain counter snapshots after every transition.
At least one independent runnable process must advance through
provider contention and slot-zero retry while qualified timer
interrupts continue. Combine a failed unpublished RUN with a
separately OOM-killed BRK requester and APR/EXEC fatal flow.
Maintain correct CHLD/WAIT, foreground TTY and PID/slot lifetime,
without free-after-use, unbounded runnable kernel loops, leaking
process words, backing sectors, MM descriptors or unexpectedly
killing another process. A reused numeric PID can validly address
a *new* process: test stale requests **between** lifetimes and
exact event attribution, not fictitious generation-number semantics.

Distinct negative cases are mandatory: invalid/out-of-range BRK,
address ceiling, fragmentation with enough total words but no
aligned fit, exhausted descriptor table, pinned MM_BUSY, storage
exhaustion without physical shortage, and failed child RUN. The
requester may only die for real MM_ERR_NOMEM. Rebuild both default-
off and isolated enabled variants on the same candidate revision.

### F: image, physical RAM and enablement

First freeze the supported boot profiles and root device/provider
set. The most recent unchanged KCC full-image KCORE low endpoint
reported after the v1100-v1122 tests is `023171` octal. This is
not a measured runtime peak, and the historical v513 full/auto
`permanent-size` package sum exceeded the `040000` octal image
budget. **Do not infer that all packaged MRES are resident at once**;
measure the actual live set, KCORE text/data, MM extents, provider
buffers, stacks, APR/alignment waste, and remaining 16K executive
address space. The strict `<020000` octal KCORE goal and full 32K
DSH command-boot optimization are already explicitly deferred;
do not silently use a 256K guest to claim either of them.

After a clean final source/negative native matrix and complete
`P0D_FULL=1` reboot matrix, rerun all v1075/v1110-v1116/v1120
default-disabled and disposable enabled physical-OOM variants.
Compare complete source and RAM/size reports with the last disabled
production baseline. Manual acceptance must explicitly record
provider quarantine limitations, supported media/profiles, testing
platforms and any excluded optional features. Only then change the
production switch from 0 to 1, rebuild, rerun the entire matrix,
and revert it on an unexplained regression. Commit and push only
the verified result with updated testkit STATUS, DAIMOS TODO/
PROJECT-STATE/KCORE and this documentation.

### X: optional MEMFS XIP must not expand this release accidentally

The private MEMFS MRES op `027` offers XIP admission; op `026`
provides release. Production `proc_run.s` calls `vm_space_startup`
once and `exec_lowmem_pdp6.s` calls it twice. Its `movem 3,(11)`
writes the user's startup record even when argc/envc are zero.
No ordinary RUN/EXEC loader calls the raw-XIP admission routine.
The v1123 source guard and six negative mutations check this
**current scope**, not the safety of a scheduled XIP process.

Therefore raw-XIP process creation remains OFF and may be excluded
from *current normal-process* OOM activation. Once root-only XIP
admission is introduced, this exemption ends: complete the separate
X03-X08 gate first (trusted non-writable image, strictly no-user-
write startup, swap/BRK guards, physical pins, provider lifetime,
scheduled RUN/EXEC/EXIT/WAIT, failure rollback, PDP-6 simulator
coverage) and re-run the applicable D/E/F OOM tests. Ordinary PURE
DXR files do not automatically satisfy that distinct raw-XIP gate.

## Fast review procedure

1. From `daimos-testkit`, run `make test-daimos-oom-scope-v1123` to
   check the default-off and optional-XIP source boundary. A PASS
   confirms only that limited contract; six mutations must fail.
2. Run the source-linked component matrix
   `make test-daimos-p0d-release-matrix-v1096` on current source.
   Before production enable, run with `P0D_FULL=1` to require all
   disposable guest modes and record their complete logs.
3. Complete every applicable unchecked S/B/C/D/E/F item in DAIMOS
   `TODO.MD` **through F11**, plus any relevant new code-path
   regressions. F12 and F13 occur only after signoff. Do not infer
   green readiness from a partial matrix or from this document.
4. Reconfirm `PROC_OOM_BRK_ENABLED_V1000,0` until the documented
   final signoff. The switch remains 0 while any *pre-activation*
   gate is open. Optional raw-XIP is explicitly excluded until separately
   admitted and qualified.
