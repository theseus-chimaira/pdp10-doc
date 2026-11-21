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
non-NONE VM PREVM path had a zero-base branch that skipped the
owner rather than registering an unsuccessful release. That old
branch **already prevented the low-memory dereference**. DAIMOS
`0f7a783f` changed SKIP to FAILED so an inconsistent stable-u-area
owner does not silently clear the shared pending bit. This is
defensive descriptor validation, not evidence that a valid path
creates a zero base.
Isolated PDP-6 DAS assembly PASS; physical acceptance outstanding.

## 2026-11-21 v1758: EXIT versus unpublished DISCARD owner

The foreign fatal-event path and the slot-zero reaper both invoke
`proc_finish_slot`. Previously its TRANSITION resume check accepted
any FINALIZE marker, including FINALIZE|DISCARD from an unpublished
RUN rollback. The reaper ordinarily routes DISCARD to
`proc_slot_discard`, but the EXIT entry point lacked that guard.
DAIMOS `72ca8a9c` rejects DISCARD at EXIT resume, before VM/u-area
release or zombie publication. Isolated PDP-6 DAS assembly PASS;
physical acceptance remains outstanding and K04 stays OPEN.

## 2026-11-21 v1759: PREVM diagnostic fallback

The opt-in GETTEARDOWN query now bypasses fatal FILE-stage
precedence only when the descriptor contains both FINALIZE and
PREVM marks. TRANSITION plus a live VM alone does not establish
that ownership. When the marks are absent, the original FILE-stage
check runs; exact PREVM EXIT owners still outrank the stale fatal
bit. DAIMOS `68200592` assembled with PDP-6 DAS. No ABI or
resource-lifetime changes; physical acceptance remains OPEN.

## 2026-11-21 v1760: committed PREVM fatal HUP ownership

A foreign fatal event can finish FILE closure but fail on TTY or VM
release. The failing finalizer retains TRANSITION and an exact EXIT
FINALIZE|PREVM marker for slot-zero retry. Previously the foreign
event path treated only VM_NONE+FINALIZE as committed delivery;
PREVM remained reported as failed. If session HUP was retried, its
member scanner also recognized only VM_NONE finalization, so it
attempted another HUP on the PREVM owner. Event delivery rejects
TRANSITION and the session leader could stay blocked indefinitely
under a persistent provider refusal.

DAIMOS `d29efc4d` makes an exact, non-DISCARD PREVM owner count as
committed fatal delivery and skips it during later session HUP
passes. Unmarked or pre-commit failures continue to be reported;
no resource is freed without provider acceptance. Isolated PDP-6
DAS assembly PASS. K04 remains OPEN for live fault-injection,
multiple-owner, clock-wrap and WAIT/PID reuse acceptance.

## 2026-11-21 v1761: session membership after VM teardown

The shared session scanner previously counted a process that had
committed VM destruction but still retained its pinned u-area as a
live session participant. If process A reached post-VM FINALIZE
while B remained in the session, A could leave the TTY intact.
When B later exited, it could see A as a member and skip TTY
retirement; A's later u-area retry never revisited TTY cleanup.
This path also applies to unpublished RUN rollback with retained
post-VM MM ownership.

DAIMOS `95fd5e65` makes `proc_tty_session_has` exclude only an
exact TRANSITION+VM_NONE+FINALIZE owner **without PREVM**. PREVM
continues to count because its TTY cleanup may still require retry.
The post-VM process still retains its u-area, slot and identity
until MM accepts its release. An inconsistent UAREA flag with a
zero base now returns -1 rather than reading low memory: TTY release
conservatively retains ownership, while SETFG rejects that error.

No additional persistent state or forced release. Isolated PDP-6
DAS assembly PASS; the ordered A/B TTY case and other physical K04
acceptance scenarios remain unverified. K04 stays OPEN.

## 2026-11-21 v1762: DISCARD is not a committed HUP

The session HUP retry path introduced in v1760 skipped an already
marked VM_NONE finalizer, but did not distinguish an unpublished
RUN rollback's FINALIZE|DISCARD from a fatal EXIT's FINALIZE.
The PREVM path excluded DISCARD; the post-VM path did not.

DAIMOS `22278d35` checks DISCARD in the shared marked-owner
branch. A rollback owner cannot satisfy fatal HUP completion;
ordinary delivery reports its TRANSITION state as busy, retaining
the session leader's retry owner. Valid committed EXIT owners are
still skipped. Isolated PDP-6 DAS PASS; physical acceptance OPEN.

