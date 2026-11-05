## v1148: C04 integrated private-owner HUP and real PI6 closure

Two independently rebuilt 256K PDP-6 v1148 guests PASS the combined
C04 scenario. The original session leader's first EXIT broadcasts a
remote HUP while the attached canonical TTY MM extent is physically
pinned and returns genuine MM_BUSY. One child records exactly one HUP
deferral on its private executive continuation and exactly one HUP
delivery at its user-return boundary; parent WAIT checks the actual
HUP exit status. The original session is retained until the retry.
The second leader EXIT then encounters actual pinned process VM and
u-area MM_BUSY and completes through the normal slot-0 retry clock.
Each guest observes one pin/refusal/unpin for TTY, VM and u-area, one
final commit and no hook errors. PI6 advances 32 actual ticks after
the refusal phases, with 48 sleeps of an unrelated session process.
The existing v1098 clock/backoff source contract passes all three
negative mutations. No fake timer tick or allocator return is used.

C04 is CLOSED for the current writable-process OOM scope. Mandatory
checks: **39/65 complete, 26 open** (24 pre-enable plus F12/F13).
B06/B07, E05-E13, F02-F11 and real RAM/diagnostics signoff remain
open. Production requester OOM remains disabled; no kernel, MRES,
persistent RAM or ABI change was made.

## v1146r3: D16 exact-once matrix closed on frozen candidate

The nine-stage `daimos-testkit` target
`test-daimos-oom-exact-once-matrix-v1146` PASSes completely against
DAIMOS `2e12c1da` and testkit `253a813`, using the fixed installed
P10FOLD, DLINK, DAS and PDP-6 simulator checksums recorded alongside
the retained test logs. Its cases cover the transitive fatal-release
graph and mutants, physical PIPE/DPY/VM/u-area pin/BUSY/unpin/free,
canonical TTY, FILE/HUP refusal and foreign provider-lock retention,
swap-out/in and bitmap-corruption/refusal recovery, the nested
APR/EXEC/unpublished-RUN rollback, exact owner/ZOMB/WAIT publication,
bystander progress, and stable PID/MM/slot accounting. Both normal
OOM-disabled and disposable requester-OOM-enabled variants complete
24 PID-reuse lifetimes and distinguish the original requester from
unrelated processes. D16 is CLOSED for the current writable-process
scope; scheduled XIP launch is still explicitly excluded.

Mandatory OOM checklist: **38/65 complete, 27 open** (25 pre-enable
blockers plus F12/F13). The remaining C04, B06/B07, E05-E13, F02-F11
and full actual RAM/release signoff remain mandatory. A full-image KCC
build emits known identifier-truncation and unused-static diagnostic
notes; F10 has not been accepted. The production OOM flag remains
disabled, with no resident kernel/MRES code/data or ABI change.

## v1144: D14 combined real MM/provider acceptance

The 256K PDP-6 `daimos-testkit` integrated v1144 guest PASSes one
real physical pin/MM_BUSY/unpin/free each for four distinct owned
extents: PIPE, Type-340 DPY raw list, process VM and executive
u-area. The original fatal owner retains its real filesystem-provider
lock and sleeping waiter through FILE/HUP retries. A pre-commit
WAIT(NO-HANG) returns zero, and a publication witness checks that
PIPE/DPY/FILE release precedes VM destruction and VM/u-area release
precedes zombie publication. The final parent WAIT/CHLD is correct;
one swap detach, 48 stale WAIT/PID/event checks and independent
spinner progress PASS. All witness errors are zero.

The original separately rebuilt v1118 mixed PIPE/VM/FILE/HUP guest
remains PASS after this testkit extension. The v1087 transitive
release suite and its native regression/mutation controls also
PASS; an indefinitely in-flight foreign provider remains subject
to the existing fail-closed lifetime policy, not bounded completion.

D14 is CLOSED for this release scope. Mandatory OOM checklist:
**37/65 complete; 28 open** (26 pre-enable blockers plus deferred
F12/F13). D16, C04, sustained E/F and size/signoff remain pending.
Only the disposable test image contains the new hooks and counters;
there is no production kernel, MRES, ABI or OOM switch change.

## v1143: D15 completed by explicit unscheduled-XIP exclusion

For the current requester-only BRK OOM activation scope, ordinary
production EXEC and RUN use writable process startup, not MEMFS XIP
process admission. The current source-linked v1123 scope/mutant
verifier PASSes with six unsafe variants rejected. Current PDP-6
v1108 and v1109 MEMFS MRES XIP physical release/retry tests PASS,
including unsafe mutation rejection. The kernel supports XIP-owned
VM/provider release but not scheduled XIP process launch. D15 is
therefore CLOSED only for the present OOM scope, which EXCLUDES
scheduled XIP processes. The separate future trusted-XIP process
launch work X03-X08 is still required before DAIMOS release and
would require requalifying OOM if subsequently enabled.

Mandatory OOM checklist: **36/65 complete, 29 open** (27 pre-enable
blockers plus deferred F12/F13). D14/D16, C04, E/F remain open.
Production `PROC_OOM_BRK_ENABLED_V1000` is still zero; no production
kernel/resident/MRES/ABI change was made.

## v1142: physical MM_BUSY completes the D13 nested RUN subcase

The new full-system PDP-6 `daimos-testkit` target
`test-daimos-fatal-exec-physical-uarea-v1142` PASSes a combined
APR fault, EXEC postcommit FILE refusal/retry, and unpublished
post-VM RUN rollback. The test pins the child's actual allocated
u-area immediately before its real `mm_free`. The first free returns
MM_BUSY (-5); the original rollback defers the owner, then a checked
unpin permits a second real MM free in the slot-zero finalizer.
Two independently rebuilt guests each report exactly two frees,
one pin, one BUSY refusal, one unpin and zero errors, with parent recovery,
process-word/slot accounting, and the remaining EXEC child's
terminal WAIT/ZOMB all confirmed. No fake MM error replaces the
production allocator; all hooks live only in the disposable guest.