## 2026-11-21 v1763: first-refusal owner arming under shared pending

`proc_exit_current` publishes FINALIZE|PREVM before attempting to
release VM resources. With another owner already pending, the old
registration helper interpreted that prepublished marker and the
global pending bit as an existing refusal. A newly failed EXIT
owner therefore inherited the old owner's (possibly saturated)
retry delay and epoch, rather than starting a 020-octal-tick delay.

DAIMOS `36859b24` assigns an armed marker to each FINALIZE owner
upon its first failed release. The bit comes from the otherwise
exclusive swapped-RUN retry-tag field (bit 32 of existing u-area
word 067); it is interpreted as armed only when FINALIZE is set.
Unarmed owners reset the shared retry clock to the present epoch
and 020-octal-tick delay, even if another owner is pending. Armed
owners preserve the current backoff on further failures. The swap
and finalization lifetimes cannot coexist in one process.

This adds no permanent RAM, process slot or new user ABI. The SIDL
no-u-area pre-VM retry remains unchanged. Native PDP-6 DAS
assembly PASS; live overlapping-refusal, clock-wrap, WAIT/PID
reuse and permanent-provider acceptance remain OPEN.

## 2026-11-21 v1764: WAIT and HUP inconsistent-owner guards

`proc_wait_status` previously checked only that a non-ZOMB child
retained its u-area before reporting STOP/CONT. Deferred EXIT and
unpublished RUN rollback both retain a u-area under TRANSITION;
their old job-control report could therefore be consumed while
terminal MM cleanup remained refused. DAIMOS `42ded8c3` excludes
TRANSITION owners from STOP/CONT reports, without hiding them as
children or reaping them before ZOMB. A zero u-area base is also
rejected before the report read.

The session HUP member scan likewise checked the u-area flag but
not its base before reading session identity. An inconsistent
zero-base member now causes session teardown failure, preserving
the TTY owner rather than inspecting low memory and potentially
committing an incomplete HUP. No extra resident RAM or ABI change.
Isolated PDP-6 DAS PASS; physical K04 acceptance remains OPEN.

## 2026-11-21 v1765: reaper retry phase ordering

The first-refusal registration introduced in v1763 reset a new
owner's shared retry interval to 020 octal ticks. But when that
registration happened *inside* a slot-zero reaper scan, the idle
caller then invoked `proc_finish_backoff_advance_v885` after the
scan, immediately doubling the freshly registered 020 interval
to 040. This could affect a PREVM owner which first encounters
MM refusal during reaping, or an owner that crosses from PREVM to
post-VM u-area retry within the scan.

DAIMOS `419ed004` advances the already pending owners' interval
**before** servicing the scan, under the existing PI6 inhibition.
This lets an owner first registered during the scan preserve its
020 interval and current epoch; older persistent failures continue
to back off at 020, 040, 0100 ... 01000 octal ticks. If no owner
remains pending, the reaper clears the unused delay as well as the
pending marker from the existing clock word.

No added resident RAM, scheduler ABI or forced provider release.
Native PDP-6 DAS PASS; physical concurrent-refusal, wraparound,
WAIT/PID reuse and permanent-owner acceptance remain OPEN.

## 2026-11-21 v1766: queued HUP is not completed HUP

The foreign fatal-event API returns zero when HUP is queued for
an owner with a suspended private executive continuation. The
session leader previously treated this as completed HUP delivery,
allowing controlling-TTY retirement while the member still owned
open FILE entries, VM, and its u-area. This violated the session
teardown ordering requirement.

DAIMOS `047a4ac1` verifies the target after `proc_event_apply`.
No-u-area terminal publication succeeds; a retained u-area must
have TRANSITION and an exact FINALIZE, non-DISCARD marker (with
PREVM if VM remains live). An otherwise still-live member leaves
the session teardown unsuccessful, retaining the TTY and the
leader's private-stack retry owner. Pending HUP coalesces until
the target can complete it; the general asynchronous event-send
contract is unchanged. No new permanent RAM. Isolated PDP-6 DAS
PASS; physical deferred-HUP and multi-owner acceptance OPEN.

## 2026-11-21 v1767: dying INIT cannot adopt orphan children

The orphan reparenting path formerly selected INIT (slot 1) if
its state was anything except FREE/ZOMB. A deferred EXIT retains
its scheduler state and u-area while the MM provider refuses
release, and a fatal FILE-close owner can retain a private stack
before TRANSITION publication. Neither can execute WAIT again.
If another parent terminated during this period, its children
were assigned to this nonreturning INIT, potentially leaving
completed zombie slots unreapable under persistent refusal.

DAIMOS `821d8be5` allows INIT adoption only for RUN/SLEEP/STOP
with a flagged nonzero u-area, no TRANSITION and no fatal FILE
close. Otherwise the existing parentless-orphan fallback handles
the children: live processes become parentless and orphan zombies
are immediately reaped. Normal live INIT adoption is unchanged.
The parent CHLD notifier now also validates u-area flag and base
before accessing the control word, avoiding a low-core write on
an invalid parent.

No permanent RAM, process-table layout or ABI change. Isolated
PDP-6 DAS PASS; real orphan/INIT refusal, WAIT/PID reuse and
recovery acceptance remain OPEN. External testkit untouched.

## 2026-11-21 v1768: late child EXIT under dying parent

v1767 protected orphan adoption when a parent *successfully*
finished, but a child could still exit while its parent retained
TRANSITION+FINALIZE (or an irreversible fatal FILE-close owner)
under provider refusal. The native `proc_finish_slot` published
the child as ZOMB under that now-nonreturning parent, and
`proc_notify_parent` could queue CHLD into the parent's preserved
u-area. Indefinite refusal left the child's PID and slot pinned
even though the child's own resources had been completely released.

DAIMOS `0a15e82b` shares one WAIT-capability check between INIT
adoption and child terminal publication. Eligible parents are
RUN/SLEEP/STOP with a flagged nonzero u-area, without TRANSITION
or fatal FILE-close ownership. If a child's original parent is
ineligible, a healthy INIT receives the zombie. If INIT is also
ineligible, the child is reclaimed immediately instead of being
published as an uncollectable zombie. The notifier now suppresses
CHLD and WAIT wakeup to irreversible teardown owners.

No new resident RAM fields or process-table ABI; isolated native
PDP-6 DAS PASS. Physical refused-parent, healthy-INIT fallback,
both-parents-dying, WAIT/PID reuse and clock-wrap tests remain OPEN.

## 2026-11-21 v1769: full-system build gate

Following the v1768 ownership correction, an isolated full
`make -C system build` initially failed because all five PDP-6
Stage-1 Makefiles (DSK, DTC, DRM, MTC, PTR) still depended on
`common/decompressor.inc`, a file no longer present. Their Stage-1
assembly sources actually include `common/block_decompressor_v1458.inc`.

DAIMOS `5228712d` corrects these stale prerequisites without
altering decoder code or image format. A fresh build under a
versioned `$TMPDIR` build root passed, including the five Stage-1
artifacts. This is a full-system compile/build gate, **not** a
booted or fault-injected K04 acceptance result. No testkit run,
permanent RAM increase or KCORE size optimization was performed.

## 2026-11-21 v1770: production boot and clean reboot smoke

The first fresh 256K PDP-6 DSK boot-image build after v1769
assembled and linked KCORE but failed native KCC source staging.
DAIMOS staging requested unversioned startup/runtime object names,
whereas KCC installed `crt0-v1.dobj`, `daimos-bootstrap-v1.dobj`,
and `syscall-helpers-v1.dobj`. DAIMOS `c1155cd6` makes the
consumer use those authoritative installed names. KCC `fe6dff8`
also installs the previously omitted
`runtime/fp36-conversions-v1750.a` native bootstrap archive.
The repaired KCC `make install-native` and fresh DAIMOS image build
both completed successfully.

Real Open SIMH PDP-6 smoke: DAIMON V0.5, 256K, 60-Hz clock,
D6FS LOADED and INIT V1 reached `LOGIN:`. ROOT entered DSH V1;
PS showed TD/RETRY/PVM/DETAIL/ERR columns with normal zero
teardown state. A TRUE child returned normally, followed by PS.
Guest HALT completed at octal `022464`. Read-only D6FS checking
reported superblock 742 seq=3 CLEAN (179196 blocks, 360 FCBs).
A clean reboot repeated INIT, ROOT login, PS, guest HALT and D6FS
CLEAN seq=5. Detailed evidence: DAIMOS
`design/kcore-k04-native-boot-smoke-v1770.md` (`de20f1be`).