D13's physical unpublished-RUN requirement is now met after the
v1141 cross-object local relocation folding fix. Mandatory status
is **35 of 65 complete; 30 open**, of which 28 precede the later
activation/signoff actions F12/F13. D14-D16 and C/E/F remain open.
Production requester BRK OOM termination remains disabled.

## v1141: toolchain correction restores uninstrumented D13 subcase

The earlier v1130-v1140 D13 invalid-UUO diagnostics were generated by
an unsafe P10FOLD/DLINK cross-object local TEXT fold. Two equal numeric
LOCAL RH18 addends named different object-local destinations; the old
tools merged `proc_discard_ok` with unrelated syscall-dispatch text.
The corrected tools reject this fold and are tested by
`daimos-testkit` v1141 (both old-tool negative controls fail).

An independently rebuilt **uninstrumented** 256K PDP-6 nested guest
now PASSes the combined APR fatal, postcommit EXEC FILE refusals,
unpublished failed RUN rollback, parent continuation and WAIT;
`finish_zombie=1` and `error=0`. Historical full v890 EXEC/FILE/WAIT
and native v1129 VM_NONE regressions also PASS. No DAIMOS production
kernel source or OOM policy was modified. A separate v197 full guest
also PASSes 16 post-VM failed RUN mappings under three live children;
the nested uninstrumented v1126 guest PASSes a second fresh rebuild.
The native folding fix and
its relocation rules are documented in
`TOOLCHAIN/P10FOLD-LOCAL-RELOCATION-SAFETY-V1141.MD`.

This resolves the uninstrumented D13 return regression but does not
complete the required integrated *physical MM_BUSY* unpublished RUN
rollback or other D/E/F gates. Keep D13's mandatory checkbox open
until that evidence is accepted; requester OOM remains OFF.

## v1130-v1133: D13 remains blocked by nested syscall-return fault

After the v1129 full-VM_NONE destructor repair, the unchanged v890
EXEC/FILE/WAIT guest PASSes. Separately, v197 inherited-group and
v1130 NEW-group full PDP-6 guests each PASS sixteen real post-VM
fd-map RUN rollbacks with three blocked children and stable process
accounting. The nested v1126 APR+fatal-EXEC+failed-RUN guest still
stops before its parent R marker at a simulator UUO; the EXEC child
is not published as a ZOMB.

Disposable v1131 full rollback tracing changes this to `<EARH` and
one terminal ZOMB, v1132 unused-BSS-only padding does not, and v1133
minimal RUN claimed/return counters show the rollback entry and
return label were reached but do not restore parent R. These
diagnostics are NOT acceptance. D13 stays OPEN pending a proven
mapped-user syscall/parent continuation fix plus genuine MM_BUSY
unpublished-RUN rollback integration. The complete 29-item
pre-activation gate remains blocked; requester-only OOM stays OFF.

# DAIMOS PDP-6 requester-only OOM: activation gate v1123

Status: **NOT READY** (2026-11-04). This is the persistent OOM
acceptance specification, not an implementation handoff. The detailed
action-by-action checklist is the top `v1123` section of DAIMOS
`TODO.MD`. That section assigns unique IDs S01-S08, A01-A04,
B01-B07, C01-C04, D01-D16, E01-E13, F01-F13 and X01-X08.
At v1125, 34 of 65 mandatory items are complete. Of the
remaining 31, 29 block the *pre-activation decision*; F12 (enable
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
| D: Error-stage ownership | OPEN | Physical TTY/VM/u-area PASS v1124 and wrong-owner -4 with foreign FS lease PASS v1125; complete APR/EXEC, mixed locks/VM/FILE and exact-once multi-resource release |
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

- v1129: The exact `VM_SPACE_NONE` (-1) sentinel is now recognized
  by the production PDP-6 VM destructor before any physical MM/XIP
  free, while checked backing detach remains active. Native source-
  linked positive and negative-mutant tests PASS; the previously
  timed-out *unmodified* full v890 post-commit EXEC guest now PASSes
  its real FILE retry and parent WAIT. This fixes a D13 prerequisite,
  but the combined nested APR/EXEC/failed RUN guest is still OPEN.

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
- v1124: a real canonical TTY line pin rejects owner-side EXIT HUP,
  leaving the original session attached. A second user EXIT unpins
  TTY, then separate real process-VM and u-area MM_BUSY refusals
  are recovered by the qualified-clock slot-0 reaper with correct
  PREVM, VM_NONE, u-area owner and exact final commit. All three
  distinct busy/pin/unpin counters are 1, and the independently
  rebuilt legacy v969 TTY-only guest also PASSes after preflight
  trigger modernization. D11 was checked at v1124.
- v1125: a real u-area MM descriptor is given a wrong owner in the
  disposable kernel and all seven real `mm_free` retries return -4.
  Another process holds the actual serialized FS gateway lease
  `5,,0` throughout those MM refusals on its own private kernel
  continuation, with 80 sleep cycles. The lease is sampled directly
  inside the MM faulting path and again at HALT. The original faulted
  owner remains in MM teardown, an independent spinner advances, and
  the parent observes no premature WAIT completion or forced slot
  release. Independently rebuilt v1101 legacy guest and the current
  source-linked v1096 release matrix also PASS; the optional complete
  `P0D_FULL=1` boot matrix has not yet been rerun. D12 is checked.
  D13-D16 remain open.

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