An earlier host-forced SIMH stop at LOGIN left the disposable
D6FS root DIRTY (superblock 743 seq=2); a subsequent boot
refused it with `?RT` as specified by root validation. Fresh
media restored CLEAN seq=1, and guest-controlled shutdowns
preserved clean media thereafter. This is not K04 fault injection.

K04 remains OPEN for refused FILE/MM owners, independent
multi-owner progress, retry clock wrap, deferred HUP,
orphan WAIT/PID reuse and owner-preserving operator recovery.
No external testkit was run or modified and no permanent kernel
RAM was added by the bootstrap packaging corrections.

## 2026-11-21 v1772: repeated normal child reuse

Using the clean v1770 256K PDP-6 image, a live ROOT/DSH session
executed five consecutive TRUE children successfully. The next
PS ran as PID 5 and showed no TD, RETRY, PVM, DETAIL or ERR
ownership for the normal processes. This is evidence of normal
child completion and slot reuse, **not** reuse while an MM/FILE
provider refuses teardown. Guest HALT at PC 022464 completed;
read-only D6FS check reported superblock 742, seq=7, CLEAN,
179196 blocks and 360 FCBs.

The first emulator launch in this session died with host signal
11 after TTY LOADED; the filesystem was still CLEAN, and the
next launch booted normally. Treat this as an unresolved host
emulator anomaly, not proof of a guest kernel fault. Full record:
DAIMOS design/kcore-k04-repeated-child-reuse-v1772.md,
commit 6769687d. K04 physical refusal/multi-owner acceptance OPEN.

## 2026-11-21 v1773: fatal WAIT_CHILD wakeup, PDP-6 negative control

DAIMOS `6d99565a` corrects a deferred fatal-event deadlock.
`proc_event_fatal_other_defer_term_v926` queued INT/TERM/HUP/PIPE
without waking a sleeping `PROC_WAIT_CHILD` owner, although that
owner's saved `proc_wait_intr_return` continuation already tests
the fatal bits. If a child never exited, its parent's fatal event
could never reach the private owner-stack cleanup at user return.
Additionally `proc_wait_child` checked only ALRM before entering
sleep, so a previously pending fatal event could suffer the same
indefinite wait.

The correction wakes WAIT_CHILD through the existing interruptible
SLEEP-to-RUN path and checks ALRM/INT/TERM/HUP/PIPE at WAIT entry.
Foreign FILE/MM release remains prohibited; STOP and internal EVENT
wait semantics remain unchanged. The full production 256K PDP-6
image and native DAS passed. `__kcore_low_end` changed from octal
025121 to 025126, adding five resident TEXT words and zero DATA/BSS.

Real dual-terminal CTY/DCS0 PDP-6 comparison: CTY USER PID 2
entered WAIT for a blocked foreground CAT PID 5. DCS `KILL 2`
returned successfully. Under the unchanged v1770 executable,
repeated DCS PS still showed USER PID 2 and CAT PID 5 sleeping.
Under v1773, DCS PS instead showed PID 2 replaced by LOGIN,
CAT absent and PID 5 reused by PS. Both trials used isolated
clean disk media and finished with guest HALT and D6FS CLEAN
(v1773 PC 022471, seq=3; v1770 PC 022464, seq=11).

Detailed source and physical evidence: DAIMOS
`design/kcore-k04-fatal-wait-child-wakeup-v1773.md` (`3ee406a0`).
The external testkits were not touched. K04 **remains OPEN** for
actual FILE/MM refusal, simultaneous owners, clock wrap, and
WAIT/PID release under refused ownership.

## 2026-11-21 v1774: real PDP-6 post-VM MM_ERR_BUSY recovery

An isolated v1773 PDP-6 boot image was subjected to a controlled
SIMH instruction substitution at `proc_uarea_release`: octal
013616 originally `PUSHJ 17,mm_free` (`260740002316`) was
temporarily changed to `PUSHJ 17,kret_neg5` (`260740004715`).
This returns typed `MM_ERR_BUSY` only for u-area release, leaving
VM and other MM free operations unchanged. The production source,
boot binary, and external testkit were not modified.

The DCS ROOT shell ran foreground TRUE and remained blocked.
SIMH inspection found child PID 5 with TRANSITION, live u-area
054363, VM_SPACE_NONE, and FINALIZE+ARM word 067 (`440000000000`).
After the APR clock advanced from octal 012752 to 014026
(octal 01054, 556 decimal ticks), the same owner remained
quarantined. Restoring the exact `PUSHJ 17,mm_free` instruction
allowed the pending reaper retry to complete. The shell returned,
and PS reused PID 5 with TD/RETRY/PVM/DETAIL/ERR all zero.
Guest HALT PC 022471 and D6FS CLEAN seq=5 PASS.

Evidence: DAIMOS `design/kcore-k04-postvm-refusal-simh-v1774.md`
(`668d39f9`). This verifies synthetic post-VM provider refusal
and subsequent owner release on the running PDP-6, **not** actual
MM pin accounting or precise retry timing. K04 remains OPEN for
physical pin, multiple independent owners, clock wrap, FILE
refusal, and HUP acceptance.

## 2026-11-21 v1776: actual MM extent pin and deferred release

DAIMOS `7f179594` records a live Open SIMH PDP-6 test with a
scratch-only assembled `proc_uarea_release` fixture. For PID 5,
the fixture calls real `mm_pin` on the descriptor's owned u-area,
then invokes the ordinary `mm_free` on each cleanup attempt. It
checks the same base and calls real `mm_unpin` on attempt ten.
No provider return-code substitution, direct MM extent metadata
edit, production source change or external testkit edit was used.

With CTY foreground TRUE blocked, DCS PS observed PID 5 retained
in teardown (TD=2, RETRY=512, PVM=0, DETAIL=6). Read-only SIMH
inspection showed the same pinned u-area base 054377 through
release counts 5, 6, 7 and 9. After real unpin, the CTY parent
resumed and DCS PS reused PID 5 with all teardown diagnostics
zero. Final fixture memory: base=0, attempts=octal 012 (ten),
done=all ones. Guest HALT PC 022527 and D6FS CLEAN seq=3 PASS.

The scratch KCORE low-end 025167 includes the test code and three
test words; it is **not** a production memory-size change.
The physical pin test demonstrates owner preservation, retry and
eventual real provider release, but does not close independent
multi-owner, clock wrap, FILE refusal, HUP or other-stage gates.
K04 remains OPEN. Full test evidence is in DAIMOS
`design/kcore-k04-physical-mm-pin-v1776.md`.

## 2026-11-21 v1777: two independent real MM-pinned owners

DAIMOS `37bf2680` records a disposable PDP-6 image with two
independent post-VM owners pinned through the actual `mm_pin`
provider. PID 5 remains pinned for 24 cleanup attempts; PID 6
is released by real `mm_unpin` on its fifth attempt. Neither
production kernel source nor the external testkit is modified.

Two ROOT/DSH terminals ran foreground TRUE. Read-only SIMH
inspection initially showed PID 5 base 054441 at attempt 9 and
PID 6 base 055051 at attempt 4, both still pinned. PID 6 then
unpin/released, and its DCS parent resumed while PID 5 remained
pinned. Subsequent PS reused PID 6 with zero teardown diagnostics,
while PID 5 retained TD=2, RETRY=512, DETAIL=6. This is a
**physical independent-owner progress PASS**, not merely a
synthetic MM_ERR_BUSY return.

The separate initial 020-tick delay, SIDL/PREVM orderings, clock
wrap, FILE refusal, HUP and dying-parent WAIT remain OPEN.
Details: `design/kcore-k04-two-physical-owners-v1777.md`.

The same guest subsequently completed PID 5's twenty-fourth
cleanup attempt: both fixture pin bases were zero, both completion
flags were all ones, and CTY's foreground TRUE returned. Guest
HALT PC 022566 and final read-only D6FS CLEAN seq=3 PASS.
DAIMOS evidence completed in `a22dd30d`.

## 2026-11-21 v1778: first refusal resets saturated physical owner delay

DAIMOS `ea875bbf` records a third, uncontaminated real PDP-6
two-owner physical-pin trial that closes the remaining **maximum
shared-delay / new-owner first-refusal timing** scenario. The
disposable v1778 image keeps the real `mm_pin`, `mm_free` and
`mm_unpin` paths for both post-VM u-areas. Four scratch-only BSS
snapshots capture the retry word before and after B's first
refusal and the qualified APR clock at B's first and second
release attempts. Production KCORE and the testkit are unchanged.

CTY foreground TRUE pinned owner A (PID 5) until a read-only
SIMH examine directly confirmed a saturated shared retry word
`401000005134` (octal). No DCS child ran before this point;
PID 6's base, counter, completion and timing snapshots were all
zero. The DCS TRUE command then started owner B (PID 6).
Read-only guest memory captured:

    B before first refusal    401000005134  ; pending + 01000
    B immediately after mark  400020005477  ; pending + 020
    B first refusal clock     000000005477
    B second release clock    000000005517

The second minus first clock is exactly octal 020 ticks (16
decimal). Therefore the real first B retry was not delayed by
A's saturated 01000 interval. B's fifth real release attempt
unpinned/freed its u-area, its parent resumed, and DCS PS reused
PID 6 with teardown diagnostics zero. The same PS still showed
owner A retained with TD=2, RETRY=512 and DETAIL=6. Operator
guest HALT PC 022607 and read-only D6FS CLEAN seq=7 PASS.

Two preliminary trials used DCS PS to check A's delay, but PS
itself was PID 6 and prematurely activated B's fixture. They
correctly measured a 0400 -> 020 reset but are not accepted as
maximum-delay tests. The final trial avoided that harness error
by using only read-only SIMH examine before launching B.

This closes K04's **two post-VM owner maximum-delay reset and
exact first retry timing** gate. SIDL/PREVM, 17-bit clock wrap,
FILE, HUP and dying-parent WAIT cases remain OPEN. Full evidence:
DAIMOS `design/kcore-k04-max-delay-reset-v1778.md`.

## 2026-11-21 v1780: physical SIDL VM rollback

DAIMOS `ee597108` records a disposable PDP-6 image that forces
one RUN child (PID 5) into the existing SIDL rollback path after
VM/u-area ownership is established. Scratch-only instrumentation
pins the child's exact VM extent with real `mm_pin`, allowing
unchanged `vm_space_destroy` to encounter real MM refusal. After
five discard attempts, real `mm_unpin` permits the ordinary
rollback to finish. The injection is disabled automatically after
the first completed rollback so normal commands can execute.

Read-only SIMH monitor inspection: VM pin base 0, attempt count
octal 005, completion all ones. The next PS reused PID 5 with
TD/RETRY/PVM/DETAIL/ERR zero. Guest HALT PC 022536 and read-only
D6FS CLEAN seq=3 PASS. No production source, ABI, fixed RAM or
testkit changes. The first repeating-failure v1779 fixture was
discarded because it prevented guest HALT; the accepted v1780
run used fresh clean media and a one-shot failure trigger.

This passes one real SIDL VM-refusal/rollback/PID-reuse scenario.
Permanent refusal, SIDL/post-VM simultaneous ordering, PREVM EXIT,
clock wrap, FILE/HUP and dying-parent WAIT remain OPEN. Full
evidence: `design/kcore-k04-sidl-real-vm-pin-v1780.md`.

## 2026-11-21 v1776: actual physical MM pin/refusal/recovery

K04's post-VM retry now has **real PDP-6 physical pin evidence**.
A disposable v1773-derived 256K kernel contained a source-level
test-only fixture within `proc_uarea_release` for PID 5. The
fixture called the authoritative `mm_pin(uarea_base)` before
the unmodified `mm_free(base, MM_TYPE_KERNEL_DYNAMIC, 01000+PID)`
operation, then retained that pin through ten release attempts
before calling the real `mm_unpin`. There was no substitute
MM_ERR_BUSY result and no production source change. The three
additional test state words are absent from production KCORE.

While the CTY shell waited for foreground TRUE, DCS0 `PS`
showed PID 5, TD=2, RETRY=512, DETAIL=6, with the slot still
allocated. Detail 6 is stage 2 plus one physical MM pin shifted
two bits, matching `SYS_TEARDOWN_MM_PINS`. Read-only SIMH memory
inspection captured u-area base octal 054424, attempt counter
octal 010 (eight) and incomplete flag zero. An additional PS
still showed the pinned deferred owner; no premature PID reuse.

After the tenth real release attempt, `mm_unpin` succeeded,
ordinary `mm_free` completed, and the parent resumed. DCS PS
reused PID 5 and showed all teardown fields zero. The final
read-only fixture snapshot showed base zero, counter octal 012
and completion flag all ones. Guest HALT PC 022527 completed,
and read-only D6FS check reported CLEAN seq=3, 179196 blocks,
360 FCBs.

The full physical acceptance record is in DAIMOS
`design/kcore-k04-physical-uarea-pin-v1776.md` (`6af842d9`).
This closes the **single physical u-area pin refusal/recovery**
scenario only. It does not establish two independent owners,
exact backoff progression, clock wrap, SIDL discard, HUP, FILE
refusal, or dying-parent WAIT. K04 therefore remains OPEN.
