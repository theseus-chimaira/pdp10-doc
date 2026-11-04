## v1124: booted physical canonical TTY, VM and u-area retry

The 256K PDP-6 v1124 full guest PASSes three actual independent
MM_BUSY (-5) failures during the final INIT process lifetime.
The first EXIT returns -1 from its genuine private-stack owner-side
HUP preflight because the original canonical TTY input buffer is
physically pinned. The session and process remain attached and
owned. A second user-issued EXIT unpins the same TTY extent and
commits HUP; its original resident process VM is independently
pinned and refuses physical release once. The real qualified-clock
slot-0 PREVM reaper retains the VM owner, unpins it, and pins the
still-live u-area before its subsequent true MM deletion.
The u-area's next real MM free refuses once while pinned, leaving
VM_NONE/FINALIZE and the exact owner intact until the following
clock-driven retry unpins and frees it exactly once. TTY, VM and
u-area MM_BUSY count=1 each; pin/unpin count=1 each; retained
VM/destroyed VM=1, retained u-area=1, terminal commit=1,
ownership errors=0. No synthetic error, forced reclaim or hand-
invoked finalizer appears in the accepted instrumentation.

The original v969 TTY-only guest was separately rebuilt/rebooted
with its arm fixed for the v991 owner-side HUP preflight: PASS,
one real pin and exact unpin/retry. D11 is checked. The v1123
readiness inventory now counts 33/65 mandatory complete, 32 open,
including 30 preenable blockers and two post-signoff actions.
D12-D16 and remaining full P0-E/F acceptance stay OPEN. Test-only
instrumentation, production OOM disabled; no production kernel
code/data, MRES or process ABI changed.

## v1123: production enablement prerequisites and optional-XIP scope

The active fine-grained acceptance list is now the top v1123 section
of DAIMOS `TODO.MD`, explained in
`KERNEL/OOM-ENABLEMENT-CHECKLIST-V1123.MD`. It has 65 mandatory
S/A/B/C/D/E/F tasks (32 checked, 33 OPEN: 31 pre-enable blockers,
plus F12/F13 post-signoff actions) and eight separately
optional raw-XIP tasks (two checked). P0-A v1087, P0-B v1092
ownership containment, and P0-C v1086 are closed; P0-D/E/F and
the production switch remain open. Permanent provider quarantine
is safe against premature resource reuse but does not guarantee
reclamation or eventual completion.

The v1123 testkit source guard confirms the current default-off
`PROC_OOM_BRK_ENABLED_V1000,0` and the writing startup calls in
ordinary RUN and both EXEC paths, with no raw MEMFS XIP admission
wired. It rejects six unsafe mutations. The checklist auditor
rejects five invalid/premature-completion mutations and its
`--require-ready` check correctly fails with 31 pre-enable blockers;
the other two unfinished actions follow signoff. The full
source-linked v1096 P0-D matrix PASSes on current
code. This does not authorize the separate privileged XIP process
launcher or OOM activation. No production kernel memory/ABI changed.

## v1122: combined cross-owner physical PIPE/MM and DISK bitmap faults

A complete disposable 256K PDP-6 session-HUP guest now reaches two
different real checked release refusals across two process owners.
Session member 3 holds a real private PIPE extent. Its first HUP
reaches final FILE/PIPE close; a test-only `mm_pin` makes the actual
physical `mm_free` return MM_BUSY (-5), retaining the original PIPE
owner and blocking HUP. One later HUP re-enters that exact member,
unpins and successfully frees that PIPE. The second session member,
slot 4, independently enters a genuine DISK swap-out and owns a
physical BACKSTORE bitmap allocation. During its fatal teardown,
the production bitmap verifier detects one test-cleared allocated
bit, returning an actual refusal rather than a synthesized failure
code. Slot 0 preserves its PREVM owner until clock-backed retry,
when the bit is restored and the exact backing record freed once.

The full guest PASSes PIPE calls=2, one actual pin, busy, unpin and
free, zero owner errors and one first-member HUP revisit. The real
DISK verifier runs twice, with one deliberate bitmap corruption,
one repair and one backing free, zero swap errors, zero terminal
record and block count. The first member's pinned u-area, the
intermediate nonwaitable leader, three HUP attempts, terminal WAIT
statuses and TTY foreground handoff PASS. This directly closes the
cross-owner PIPE+DISK bitmap case that v1121 could not reach: v1121
restored its own DPY-holding member before DISK detach.

`daimos-session-hup-pipe-swap-v1122` registers this scenario in
daimos-testkit without modifying production KCORE, MRES, RAM, ABI or
OOM policy. Additional booted MEMFS XIP and sustained concurrent
provider/liveness cross-products remain open under P0-D/E/F, and
the production requester-only OOM branch stays disabled.
The original v1104 physical PIPE-only and v1107 real bitmap-only
full PDP-6 guests were independently rebuilt and run after v1122.
Both PASS the original checked fault/retry, ownership, WAIT and TTY
assertions; v1107 still observes exactly two bitmap verifier calls,
one deliberately missing bit, one restoration and one backing free.

## v1121: booted physical DPY/MM refusal across genuine DISK restoration

The v1121 256K PDP-6 disposable guest combines a Type-340 raw-list
MM_BUSY on second-member HUP with a previously completed production
DISK swap-out of that member. The first integration boot reached
normal HALT but exposed an invalid old v1107 test expectation:
the physical DPY FILE-close refusal leaves a private continuation
which must resume while resident. A second member-HUP delivery was
therefore legitimate, and the checked DISK bitmap-detach verifier
was *not* invoked. Those observations do not constitute a bitmap
fault and were not counted as successful bitmap-refusal coverage.

The accepted version adds test-only counters to the genuine SWAP
MRES `proc_swap_in` entry and successful return for this slot, with
the counters living in the disposable KCORE. The full guest PASSes
one real swap-out, one real successful swap-in, one backing free and
zero terminal DISK record/blocks. The live display list records two
release attempts, exactly one pin/MM_BUSY (-5), one unpin, one real
free, zero owner errors, and 118 PI6 refreshes, 90 after the first
busy error before unpin. Correct first-member HUP commit, later
second-member retry, leader/child WAIT and TTY handoff also PASS.

This is verified physical swap-restore/DPY fault integration and
excludes the unreachable bitmap-detach fault in this configuration.
An actual simultaneous bitmap-detach refusal and different resource
fault requires a distinct live owner which can retain DISK storage
through its checked finalization. Restricted MEMFS-XIP process
admission and complete P0-D/E/F acceptance remain outstanding.
Production KCore/MRES code/data, process ABI and disabled requester
OOM policy remain unchanged.
The original swap-only v1107 guest was independently rerun after the
v1121 generator changes and PASSed its actual checked bitmap fault:
two verifier calls, one deliberate corrupted bit, one restoration,
one exact backing release and no terminal DISK resource leak.

## v1120: actual slot/PID reuse after fully reaped physical OOM

Two independent full 256K PDP-6 guest images PASS a new sequential
PID-reuse regression after real physical BRK exhaustion. With
production OOM disabled in the first disposable image the exhausted
requester returns -1 normally; with OOM enabled only in the second
test image the genuine exhausted requester is terminated. The
independent blocked witness survives in both, and WAIT and SYS_MEMINFO
show exact recovery of process slots and process words to the parent
baseline before reuse begins.

Each guest RUNs, waits for and reaps 24 successive children with
alternating 062 and 063-octal exit statuses. All 23 consecutive pairs
reuse the same numeric slot/PID. A second WAIT and directed TERM
issued *after* each reap, before the next RUN, both return -1; the
parent observes the original process-slot/process-MM-word baseline
after each iteration. The transcripts end `<DNWQP` for the default
policy and `<KWQP` for the isolated enabled policy, both with clean
HALT. The test is `daimos-oom-pid-reuse-v1120` in daimos-testkit.

PIDs in DAIMOS are numeric process slots without generation tags.
An old PID can legitimately name a different live owner after reuse;
the test does not promise permanent stale-handle rejection or send
an event to the newly assigned PID under the old identity. This
closes sequential actual numeric reuse after OOM, not sustained
simultaneous HUP/PIPE/DPY/XIP/swap faults. The existing native v1109
real XIP/VM retry and v1087 allocation-free release audit also PASS.
Production kernel and MRES RAM, ABI and OOM switch are unchanged;
P0-D/E/F remain OPEN.

## v1119: physically faulted PIPE and Type-340 DPY in the same HUP

The v1119 disposable 256K PDP-6 session guest now equips the second
HUP recipient with both a real private PIPE extent (MM owner 5) and
a published Type-340 display-list extent (MM owner 014). The two
physical `mm_free` hooks are independently armed at the same actual
HUP delivery; the combined test generator preserves the original
entry, MM_BUSY, delete and owner/type paths for both.

Two independently assembled and booted guest images PASS, including
the final corrected C89 test-source build. Both distinct extents
report exactly two free attempts, one actual MM pin-induced -5 busy,
one corresponding MM unpin and one real release, without ownership
errors. The DPY raw list remains published and PI6 refresh executes
14 times, eight after MM_BUSY and before unpin. The session's first
member keeps its pinned u-area through the incomplete HUP; the
leader and all members complete without duplicate first-member HUP,
premature WAIT completion or incorrect TTY foreground ownership.

This satisfies a physical PIPE+DPY HUP cross-product. It does not
test live MEMFS XIP admission with simultaneous VM teardown, combined
DISK bitmap faults or numeric PID reuse under sustained pressure.
P0-D/E/F are still OPEN and production requester OOM remains OFF.
No production KCore/MRES RAM, ABI or kernel code changed; testkit
commit `42736cd` contains the test-only generator and fixture.

## v1118: booted real PIPE and process-VM MM_BUSY in one fatal teardown

The v1118 full 256K PDP-6 disposable guest now exercises *both*
physical MM release paths during the same fatal child retirement.
The test generator composes the v1103 PIPE MM_TYPE_KERNEL_DYNAMIC
free hook with the v1106 process-VM MM_TYPE_PROCESS free hook at
`mm_free`, preserving both owner/type tests; its first attempt
exposed and corrected a conflicting test-only insertion anchor.

The guest PASSes one physical -5 refusal and later successful free
for the exact PIPE extent, and one process-VM pin-induced -5 followed
by the real VM extent unpin/free and one swap detach. The existing
FILE close (three attempts/two refusals), HUP-stage (two calls/one
injected refusal), u-area (two calls/one refused release), exactly
one EXIT/CHLD/finalization, foreign provider lease, and independent
spinner (37 counted cycles) all PASS. After the child is reaped,
48 further stale WAIT and PID-directed TERM requests fail normally
without increased process-slot or process-MM-word accounting.

This is stronger physical cross-stage evidence than v1103 or v1106
in isolation, but the HUP refusal remains a deliberate stage fault.
Physical DPY/HUP, MEMFS-XIP admission/lifetime, simultaneous failing
DISK swap bitmap release, real numeric PID reuse, and complete P0-E
resource/liveness coverage remain OPEN. Production OOM remains OFF;
the entire production kernel, ABI and KCore/MRES footprint are
unchanged. Testkit: `daimos-oom-physical-mixed-v1118` (`7df48f0`).

## v1117: booted mixed fatal release and repeated stale PID/WAIT checks

The v1117 disposable 256K PDP-6 integration guest reuses the v1103
PIPE final-close MM_BUSY, FILE, HUP-stage, process VM and u-area
fault/retry scenario with a held foreign FS-provider lease and a
continuously runnable independent spinner. After the exact child
WAIT completes, the parent makes 48 further WAIT requests and 48
PID-directed TERM requests for the now-reaped child. Every request
must return -1. Process-slot and process-MM-word counts may decrease
through legitimate reclamation but must not increase above their
post-reap baseline. The original v1103 release-stage counters,
one EXIT/CHLD/finalization, provider ownership and bystander
progress also PASS in the booted guest.

A first attempt to create fresh children after this teardown was
invalid because the scenario intentionally holds the serialized
provider needed for executable loading. It showed continued spinner
progress but no new RUN completion; the accepted test instead checks
stale WAIT/PID/event semantics without requiring that provider.
This is a narrower P0-E liveness subset, not evidence of numeric
PID reuse or of complete HUP/PIPE/DPY/XIP/swap fault combinations.
No production code, KCore/MRES or OOM setting changes; production
requester OOM remains disabled pending P0-D/E/F acceptance.

## v1116: real MM descriptor-cause tracing across child RUN

Both independently booted disposable 256K PDP-6 configurations now
confirm actual MM descriptor saturation during a valid child RUN.
Test-only instrumentation records the native allocation result at
`vm_space_create` and the later child u-area allocation in
`proc_run_block`, before those layers collapse the result to the
public RUN ABI -1. The fixture demands exactly one recorded -3
(descriptor capacity exhausted) at the failing stage and records
the other as success (0) or not attempted (-7 sentinel). A failed
RUN retains no child process slot, user VM or u-area, and the
requester's physical memory accounting is unchanged.

The private test INIT first fills the real MM descriptor table with
one-word kernel-dynamic allocations, then frees each recorded base
using the true MM type/owner and continues the existing invalid
BRK/RUN, physical-OOM, blocked witness and WAIT tests. The
default-disabled guest PASSes: the exhausted requester receives -1.
The disposable enabled-policy guest PASSes: the later genuinely
physical-OOM requester alone is fatally retired. The RUN caller and
the independent witness survive.
Native v1111/v1113 MM classification and BRK cause propagation tests
also PASS, including their unsafe mutation controls.

All fault preparation, cause capture and enabled OOM policy exist
only in disposable images. Production OOM remains OFF; the kernel
text/data/MRES/ABI do not grow. The direct RUN -3 cause observation
is now covered, but P0-D release cross-products, P0-E integrated
liveness/WAIT/PID leak acceptance and P0-F enable decision remain
OPEN. These two boots are not a full P0-E matrix.

## v1115: full-system fragmented-core BRK preserves nonfatal -2

Two independently built, booted 256K PDP-6 images now verify actual
fragmented physical memory rather than a synthetic MM return code.
In an isolated, disposable syscall-copy test, the real allocator
reserves aligned 040000-word kernel-dynamic extents, then releases
alternating blocks. The genuine `mm_find_fit` search, exported solely
for this fixture, confirms at least 0200000 (64K) aggregate free words
but no fitting contiguous 64K extent. Real `vm_space_brk_current`
rejects a 64K resize with AC1=-1 and AC2=-2, observed through a
test-only AC2 recorder. The requester stays runnable, its original
logical break remains intact, and all recorded temporary MM extents
are released through exact `mm_free` base/type/owner checks.

A smaller 40K request had legitimately succeeded by in-place
delta-only growth despite lacking a full-image gap. The final 64K
request forces the full-image fallback. During the real BRK retry,
one unrelated MM descriptor was reclaimed in the observed guest;
the fixture therefore rejects a growing final descriptor count,
not legitimate reclamation that reduces it. Exact test extent release
and unchanged parent process-MM accounting remain required.

The OOM-disabled guest PASSes (`<FRILDNWP`): the later genuine
physical exhaustion also returns -1. The disposable OOM-enabled
guest PASSes (`<FRILKWP`): fragmentation never selects the fatal
requester, but later genuine physical OOM retires only that requester.
The independent blocked witness, expected WAIT statuses, failed-RUN
and invalid-BRK controls and final process-MM accounting all PASS.

The instrumentation lives exclusively in disposable test images.
Production OOM remains OFF, with no permanent KCore or MRES growth.
Booted direct RUN -3 cause tracing and the P0-D/E cross-product and
final P0-F gates remain open.

## v1114: booted actual MM descriptor saturation and failed RUN

The full 256K PDP-6 v1114 test builds disposable kernels with a
private BRK-sentinel extension; ordinary production BRK dispatch is
unchanged. The test allocates one-word MM_TYPE_KERNEL_DYNAMIC extents
through the real `mm_alloc_aligned_noreclaim` until the live
`mm_extent_count` equals `mm_extent_capacity`. A valid RUN request
for the executable already admitted by v1110 returns -1 under this
descriptor pressure without terminating its parent. The extension
records every physical base, then frees each through `mm_free` with
the original type and owner. Parent process slots and process-memory
words return to baseline.

Both independent physical PDP-6 simulator boots PASS: OOM disabled
`<MRILDNWP`, disposable OOM-enabled `<MRILKWP`. Subsequent ordinary
RUNs, the invalid-BRK children, physical OOM requester and blocked
bystander complete their expected lifecycles. Only true physical
exhaustion retires the requester with the test OOM policy enabled.
The failed RUN's exact internal MM error is not directly observed;
the allocator descriptor table being full before RUN is verified.

The instrumentation and temporary scratch state exist only in the
disposable image, not in production KCore, MRES, data or ABI. Default
OOM remains disabled. Booted BRK fragmentation, explicit failed-RUN
cause tracing, and complete P0-D/E/F acceptance remain open.

## v1113: real MM error through VM BRK and opt-in OOM dispatcher

The source-linked native PDP-6 v1113 test now executes the complete
production allocation-to-BRK failure chain: `mm_find_fit`,
`mm_extent_insert`, `mm_alloc_aligned_noreclaim`,
`mm_alloc_aligned`, `mm_compact`, `vm_space_brk_current` and
`native_sys_brk` with the fatal OOM branch enabled only in the test.
An adjacent extent blocks in-place process growth. Three separate
02000-word gaps supply 06000 total words but not one 04000-word
replacement extent; the real allocator returns fragmented -2 and
the VM/BRK paths preserve it, returning ordinary failure rather than
retiring the requester. With a large fitting gap but a full descriptor
table, the same path preserves real -3. Both failures retain the
process mapping, logical break and original extent count. Three
unsafe source mutations which collapse either error to -1 are rejected.

No-progress FS/swap reclaim, refused relocation and fatal selection
are explicit native fixture observers; they are not booted physical
resource-pressure events. Full-system BRK/RUN, WAIT/PID, bystander,
and P0-D/E acceptance remain open. No production code, KCore memory,
MRES, ABI or OOM policy changes; the default stays disabled.

## v1112: booted nonfatal failed child RUN under both OOM policies

The v1112 extension of the existing 256K PDP-6 requester/bystander
guest submits a structurally valid RUN V2 request naming a nonexistent
absolute executable. Its immediate -1 rejection leaves the parent
alive with the original process-slot and process-memory counts. The
same parent then runs the v1110 below-floor and address-limit BRK
negatives and the established real physical-OOM requester plus blocked
witness. Both separately booted disposable policies PASS:

- Default OOM disabled: `<RILDNWP`; genuine exhaustion also returns -1.
- OOM enabled only in an isolated copy: `<RILKWP`; only the genuinely
  exhausted requester is fatally retired.

Expected WAIT outcomes, bystander survival and recovered process MM
accounting PASS in both images. Ordinary failed RUN never selects a
fatal requester. No production policy, resident kernel data, code,
MRES or ABI changes. This verifies missing-executable rejection, not
yet a RUN failing from actual MM descriptor exhaustion or fragmented
core. Those physical-pressure cases and P0-D/E remain open before
production OOM activation.

## v1111: real MM allocator distinguishes shortage, fragments and slots

The native PDP-6 v1111 regression executes the authoritative
`mm_find_fit`, `mm_extent_insert` and `mm_alloc_aligned_noreclaim`
assembly. Two extents occupy a managed 010000-word arena, leaving
two disjoint 02000-word gaps. An allocation larger than their total
free space returns -1 (insufficient physical RAM). A 03000-word
allocation returns -2 (fragmentation, even though 04000 words are
free overall). A smaller fitting allocation returns -3 when the
extent-descriptor table is full. All refusals preserve the original
extents; increasing only table capacity then permits sorted insertion
of the smaller allocation. Two unsafe error-code collapses to -1 are
rejected by the native fixture.

The independent real OOM-BRK dispatch v1000 test also PASSes for
nonfatal fragmented/invalid causes, distinguishing the allocator's
error classification from fatal requester selection. This is native
allocator evidence only; it does not establish booted BRK error
propagation under descriptor exhaustion or fragmentation. Production
OOM remains disabled, no KCore/MRES/persistent RAM changes, and P0-F
remains open alongside failed RUN and P0-D/E integration.

## v1110: invalid BRK remains nonfatal across both OOM policy modes

The versioned v1110 extension of the 256K PDP-6 v1075 full-system
guest adds two distinct validation errors: BRK below the original
image floor (`1`) and the maximum 18-bit logical address (`0777777`
octal), which cannot be rounded to a legal APR extent. Both calls
return -1, leave the child's existing break unchanged and permit
normal EXIT/WAIT and complete process-MM reclamation.

The guest PASSes in both independently booted configurations. Under
production's disabled OOM setting, subsequent actual physical memory
exhaustion also returns -1 without killing the requester; an independent
blocked witness survives and final memory accounting matches baseline.
Under the enabled OOM policy in a disposable test-only kernel, both
invalid children still finish normally while the genuinely exhausted
requester is the only fatal victim. Witness and MM accounting PASS.
The test's opt-in mode does not alter ordinary v1075 behavior; production
OOM remains disabled. No production KCore, MRES, ABI or data growth.

This covers only invalid floor/address-limit errors against physical
exhaustion under P0-F. Fragmentation, exhausted MM descriptors,
failed child RUN, integrated P0-D/E gates and final OOM enablement
remain OPEN.

## v1109: real VM destroy plus physical MEMFS XIP fault, linked PDP-6

The v1109 native PDP-6 fixture links the unchanged production
`vm_space_destroy` leaf with the direct MEMFS provider gateway,
`memfs_xip_owns_base`, `memfs_xip_unpin_base` and real MM pin
adjustment. An existing owned-XIP reference points to a true MM
extent whose physical pin counter is zero. The real MM unpin rejects
that state with -4; the provider distinguishes owned failure from an
absent/non-XIP service. VM destruction then refuses without attempting
an unrelated MM_TYPE_PROCESS deletion, changing its original mapping
or executing swap detach.

After repinning the *same* extent through production `mm_pin`, a
second VM destroy successfully consumes the MEMFS XIP reference,
performs exactly one physical MM unpin and one backing-detach witness
call, and publishes VM_SPACE_NONE. Two unsafe mutations (treating
owned XIP -4 as ordinary MM fallback, and inverting the detach success
condition) must fail. The foreign filesystem provider's serialized
lease is neither stolen nor waited upon.

Only the resident-mode and swap-detach responses and the forbidden
process-MM fallback are test stubs. This test does not claim genuine
MEMFS executable admission, certified no-write mapping, device
backing or booted WAIT/PID ownership. The expanded native v1096
P0-D release matrix PASSes with the new v1109 test; the optional
booted full matrix was not rerun. `memfs_xip_admit` explicitly
requires trusted no-write certification before executable use; normal
DXR loading cannot be substituted for that certification. The booted
XIP lifecycle remains P0-D OPEN, as do broader P0-D cross-products
and P0-E/F. No production KCore, MM or MRES change; OOM remains OFF.

## v1108: source-linked physical MEMFS XIP MM -4 and retry

The v1108 native PDP-6 regression reuses the production MEMFS XIP
direct dispatch, `memfs_xip_unpin_base` and MM pin adjustment. The
XIP reference is still owned but the corresponding MM extent's pin
count is zero. The real `mm_unpin` rejects it with -4; MEMFS retains
the XIP reference, and the slot-0 direct gateway leaves the foreign
serialized provider lease untouched without clock waiting.

Production `mm_pin` repairs the same extent. A second direct XIP
release consumes the reference and unpins successfully; a third is
correctly reported as not owned. Original v1086/v1094 source-linked
tests and v1094 negative mutants PASS. Two new v1108 negative controls
also reject the unsafe variants which erase the reference on a failed
physical unpin or retain it after successful unpin. The expanded v1096 native P0-D
release-stage matrix PASSes with v1108. Production KCore, MRES and
persistent state are unchanged. This is not a booted process-VM XIP
failure test; that P0-D integration, P0-E/F and OOM enablement remain
open.

## v1107: live PREVM run-queue alias and allocated DISK bitmap retry

The slot-0 `proc_finish_service_v881` scanner previously tested
scheduler RH bit `02` as the unpublished-SIDL discard-VM marker before
decoding the process state. For ordinary live processes the same RH
field stores a run-queue successor. A PREVM+FINALIZE process whose
successor was slot 2 was skipped indefinitely instead of retrying
its checked VM/swap backing release. Production now verifies the SIDL
state first; only SIDL may interpret this low bit as a discard marker.
The source-linked PDP-6 test executes both a live PREVM owner with
runq successor 2 and an actual SIDL rollback. It PASSes; reverting the
check order fails the unsafe-mutation control.

The 256K booted v1107 test uses the existing two-member session/TTY
HUP guest. Its special INIT enables actual SWAP through the privileged
STORAGECTL syscall. An isolated test hook selects the second member
through the KCore `proc_swap_reclaim` gateway, letting the installed
SWAP MRES perform its normal swap-out transaction and allocate a real
16-block DISK extent beginning at legal block zero. The first remote
HUP delivers the default-fatal event to this member, then calls the
real `proc_swap_detach`/backstore verifier. The test-only hook clears
one bit from the actually allocated bitmap run. The first verifier
rejects the missing bit, so the process retains its original swap
record and deferred PREVM/FINALIZE ownership. WAIT_NOHANG cannot
prematurely reap the session leader.

The slot-0 reaper retries the marked member without delivering HUP
again. The test restores the original bitmap word on the second
verifier call. Production backing-store release frees the exact run
once and clears the final DISK record; the allocation count returns
to baseline. The first member's physically pinned u-area is unpinned
at successful session commit. All three leader/member HUP/WAIT
statuses, final TTY handoff and no-duplicate-HUP checks PASS.

Booted counters: one SWAP MRES swap-out, two bitmap-verifier calls,
one test-only bitmap corruption/restoration, one real backing free,
five reaper scans, one first-member u-area pin/unpin, no duplicate
HUP and zero ownership errors. Original v907, PIPE v1104, DPY v1105
and resident-VM v1106 booted compatibility tests PASS. The source-linked
native P0-D matrix PASSes. The full production PDP-6 KCC image builds;
`__kcore_low_end` remains `023171` octal. There is no new resident
state, no new MRES dependency, and production requester OOM remains
disabled.

This completes the allocated-DISK *bitmap ownership verification*
fault leaf, not physical disk-I/O failure injection. A genuine
MEMFS-owned XIP lifecycle and broader multi-resource, WAIT/PID reuse
and long-duration leak/liveness acceptance remain OPEN in P0-D.
P0-E and P0-F also remain OPEN.

## v1106: booted physical resident VM extent MM_BUSY and slot-0 retry

The v1106 disposable 256K PDP-6 guest is a stricter physical variant
of the original v997 owner/provider/WAIT fixture. It preserves the
foreign in-flight filesystem lease, two private-stack FILE close
refusals, one u-area MM_BUSY, a continuously runnable bystander and
CHLD/EXIT finalization. Instead of synthesizing an early failure at
`vm_space_destroy`, the test first pins the actual slot-5 process VM
extent (MM_TYPE_PROCESS, owner 5, base `0752000` octal). The unchanged
VM destroy follows its normal MEMFS-XIP classification and real MM
path. Production `mm_free` returns MM_BUSY (-5), leaving the original
VM base and PREVM+FINALIZE owner live for a safe slot-0 retry.

On that retry the isolated hook unpins precisely the same extent;
ordinary `mm_free` verifies its type/owner, deletes it exactly once,
then `proc_swap_detach` runs once and `VM_SPACE_NONE` commits. This
resident process has no DISK backing allocation. The swap-detach
execution proves that the VM release stage does not bypass backing
cleanup on success, but does not validate a refused physical disk-swap
release; that remains a separate P0-D gate.

Booted v1106 PASS: 2 VM calls, 2 real process-MM free attempts,
1 physical pin/unpin, 1 MM_BUSY, 1 successful MM deletion, 1 actual
swap-detach-stage execution, 2 u-area frees with 1 MM_BUSY, 3 FILE
closes with 2 refusals, 36 independent bystander cycles and no
ownership errors. Original booted v997 and v1103 multistage guests
both rerun PASS, as does the native P0-D release matrix. All new fault
injection/counters live only in daimos-testkit's disposable image;
production KCore end remains `023171` octal, with no extra BSS/MRES
and the requester OOM policy disabled. MEMFS-owned XIP and allocated
disk-swap backing fault integration remain open, alongside P0-E/F.

## v1105: real HUP / active DPY Type-340 raw-list MM refusal

The v1105 booted 256K PDP-6 test adds a genuine Type-340 display-list
owner to the v907 two-member physical session/TTY HUP fixture. The first
member has already received HUP and published VM_NONE+FINALIZE but still
owns a pinned stable u-area pending slot-0 retry. The second member has
opened `/DEV/DPY0` and published a 32-word raw program using the real
PI6-clock/PI7-BLKO driver before its remote HUP.

The production FILE DPY close path calls `dpy_write_words` with zero
words, quiesces the Type-340, and uses the ordinary `mm_free` for the
existing list. In the test-only isolated MM implementation, the exact
MM_TYPE_KERNEL_DYNAMIC / DPY owner 014 extent is physically pinned on
its first attempted free, causing the original allocator to return
MM_BUSY (-5). The FILE descriptor and published raw-list/IOWD remain
owned and valid. The first leader HUP therefore fails, and its parent
sees no completed WAIT while one member's HUP is still unresolved.

Eight genuine PI6 raw-list refreshes occur during a delay between the
first HUP and unpin/retry, demonstrating that failed DPY STOP does not
strand the display program. The second HUP skips the first member's
irreversible finalization, unpins the exact DPY extent, and frees it
once through the original MM validation. The session TTY commits and
releases the first member's u-area pin, then existing WAIT/TTY checks
confirm the leader and members' HUP statuses and proper handoff.

Booted guest counters PASS: 2 DPY frees, 1 MM_BUSY, 1 successful free,
1 physical DPY pin/unpin, 14 total raw PI6 refreshes, 8 after refusal,
0 repeated HUP deliveries and 0 MM/owner errors. The original booted v907
session HUP guest, v1104 physical PIPE/HUP guest and the complete
v1096 source-linked native P0-D matrix all independently PASS.
All new hooks and counters exist only in daimos-testkit's isolated
guest. The unchanged production KCore endpoint
is `023171` octal, with OOM requester termination still disabled.
P0-D/E/F remain OPEN, particularly integrated MEMFS-XIP/VM/swap
failures and broader WAIT/PID/leak cross-products.

## v1104: real member HUP concurrent with physical PIPE and u-area pins

The disposable 256K PDP-6 v1104 guest extends the real two-member
session HUP/TTY transaction from v907. Each member now owns a private
anonymous PIPE, and actual `file_close_all_at_v865` attempts to retire
those FILE descriptors during remote HUP delivery. The first member
has a physically pinned stable u-area; after its HUP it has committed
VM_NONE+FINALIZE but retains its original slot-0 finalizer owner.

On the second member's first final PIPE close, a narrowly targeted
test-only hook physically pins that PIPE extent. The original `mm_free`
therefore returns MM_ERR_BUSY (-5) without decrementing the last
writer reference, clearing the FILE descriptor or freeing the extent.
The session leader's first HUP reports incomplete; the original parent
checks that the leader cannot yet be reaped by nonblocking WAIT.

The second leader HUP skips the first member's irreversible finalizer
and retries the still-owned second-member PIPE. The test unpins the
same exact MM extent; the production allocator validates and frees it
once. The real HUP delivery finishes, its TTY session commit unpins
the first member's u-area, and the existing guest verifies HUP status,
WAIT and TTY handoff. Counters: two physical PIPE frees attempted,
one physical pin/unpin, one -5 busy rejection, one committed free;
zero repeated HUP for the committed member, zero ownership errors.

The full native P0-D component matrix and the unmodified original v907
booted physical-HUP scenario also pass. All injection and
observability state is confined to the testkit's isolated guest;
production KCORE end remains `023171` octal and OOM stays disabled.
This verifies real session-member HUP combined with a physical PIPE
MM refusal, not simultaneous DPY, XIP/VM or swap release combinations.
P0-D/E/F remain open.

## v1103: one booted fatal owner across PIPE/FILE/HUP/VM/MM stages

The optional daimos-testkit v1103 guest adds an integrated 256K PDP-6
teardown sequence to the existing v997 held-provider/bystander fixture.
The selected child retains a real anonymous PIPE until fatal FILE
cleanup. Two test-only FILE close refusals first retain its private
kernel continuation and descriptors. Next, `pipe_close_ref` reaches
the production MM allocator: the first final free returns MM_BUSY
(-5), preserving both the PIPE descriptor and exact pre-free reader/
writer count; the next succeeds and commits its final release.

In that same guest, a *test-only* refusal of the session-teardown
entry requires one more private-stack HUP-stage pass before VM release.
It is not a physical member HUP; the independent booted v907 physical
session/TTY test covers successful and rejected HUP deliveries. After
the HUP stage succeeds, the v997 synthetic first VM refusal retains
the original PREVM owner, and a real MM_BUSY (-5) refusal on final
u-area free retains the same process until the slot-0 retry commits.
The PREVM/post-VM markers prevent HUP from running twice after it has
completed. The foreign provider's original live lease remains intact.

Booted v1103 PASS: FILE attempts 3/refusals 2, PIPE MM frees 2/
refusals 1, HUP-stage attempts 2/refusals 1, VM attempts 2, stable
u-area attempts 2/refusals 1, one EXIT/CHLD/finalization, 36
independent bystander cycles and no ownership errors. The original
v997 scenario and source-linked component tests remain independent.
Both the unmodified v997 booted baseline (v1103 option disabled) and
the complete v1096 native P0-D stage matrix were rerun and PASS.

All new hooks and counters live only in disposable test sources.
Production DAIMOS source, memory layout, MRES and OOM-disable flag
are unchanged from v1102 (`__kcore_low_end` `023171` octal).
P0-D remains open for simultaneous *physical* HUP, PIPE/DPY, XIP
and swap faults plus further WAIT/PID invariants; P0-E/F remain open.

## v1101-v1102: permanent invalid u-area owner and bounded MM retries

An isolated booted PDP-6 v1101 guest deliberately changes exactly
one live u-area MM extent owner (slot 2 owner 01002 -> wrong owner
01003) after it reaches fatal release. The real `mm_free` then returns
MM_ERR_INVAL (-4), rather than the MM_ERR_BUSY (-5) used by the old
v909 permanent-pin guest. The production read-only `GETTEARDOWN`
reports stage MM and WRONG_OWNER sentinel 0402, while the original
process descriptor, u-area and unreaped WAIT identity remain owned.
An independent process continues running; no force-free or ownership
override is permitted.

The first v1101 guest found a real reaper throttling regression:
41 invalid frees and 40 idle-reaper scans during the intended bounded
test. The error is the interaction with v1098, not the new test's
owner corruption. `proc_finish_mark_pending_v892` was intentionally
changed in v1098 to arm a first delay for an already prepublished
FINALIZE+PREVM owner without an active clock. However, the idle loop
still cleared the global pending bit immediately before scanning,
so *every* refused -4/-5 release looked like that unarmed first error
and restarted the backoff at 020 octal ticks.

The v1102 correction retains the existing shared pending bit during
the entire `proc_finish_service_v881` scan. It uses saved AC11's
second bit to note any failed owner, then clears the pending bit only
if no cleanup attempt failed. The historical completion/last-user
return is restored before leaving. The existing backoff routine now
doubles delays from 020 to 040 to 0100 up to 01000 octal clock ticks
without adding per-process fields, global data, MRES or new tables.

Native source-linked PDP-6 v1102 proves the first two retry delays,
their increase and successful retirement, with four unsafe mutations
rejected (including restoration of the original premature idle clear).
Native v881/v1098 and full P0-D source-linked release matrix PASS.
After the fix, booted v1101 wrong-owner and booted v909 permanent-pin
guests both PASS, observing seven rejected frees and six bounded
reaper scans, correct diagnostics, retained WAIT ownership and live
bystanders. The full 256K KCC production image PASSes, with KCore
end `023162` -> `023171` octal (+7 code words), no additional BSS.

P0-D still needs simultaneous HUP/PIPE/DPY/VM/XIP/swap/u-area faults
and additional PID/WAIT controls; P0-E/F remain open. Production
automatic OOM termination is disabled.

## v1100: preserve typed DPY replacement rollback refusal

During a Type-340 display-list replacement, the old published list's
`mm_free` can reject due to a busy pin (-5) or invalid type/owner
(-4). Although rollback of the newly allocated replacement kept the
old list, IOWD, refresh state and FILE descriptor intact, that branch
discarded the old MM error and returned generic -1. The error now
survives the second (new-list) MM free in the existing stack-local
word, and returns unchanged on successful rollback. A failure of
that second free still retains the separately owned unpublished list
through `dpy_list_orphan_v912`, without any new owner field or BSS.

Source-linked native PDP-6 v1100 tests both error types and rejects
two mutations that discard the first error, alongside v874 normal
replacement failure/recovery. The P0-D component release matrix and
256K KCC full image PASS; KCore remains `023162` octal with no BSS
growth. This leaf fix does not prove all simultaneous HUP, PIPE, DPY,
VM/XIP, swap backing and u-area faults in a booted process. P0-D/E/F
remain open and requester OOM termination is still production-OFF.

## v1099: typed PIPE/FIFO final-close stage

P0-D now propagates the actual allocator error on a final PIPE/FIFO
close: -5 means a pinned extent (busy), while -4 means an invalid MM
type or owner. Previously both returned -1 despite correctly retaining
the descriptor, counts and named FIFO list. The named list reinsertion
now uses AC3, so its original `mm_free` status survives in AC1 through
`file_close_entry_v865` and the fatal FILE owner diagnostic.

Failure cannot clear a FILE, decrement the final PIPE reference,
discard the FIFO list owner, publish an EXIT or free a foreign extent.
The original successful close path and owner-side retry remain intact.
Native source-linked PDP-6 v1099 verifies both typed failures,
subsequent recovery, exactly-once free, and four rejected unsafe
variants. Updated v1096 component release matrix and v1087 allocation
closure with eleven unsafe source variants PASS. The full 256K KCC
image has unchanged KCORE endpoint `023162` octal and no new permanent
data or process fields. P0-D/E/F are OPEN; production OOM kill OFF.

## v1096-v1098: typed u-area errors and first PREVM retry-clock repair

P0-D now preserves the exact allocator refusal at the final u-area
release boundary. `proc_uarea_release` returns zero only when the
stable u-area was successfully freed or is genuinely absent, -5
(`MM_ERR_BUSY`) if the original extent is pinned, and -4
(`MM_ERR_INVAL`) for bad type, owner, missing extent or a descriptor
whose u-area flag is still set but whose base has become zero. It
does not clear its process flag/base, free another owner's extent,
publish FREE/ZOMB or abandon its retry continuation after a refusal.
Existing `proc_finish_slot` and `proc_slot_discard` interpret any
nonzero return as incomplete and retain the original owner for a
later retry. A permanent invalid-owner result remains quarantined;
it must not be misreported as a successfully released u-area or as
a mere transient pin. `GETTEARDOWN` MM details expose the current
physical extent ownership, not a past `mm_free` error history.

Native PDP-6 v1097 executes the real `proc_uarea_release` and MM
leaves: wrong type and owner produce -4, a pin produces -5, an
inconsistent zero-base/owned descriptor produces -4, a repaired
extent frees exactly once, and a repeated release does nothing.
Three unsafe mutations (generic error collapse, clearing before
successful free, wrong invalid-base status) are rejected.

The expanded v1096 component matrix verifies the preceding PIPE
last-close and DPY replacement rollback, VFS/HUP committed child
handling, TTY PREVM and unpublished RUN rollback, VM/XIP and disk
backing detach, exact u-area free, and the transitive allocation-
free fatal graph. It does not by itself prove all possible cross-
product faults in a single guest. P0-D remains open for those cases.

The combined full-system acceptance exposed a separate **real**
preexisting progress bug, now fixed as v1098. As part of the safe
PI6-masked EXIT stack handoff (v1086), `proc_exit_current` publishes
the process's FINALIZE+PREVM owner marker *before* destruction. This
marks ownership but does not arm a clock. On its first refused
`vm_space_destroy`, the old `proc_finish_mark_pending_v892` mistook
that prepublished marker for evidence that retry timing was already
initialized. It set the shared pending bit without a valid delay;
the idle reaper correctly rejected the zero delay and never retried.
The guest exhibited one VM attempt, zero u-area attempts, correct
CHLD and hundreds of independent bystander cycles, then timed out.

v1098 tests both the per-owner marker and the shared pending-clock
state. A premarked EXIT owner without an armed shared timer now starts
the existing 020-octal (16-tick) first retry using the current APR
clock epoch. A genuinely pending owner keeps its existing exponential
backoff; a newly marked owner can reset the first delay. The
zero-delay rejection remains intact, and no process-private stack,
PI6 exclusion rule or MM ownership changes. The existing timer word
is reused without additional permanent state.

Source-linked PDP-6 v1098 tests the prepublished first failure,
existing capped backoff, a new owner and a stale prior epoch;
three unsafe mutations are rejected. The previously failing **booted
v997 256K PDP-6 regression now PASSes**: two VM attempts, two final
u-area attempts including a refused MM_BUSY, three FILE close calls
with two refusals, a correct CHLD and owner EXIT, an intact foreign
provider lease and 36 independent bystander cycles without an
ownership or finalization error. The v1077 disposable physical-OOM
guest also PASSes with two D6FS SYNC refusals, one MM_BUSY, correct
requester recovery and a surviving independent process. The booted
injection is restricted to a disposable copy; the production OOM
option remains OFF.

Matched KCC KCORE low-end measurements are `023154` octal at v1094,
`023156` after typed u-area errors (+2 code words), and `023162`
after the retry-clock fix (+4 further code words). There is no new
resident owner table, MM descriptor, MRES state, timer word, syscall
or process ABI. P0-B containment is closed; P0-D/E/F remain OPEN.

## v1093-v1094: P0-D release-stage error typing and VM/XIP ownership

The checked canonical TTY line reset now propagates the exact physical
MM_FREE result: -5 for a pinned busy extent, -4 for invalid type,
owner or missing extent. Neither error clears the per-TTY base or
permits a session change; only a successful release commits the free.
The old void reset uses the same checked implementation without a
second MM release path. Native tests and the v898 booted pinned-TTY
release/retry guest PASS, including two rejected unsafe mutations.

MEMFS XIP release now reserves -1 for the absent-provider fallback and
returns -4 for an *owned* unpin refusal. `vm_space_destroy` passes
through to MM_TYPE_PROCESS free only for a provider that returned 0
(not XIP) or -1 (unloaded). An owned negative release prevents process
MM free, swap detach, and VM_NONE publication; the original VM/XIP
owner remains available for the existing PREVM retry. Native source-
linked v1094 VM/MEMFS tests with four unsafe variants PASS, as do the
previous swap/PI6/finalizer/transitive allocation-graph regressions.
The booted v1077 disposable 256K physical-OOM fault guest also PASSes
against the updated production sources: two D6FS SYNC refusals, one
MM_BUSY refusal, correct requester exit and independent witness survival.

The full 256K KCC production image builds with KCORE end `023154`
octal versus v1092 `023161`, saving five resident words without new
fixed memory, MRES state or process fields. P0-D is not closed: the
integrated HUP, PIPE/DPY, backing, TTY and final u-area fault-stage
matrix remains open. Production automatic OOM termination stays OFF.

## v1092: P0-B fail-closed provider lifetime contract CLOSED

P0-B is closed **for memory and resource ownership safety**, not for
eventual completion. The accepted response to a provider that never
returns or permanently rejects CLOSE is **fail-closed quarantine**.
This is not a new MRES module, process table, or discretionary killer:
it uses the exact provider, FILE, process and kernel-stack owners
already present in KCore. Unknown provider completion cannot be
inferred from elapsed time or a capped retry interval.

The shared DTFS/D6FS/TSFS register gateway retains the encoded owner
slot+1 in the LH of `fs_provider_ready` while its RH is zero. Its
shared block workspace and owner's stable private kernel continuation
remain live until the real provider returns and releases that lease.
Another process waiting before provider dispatch may respond to a
fatal event on its own saved stack (v1090/v1091); it cannot terminate
or unlock the provider owner. Ordinary blocked callers may have no
completion until service resumes. An in-flight owner must never have
its PID reused, FILE/MM buffers freed, event fabricated or lock reset.

A provider that *returns an error* during fatal FILE closure is a
different case. The original fatal owner retains its descriptor,
process/u-area, provider error witness and private retry continuation.
Backoff reaches 01000 octal ticks, then stays bounded in frequency;
it does NOT guarantee eventual reclamation. The existing read-only
`SYS_PROCCTL_GETTEARDOWN` and `PS` fields expose stage 1 FILE and the
actual ERR class, while MM stage diagnostics belong to P0-D.

Administrative diagnosis may inspect the `fs_provider_ready` address
from the KCC `kcore.map` using a read-only simulator console EXAMINE.
RH nonzero means the gateway is free. RH=0 and nonzero LH means owner
slot = LH - 1; an owner with a live continuation must not be deleted.
Do not use DEPOSIT to release the lease, manually signal its event,
free its MM extent or recycle its slot. Repair a faulty media/provider
subsystem and let its original caller return, or perform a controlled
system restart if no positive quiescence proof exists. Any future
online recovery needs a specific provider's verified completion and
resource-lifetime protocol. A reproducible infinite loop is a bug
to fix in its originating provider, not a reason to weaken ownership.

daimos-testkit v1092 enforces the exact owner publication, foreign and
recursive refusal, pre-dispatch fatal wake, private FILE retry, and
absence of premature allocation/exit or forced lock release. It PASSes
and rejects 14 unsafe source mutations; the v954 and v1087 owner/
no-allocation contracts plus native v1088/v1090 also PASS. Full-system
PDP-6 v891 tests 11 permanently failed -5 CLOSE returns, held FILE/
error owner, capped retry and an independent running process, with
zero premature exits; v953 tests the same eleven errors followed by
a successful twelfth close and exactly one final EXIT. The v1091
booted nonreturning provider owner and fatal pre-dispatch waiter test
also PASSes. All these contracts use unchanged production routines.
No DAIMOS code, resident data, MRES or ABI modification was necessary
at v1092; KCORE low-end remains `023161` octal.

P0-D/E/F are still required before an OOM production enable decision.
Requester-only automatic OOM killing remains OFF, because a genuinely
quarantined requester cannot be promised bounded memory reclamation.

The complementary booted v914 256K PDP-6 permanent invalid-provider
case also PASSes: eleven returned -4 CLOSE refusals, eleven private
owner sleeps, intact FILE/error witness and an independent bystander;
zero premature FD clears or exits. This is a returned invalid result,
not a recoverable MM_BUSY result and not permission to force-free.

## v1090-v1091: fatal event aborts an undispatched FS lease waiter

P0-B now distinguishes two live private-stack owners. A process already
inside the serialized DTFS/D6FS/TSFS provider retains the lease and
cannot safely be canceled from another process. A different process
sleeping on the gateway's lease event has not entered the provider and
can safely abandon its saved request on its own stack.

After a remote INT/TERM/HUP/PIPE is recorded, the event path wakes only
a target in exact SLEEP/WAIT_EVENT state with the existing FS gateway
wait-event address. STOP, WAIT_CHILD, WAIT_INTR and unrelated kernel
waiters remain untouched. The resumed waiter checks its existing u-area
fatal event bitmap and refuses the request before acquiring a lease;
the normal user-return fatal handler owns resource cleanup and WAIT
notification. An already established fatal FILE-close owner bypasses
this refusal so it can retry a free provider. No foreign stack, shared
filesystem workspace, or active provider lock is freed or unlocked.

Native PDP-6 source-linked v1090 verifies the exact wake and refusal,
other wait classes, surviving lock owner, cleanup-owner reentry and
four deliberately unsafe variants. Booted v1091 disposable 256K PDP-6
holds a provider owner indefinitely, then sends TERM to a blocked
gateway waiter; the waiter exits and is reaped with correct WAIT status
while owner lease `4,,0` is unchanged, an independent user runs for
73 cycles, and no errors are reported. Updated v1087 no-allocation
closure and production KCC image build PASS. KCORE low end grows from
`023127` to `023161` octal (+26 words), without any persistent data.
The independent booted 256K v1077 physical-OOM requester/D6FS/MM
fault guest also PASSes with two D6FS SYNC and one MM_BUSY refusals;
the unrelated witness survives and production OOM remains disabled.

This resolves *pre-dispatch* waiter termination, not recovery from
arbitrary provider software loops or permanent FILE/MM refusal. Such
owners must remain quarantined until genuine provider-specific safe
completion is established. P0-B/D/E/F still open and production OOM
killing remains OFF.

## v1088-v1089: P0-B provider-lifetime progress, not final closure

PDP-6 native v1088 tests the actual serialized FS register gateway with
an executing provider frame. Both self-recursion and fatal foreign
contention fail with -5 without borrowing the shared buffer or forcing
unlock; only the real provider return releases its lease and wakes
waiters. Two unsafe blocking mutations are rejected.

The Type-136 DTC/MTC runtime transfer previously had a missing-PI
liveness gap: after DECtape search or MTC command setup, data transfer
could wait indefinitely even while APR ticks continued. The v1089 tape
MRES now bounds these transfers with an existing-word 60-Hz watchdog
(DTC 300 ticks, MTC 5400 ticks), stops a stalled controller, and wakes
the original owner with an I/O error. The FS lease is still released
only by the returning owner. Native/online tape tests, full KCC build
and full-media INIT/LOGIN boot checkpoint pass. No new permanent data.
The matched tape MRES resident body grows by 30 words (375 -> 405),
plus one KCORE clock-call word; BSS and process metadata do not grow.

P0-B remains open for arbitrary software loops and permanently rejecting
providers. Neither a finite driver timeout nor the private FILE-close
retry interval authorizes killing or freeing a live provider owner.
P0-D/E/F remain open; production requester-only OOM termination is OFF.

## v1087: P0-A CLOSED -- transitive fatal-release graph

The v1087 testkit source-linked audit completes the bounded fatal
teardown call graph in the current PDP-6 kernel. It follows user BRK
classification, APR and post-commit EXEC fatal entry, normal EXIT,
self/foreign fatal event delivery, and deferred HUP through private
`file_close_all`/`file_close_all_at_v865`, `file_close_entry_v865`,
VFS SYNC, PIPE/FIFO and DPY STOP, then session HUP, checked TTY release,
VM/MEMFS-XIP/MM detach, swap/backstore bitmap free, u-area release,
process table ownership and final CHLD/WAIT notification.

No new MM/backstore allocation is reachable in this existing graph:
the only VFS SYNC providers enabled for FILE close are DTFS (vnode
validation) and D6FS (no-op), with other providers bypassed. The
serialized register-provider bridge keeps a process-private stack
through sleeps, or returns BUSY to a fatal caller behind a foreign
lease; it never forcibly releases that lease. An unsuccessful CLOSE
preserves the actual FILE descriptor, an unsuccessful member HUP or
TTY MM free preserves the leader's session owner, and unsuccessful
VM/XIP/TTY/U-area retirement keeps the explicit PREVM/post-VM or SIDL
retry owner. Last-ref PIPE and DPY MM failures keep their FILE owners.
Committed HUP members in TRANSITION+VM_NONE+FINALIZE are not delivered
twice, while ordinary live/unfinished members must still be retried.

The source contract checks direct edges, no new allocations, guards
before destructive steps and existing retry handoffs. Eleven negative
mutations are rejected (FILE last clear, HUP allocation, remote-close
error inversion, VFS/D6FS allocations, PIPE/DPY allocation, TTY/VM
failure inversion, failed RUN discard and private owner backoff).
Native PDP-6 v870/v904/v912/v957/v905/v922/v995/v1078/v1081/v1085
PASS, and the v1077 256K guest with combined D6FS/MM injection PASS.
The existing v922 test fixture gained the zero-return TTY line-reset
stub required by the checked v897 ABI; production code was untouched.
No KCORE or MRES change, no extra permanent RAM: KCORE low end remains
`023126` octal.

**P0-A is closed for the actual current graph, not future arbitrary
provider implementations.** The proof of how an already in-flight
foreign provider terminates or remains safely owned if it never
returns is explicitly P0-B. Detailed exact-once fault-stage tests are
P0-D, mixed complete-system acceptance P0-E, production enable P0-F.
The default OOM gate remains OFF.

## v1086: P0-C CLOSED -- PI6-masked slot-0 finalization

The v1086 reachability audit found a real duplicate session HUP under
disabled scheduler PI6. An ordinary EXIT or the APR/EXEC fatal-owner
retry first completed FILE close and HUP on its private executive
stack. It then entered `proc_exit_current`, switched to the permanent
idle stack and inhibited PI6. Without an explicit completion marker,
`proc_exit_finish` called `proc_finish_slot_owner_v932`, which could
execute `proc_session_teardown` a second time. A foreign member's HUP
may close another FILE and sleep on a provider while PI6 is inhibited.
Immediate self-delivered fatal events also bypassed the private HUP
preflight and entered the idle-stack exit directly.

v1086 reuses the existing transition and PREVM+FINALIZE marker:
`proc_exit_current` stores the exit status in the marker and sets
TRANSITION under disabled global PI *before* switching stacks. The
marked `proc_finish_slot_owner_v932` then dispatches directly to the
non-sleeping PREVM or post-VM stages and never reenters HUP. Immediate
self-fatal event delivery is routed into the existing process-owned
FILE/HUP retry path. No new resident owner record or timer is needed.

P0-C covers both PI6-masked slot-0 entries. The periodic idle retry
only visits marked PREVM/post-VM EXIT or SIDL RUN rollback. Its
qualified-clock backoff runs before PI6 inhibition, and disk-backed
swap-in is dispatched only after PI6 is restored. The only VM provider
operation reachable from either teardown is MEMFS XIP release, using
provider 4 and opcode 026. It bypasses the serialized DTFS/D6FS/TSFS
lock entirely: MEMFS's dispatcher checks XIP ownership and performs
an MM unpin. Actual TTY canonical cleanup is a checked MM free, and
swap detach only verifies/updates the in-memory backing bitmap.
Parent adoption, notification and runnable-queue edits are resident
table operations. No reachable callback in these marked paths sleeps
on a process stack, performs device I/O or waits for PI6 progress.

Executable daimos-testkit v1086 contract verifies all these edges and
rejects seven negative code mutations (masked swap, gateway route,
sleeping XIP/TTY, lost premark, direct self-fatal exit, swap I/O).
Source-linked PDP-6 tests cover the actual exit handoff plus old-code
negative result, the marked `proc_finish_slot` with a HUP-refusal
trap, and actual MEMFS reg dispatch/XIP/MM unpin with a foreign-held
serialized lease (which remains owned). Full 256K physical OOM
requester/provider/MM v1077, process-events and session-teardown
guests PASS; native v970/v974/v995 fatal/HUP, v926 owner return and
v993 EXIT preflight also PASS.

Matched KCC KCORE low end is `023117` -> `023126` octal (+7 words);
permanent data, MRES vectors and ABI are unchanged. P0-C is CLOSED.
P0-A transitive allocation/owner safety, P0-B in-flight provider
policy, P0-D error-stage idempotence, and P0-E/F full activation
coverage remain OPEN. Hardware APR clock failure is out of scope.
Production OOM termination stays disabled.

## v1085: SIDL RUN rollback releases final TTY before VM retirement

An unpublished RUN child in SIDL/TRANSITION can outlive its parent's
membership in the inherited session while its VM rollback is pending.
The old `proc_slot_discard` first destroyed its VM and then called
`proc_tty_release_session` without inspecting the checked MM result.
A final pinned/misowned canonical input buffer could therefore keep
its TTY session but lose the only remaining retry process and u-area.

v1085 moves the checked TTY call ahead of VM destruction. A refusal
arms the existing SIDL VM retry marker/backoff and leaves the child's
VM, u-area, and session ownership intact. Only on TTY success may VM
and u-area retirement proceed. Already committed post-VM retries do
not reenter the TTY stage. Source-linked native PDP-6 positive and
pre-v1085 negative tests PASS; v892/v894 RUN rollback and v1081 final
TTY-owner finalizer regressions PASS. The full 256K v1077 physical OOM
with D6FS/MM refusals and v969 actual MM-pinned final-session TTY exit
also PASS. Comparable KCORE end increases by just one instruction
word (`023116` -> `023117` octal), zero new persistent words.

## Current production OOM activation gates after v1085

G1 - TRANSITIVE RECLAIM: CLOSED by v1087 for current BRK, APR, EXEC,
normal EXIT and self/foreign HUP source graphs. All required closure,
MRES SYNC and error-owner edges are checked by an executable source
contract with native PDP-6 fault/retry regressions. Already in-flight
foreign provider completion is reserved for G2, not inferred here.

G2 - PROVIDER OWNER: Finish foreign held-lease and in-flight completion
proof. A returning provider may fail; its owner and FILE descriptor
must remain available for retry. A call that never returns cannot be
unlocked, canceled or have its private stack reclaimed without a
provider-specific completion/quiescence contract. Define explicit
fail-closed quarantine versus demonstrable subsystem recovery. A
capped delay between retries does not bound total teardown time.

G3 - PI6/IDLE SAFETY: CLOSED by v1086. The real duplicate-HUP path
was repaired by an explicit PREVM handoff marker, and all direct and
transitive slot-0 teardown callbacks were audited with negative
mutation tests. The only MRES access is MEMFS XIP direct/unpin;
disk-backed swap-in is outside the PI6 mask. Hardware clock failure
is not part of OOM recovery.

G4 - ERROR/OWNER ATOMICITY: Verify idempotent retries and exact-once
commit at HUP, TTY, XIP/VM, disk swap, PIPE/DPY, and u-area boundaries.
An MM pin/BUSY may later recover; a wrong type, wrong owner or missing
extent must not be force-freed or reported as a successful rollback.
No live continuation, lock, FILE, PID or MM extent is retired early.

G5 - SYSTEM ACCEPTANCE: Booted PDP-6 fault combinations with actual
memory exhaustion, HUP and FILE/MM refusals, APR/EXEC fatal events,
RUN admission rollback, physical TTY/XIP pins, shared-provider lock
contention, and sustained independent runnable processes. Verify
correct WAIT status, no unrelated kill, no leaked owner, and progress
without PI6 deadlock. The v1075/v1077/v969 guests and many isolated
leaf tests already pass but do not prove all combinations.

G6 - ACTIVATION GATE: Preserve ordinary failures for invalid BRK,
address limits, fragmentation, MM descriptor shortage, and failed
child RUN. Only the *requester's* genuine MM_ERR_NOMEM BRK enters the
OOM fatal path. After G1-G5, rerun disabled/enabled full-guest negative
and positive tests and check KCORE/MRES/16K resource limits before
changing `PROC_OOM_BRK_ENABLED_V1000` from zero in production.

HARDWARE ASSUMPTION: a working APR hardware line clock and normal
interrupt delivery are required for any scheduled system. Physical
clock failure is outside DAIMOS OOM recovery scope and does not need
a fallback timer. A *software*-inhibited PI6 clock, a spinning kernel
loop or a blocked clock-dependent driver remains a correctness defect.
The v1079 DRM236 timeout needs valid APR ticks and is already tested
for device-specific I/O failure; a universal provider cancellation
system is not implied by hardware-clock failure. Production automatic
OOM termination remains disabled until G1-G6 are accepted.

## v1084: interrupted FIFO OPEN preserves the last PIPE/MM owner

The `pipe_fifo_open` rendezvous is interruptible. Before sleeping,
`file_open` has already allocated a provisional FILE slot, while
`pipe_fifo_open` has incremented its reader or writer reference.
On interrupt, `pipe_close_ref` rolls that provisional reference back.
The final reference cannot be discarded if `mm_free` refuses a pinned
or misowned PIPE extent: the object and original count must survive.
Before v1084 the interrupt branch ignored the failure and returned
zero as though the rollback succeeded. `file_open` therefore cleared
its provisional descriptor, orphaning an extant MM allocation.

v1084 replaces that unconditional jump with one conditional `JUMPE`.
A successful rollback still makes `file_open` return -1; a refused
rollback instead returns the live PIPE vnode, letting `file_open`
publish its already-reserved FD and return success. This treats an
irreversible resource acquisition as accepted work, preserving a real
user-visible owner able to CLOSE/retry. No new owner record, timeout,
collector, or permanent RAM is necessary.

Real PDP-6 source-linked acceptance covers FILE OPEN/NEWFD and close,
PIPE FIFO rendezvous/last-close, and actual MM pin, failed free, unpin
and successful final free. An unpinned interrupted OPEN still fails
and frees the provisional allocation. The pre-v1084 source exhibits
the specific lost-FD/still-pinned-MM mismatch in the negative control.
The full named-FIFO simulator regression passes. Matched KCC KCORE
low end stays `023116` octal; MRES and permanent data are unchanged.
The v1077 256K physical-OOM requester guest also PASSes D6FS SYNC
and MM_BUSY refusal/retry with an independent process witness. The
old wordfifo-v2 user test now reflects v901's one-word character PIPE
semantics, and its full simulator regression PASSes.
The native v901 character stream and the separate full v902 anonymous
PIPE simulator guest also PASS; the testkit Make target for anonymous
PIPE now runs the actual v902 test after its word-FIFO prerequisite.
The pin is injected by the test; this does not establish that normal
FIFO usage pins its unpublished object. Production OOM termination
remains disabled pending the remaining transitive P0s.

## v1083: allocation-free direct fatal-release leaves, not the full chain

`daimos-testkit` now provides the executable
`test-daimos-fatal-release-scope-v1083` source contract. It checks that
FILE's dispatch through VFS SYNC, PIPE/FIFO final close, DPY raw-list
STOP, checked TTY canonical-buffer/session release, U-area release,
VM destroy, MEMFS XIP unpin and actual C swap-detach do not call an MM
or backing allocator. It also guards MM-error branches and verifies
the DPY zero-word STOP is reached before the allocating replacement
branch. This is deliberately a direct-leaf audit, not an OOM policy
switch or a proof for transitively invoked HUP callbacks.

Native PDP-6 v870 PIPE failed-MM retry, v904 DPY physical-pin and
wrong-owner STOP retry, v912 DPY two-extent MM refusal/retry, and the
actual C v957 VM swap-detach and bitmap verification contract PASS.
The current source needs no kernel modification; KCORE/MRES, permanent
RAM and ABI changes are all zero.

The audit separately detects an unresolved provisional ownership
case: `pipe_fifo_open_intr` invokes `pipe_close_ref` but discards a
failed last-reference MM free; `file_open` then discards its provisional
FD. This can strand a named FIFO object if the transient allocation
is somehow pinned or fails ownership validation. Ordinary FILE close
remains retry-safe. Whether a physical pin is reachable on the still
unpublished FIFO needs a dedicated dynamic fault test and a correct
ownership design, not an unconditional free.

Foreign in-flight provider leases, session HUP cascades and retry
liveness under clock/device failure remain open P0s. Production
automatic OOM termination remains disabled.

## v1082: refuse NEWSESSION before orphaning the previous TTY MM owner

A process could previously change session/pgrp/TTY state before the
checked retirement of the old session's last canonical-input buffer.
If MM refused that free, the TTY allocation retained its former
session but its last process had left, preventing normal owner-side
retry. v1082 moves the existing last-member checked release before
the identity change. MM refusal returns -1 without altering the
process, and the process remains able to retry. A peer still in the
old session prevents TTY retirement, as before. No extra retry
state or persistent memory is needed.

Source-linked native positive/negative acceptance passes, as does
the full controlling-TTY process guest. KCORE `023112` to `023116`
octal (+4 words, no new BSS). The other transitive fatal-teardown
provider P0s are not waived and automatic production OOM termination
is still disabled.
The full TTY session and session leader/HUP guests PASS. The v1077
physical-OOM guest also PASSes two D6FS SYNC and one MM_BUSY refusal,
owner retries and an independent process witness.

## v1081: final non-leader TTY MM refusal retains a PREVM owner

An ordinary session leader's HUP teardown already refuses to clear
the controlling TTY when its canonical input buffer is pinned. But a
non-leader can become the last session member, and the later
proc_tty_release_session used to return no status after failed MM
release. The following process VM/u-area teardown could therefore
retire the only remaining natural owner capable of retrying the line
free. The TTY session itself was deliberately retained, causing a
stranded resource rather than an unsafe reattachment.

v1081 makes proc_tty_release_session return zero on success/no-op and
-1 if a checked MM line free refuses. proc_finish_slot calls it before
vm_space_destroy; failure uses the existing PREVM+FINALIZE owner marker
and slot-0 retry, shared with the VM-destruction failure path. A
successful TTY free is not revisited after VM commits; subsequent
u-area MM_BUSY retries preserve the post-VM stage. The routine does
not allocate, and no additional persistent state word or ABI bit is
introduced. The API return type is changed from void to int; current
nonfinalizer callers need not act on the return because they retain
the session record, but session transitions merit separate audit.

The v1081 source-linked PDP-6 finalizer test verifies refusal before
VM destruction, successful retry, exactly-once VM release and
u-area finalization across two more MM refusals. v967/v968 real MM
TTY pin/owner tests, v881 u-area retry and v969 full guest with a
test-owned pinned TTY pass. The historical v969 fixture previously
injected the pin into an already detached TTY; it now explicitly
establishes a valid session record before that controlled injection.
The pre-v1081 finalizer fails the v1081 negative control as expected.
In a matched noninstrumented KCC full-image build the KCORE low-end
grows only two instruction words (octal 023110 -> 023112), with zero
new persistent data words.
The independent 256K v1077 requester-only OOM D6FS/MM regression
still passes. Production OOM termination remains disabled.

FILE close scope: the current D6FS SYNC routine returns zero without
media I/O; DTFS SYNC checks vnode shape only; TSFS SYNC is a no-op.
No generic kernel provider cancellation is justified solely for
these synchronous close paths. The shared DTFS/D6FS/TSFS gateway lock,
fatal foreign-owner nonwait and live private stack ownership are
still required: a concurrent non-close request can own a provider
lease. Do not forcibly unlock or retire its process.

# OOM policy: terminate only the faulting process (v981)

## v1080: prevent full DRM236 FIFO from busy-spinning

The v1079 packed watchdog exposed a third-request contention bug:
after RH extraction from `drm236_active_request`, the full-queue
code unconditionally jumped to retry. That branch could keep a
process spinning in its private kernel stack while another request
remained active and the single pending slot was occupied, preventing
the waiting owner from making progress.

v1080 replaces the unconditional branch with JUMPE, retaining the
wait on the active owner's event when a descriptor exists. The
private event cannot be discarded before its owner completes. Native
source-linked three-request test PASS, original v1079 negative
control FAIL, normal online Type-167/236 and v1079 watchdog tests
PASS. The full 256K physical-OOM test continues to pass.
Memory delta is zero code/data words; automatic production OOM
termination remains OFF because other provider and slot-0 liveness
gates are unresolved.

## v1079: complete in-flight DRM236 requests after hardware stalls

The DRM236 block driver was found to have only an early-boot bounded
polling timeout; normal process-context transfers slept on private
stack-owned PI2 completion descriptors indefinitely. The driver now
packs a 300-qualified-tick (five seconds at 60 Hz) countdown into the
LH of the existing active request word, retaining its RH descriptor.
KINIT patches a second storage-clock trampoline after the DSK watchdog
to its new DRM MRES export. No extra permanent state word is needed.

On timeout, the driver clears Type-167/236 hardware and uses the
ordinary PI2 completion path to mark precisely that request's
descriptor as failed (-1) and wake its live owner. The single queued
request is then promoted with a fresh 300-tick deadline. The FILE
provider still owns its lock and must release it itself after the
underlying device call returns; no forced unlock, stolen buffer or
synthetic provider completion occurs.

Native PDP-6 v1079 exact-deadline/queue/error accounting tests and
the disabled-watchdog negative control PASS. Normal real-SIMH DRM
read/write/queued I/O, direct DRM-root Stage1 boot to LOGIN with DSK0
unattached, and the full 256K physical-OOM FILE/MM v1077 guest all
PASS on the new clock/DRM integration. One additional KCORE code word
and 17 additional DRM MRES words yield 18 more resident words;
no additional persistent data words.

This is a bounded **device** completion under continued qualified
clock ticks, not a general quiescence/cancellation mechanism. A
provider stuck outside DRM I/O, permanent refusal, disabled PI6
clock progress or transitive memory allocation during fatal teardown
remain separate open problems. Production OOM killing is still OFF.

## v1078: do not sleep on a foreign provider from fatal FILE close

The shared DTFS/D6FS/TSFS register gateway formerly sent every
foreign lease contender to `proc_wait_event` until the owner returned.
That included a process already executing `proc_fatal_close_current_v879`.
A held foreign provider could therefore prevent the dying process
from completing one `file_close_all` pass and entering its own capped
sleep/backoff loop.

With no additional per-process state, the gateway now recognizes the
existing `PROC_FATAL_CLOSE_BIT` at the stable `file_table[-2]`
FDCTL offset whenever a *foreign* owner retains the lease. It returns
busy `-5` before attempting provider dispatch; FILE leaves the
descriptor owned and fatal cleanup sleeps and retries on its
owner-private continuation. The lock is **never** stolen or released
by the waiter, the shared provider workspace stays owned, and a
normal caller continues sleeping/revalidating. The PDP-6 indexed
load uses AC1 from the pre-existing saved request frame (AC0 cannot
index memory). Recursive owners and slot zero retain their earlier
immediate BUSY policy. Free providers still accept fatal operations.

Native v1078 source-extracted PDP-6 test PASS; pre-change source
fails the negative control. Existing provider lock, interleave and
indirect-dispatch native regressions and static lease safeguards PASS.
The full 256K PDP-6 v1077 D6FS/MM fault-injection requester-OOM
guest also PASSes on v1078 with its exact FILE/MM refusal counters
and bystander survival. Extra KCORE code is six words; permanent
resident data and the user ABI are unchanged.

This provides nonblocking **acquisition** for the fatal owner only.
The new path cannot cancel a provider that already entered an
unbounded call, cannot make an indefinitely rejecting provider
release its resources, and does not prove all fatal callbacks are
allocation-free. Do not fake completion, force release the lease,
recycle an active owner slot or enable production OOM termination.

## v1077: combined provider and MM refusal under real physical OOM

An isolated full-system 256K PDP-6 guest uses the existing default-OFF
OOM BRK branch with the option enabled only in a test-kernel copy. The
requesting process opens `/SYSTEM/INIT` via the real D6FS provider before
physical-memory exhaustion. Two test-injected D6FS SYNC refusals leave
its FILE descriptor owned; the third SYNC succeeds. The real FILE/VFS
boundary records three close attempts and two errors. A test-injected
u-area MM_BUSY then defers finalization; the idle-stack MM retry succeeds
on its second call. Counters: FILE 3/2, MM 2/1.

The parent receives only the requester OOM exit status and verifies that
a separately blocked PIPE reader survives and resumes and that process
memory accounting returns to baseline. Guest output `<KWP`, controlled
HALT: PASS. This covers simultaneous finite provider/MM refusals in a
genuine OOM termination. It does **not** establish safe cancellation of
a truly nonreturning provider, guaranteed reclamation under permanent
refusal, or complete transitive allocation-freedom. Production OOM
termination remains DISABLED. No production code or ABI changes.

## v1075: full PDP-6 physical-OOM requester-only test

The testkit builds a disposable 256K PDP-6 kernel image with the
already-existing default-off `PROC_OOM_BRK_ENABLED_V1000` branch enabled.
A child requests BRK 0776000 octal, a valid address but too large to
allocate after normal memory reclamation. The child is terminated with
OOM exit status 1 through the real fatal-owner teardown path. The parent
waits for this result, then releases an unrelated child blocked on PIPE
input; that child finishes normally. Process-slot and process-word
accounting return to their initial values. The full guest outputs `<KWP`
and a controlled HALT without any failure marker.

The separately built default-off guest runs the same program. Its BRK
call returns -1 rather than killing the child, which prints `D` and
exits normally. The parent confirms the non-OOM exit status, releases
the independent PIPE-blocked witness, checks memory accounting and
prints the complete successful sequence `<DNWP`. This negative control
confirms the test would detect a missing OOM-kill branch without using
an intentional assertion failure as its expected result.

Both variants are isolated build/test images, not production flags.
Neither variant injects an indefinitely failing FILE/MM provider.
Bounded provider cancellation and a transitive no-allocation proof
through all teardown callbacks remain required before enabling the
production OOM killer.

## v1074: provisional BRK extent double-release invariant

The nonadjacent PDP-6 BRK growth path can temporarily hold both the old
running image and an uncommitted larger replacement image. Previously,
if `mm_free(old)` refused, the rollback called `mm_free(new)` but did
not inspect its result. A second refusal could leak the replacement
without a process-VM reference. The kernel now preserves the exact
old MM error in an existing stack local and checks `mm_free(new)`.
If the new extent also cannot be released, execution enters the existing
resident storage-invariant HALT rather than returning with an orphaned
allocation. This is fail-closed containment of an internal MM ownership
invariant, not an OOM termination policy or a new recovery allocator.

Native PDP-6 source-linked tests accept successful -4/-5 rollback and
the double-refusal guard. Pre-fix and deliberately unguarded controls
fail. GCC KCORE and full KCC boot-image builds pass; the latter reaches
LOGIN under SIMH. Full native `daimos-brk-v1` KCC 256K guest regression
PASSes BRK/SBRK growth, heap data preservation and shrink.
Provider quiescence, fatal teardown liveness and
whole-process physical-OOM acceptance remain open. Production OOM
termination stays disabled.

DAIMOS uses a simple OpenBSD-style OOM decision: only the process whose
user-space memory allocation cannot be satisfied is a possible termination
target. It must not kill another process because that process occupies more
resident memory or backing sectors. No victim-score heuristic is used.

The native allocator first attempts normal allocation, cache reclamation,
compaction and permitted swapping. Swapping a different process to free RAM
is not OOM killing. The former read-only `proc_oom_select_v863` scanner,
which selected the largest eligible resource owner, was unused in production
and is deleted. `proc_swap_victim` remains for swap-out decisions only.

Automatic OOM termination is still DISABLED. Allocation failure currently
returns an error rather than entering an unsafe kill/teardown path. Enabling
termination requires allocation-free and bounded owner-side FILE/TTY/VM,
provider, backing and parent-notification cleanup, plus real RUN and BRK
exhaustion tests with failures and concurrent provider ownership. Kernel
internal allocation failures must follow their own safe rollback paths;
they must never trigger an unrelated process kill.

The v981 256K PDP-6 image builds and links. The source-linked policy,
provider and teardown audits pass. Historical tests written for the removed
largest-process selector are superseded by the v981 policy and must not be
used as current acceptance for OOM killing.

## v982: RUN and BRK have different failure owners

For failed growth of an existing process's address space (BRK), the
requester is the only potential fatal OOM candidate. The current BRK
allocator returns an error after normal reclaim fails; automatic killing
remains disabled.

SYS_RUN allocates resources for a not-yet-published child. A failed child
VM or u-area allocation discards that child through `proc_slot_discard`
and returns an error to its parent. Do not kill the parent merely because
it requested creation of a child that cannot fit. Never publish an
incompletely initialized child.

Testkit v982 source checks assert these existing paths. They are not
substitutes for booted exhausted-RAM failure and teardown acceptance.

## v983-v984 booted BRK testing

After repairing a missing test-only `memset` link, the standard BRK/SBRK
growth, data-preservation and shrink guest passes on a 256K PDP-6.
A separate v984 guest repeatedly grows its own break until a request
fails, verifies that the break remains unchanged, shrinks to the initial
break and exits normally. This is a verified failure/rollback behavior,
not a demonstrated physical-memory OOM: a user-address limit could also
cause the failing request. Instrument MM failure cause before claiming
true RAM exhaustion. Automatic termination remains disabled.

## v985: physical-memory pressure accepted

An isolated clean v197 RUN fixture with the existing v198 physical-OOM
extension passes on a booted 256K PDP-6. Four valid 18-bit BRK growth
requests fail under physical-MM pressure while preserving the caller's
break, VM size, data and RUN rollback/recovery. This provides a genuine
physical-pressure test distinct from the unclassified v984 repeated-growth
failure. It still does not authorize automatic termination; allocation-free,
bounded owner-side teardown is not yet certified.

## v986: owner-private fatal FILE retry audit

Native PDP-6 v926 validates fatal TERM owner-return FILE retries; current
v986 static test verifies that the owner stack performs FILE close, preserves
resources on refusal and sleeps between retries with an interval capped at
01000 octal ticks. A capped interval is not a cap on retry count. Permanent
provider refusal can prevent reclamation indefinitely. The owner-private
teardown, provider recovery and allocation-free closure graph must be proved
before any automatic OOM termination is activated.

## v987: permanent-provider containment

The source-linked v987 test verifies that the serialized provider lease is
only released by the normal owner return path. Recursive acquisition refuses,
foreign waiters sleep and retry acquisition, and owner-private fatal FILE
retry neither forces an unlock nor releases the u-area. This is containment,
not proof of a quiescence/cancellation protocol for indefinitely stalled
providers. Automatic OOM termination remains OFF.

## v988: pre-VM owner-exit safety boundary

Full-system PDP-6 v978 mixed fatal retry passes, including two failed FILE
closes and one u-area MM refusal with successful retry. A separate v988
source-linked test confirms that the current ordinary owner EXIT has a
specific limitation for OOM: `proc_exit_finish` detaches the current slot
on the idle stack; post-VM MM refusal is deferred, but earlier HUP or VM
refusal returns failure and leads to diagnostic HALT 1. Do not connect
ordinary exhausted-user-allocation handling to this exit path until its
recoverable pre-VM failures have a safe owner-preserving completion route.

## v989: pre-VM work must retain a schedulable continuation

The existing slot-zero retry is a post-VM MM finalizer; its scheduler
context inhibits PI6. Pre-VM session HUP and VM/provider callbacks may
sleep and therefore cannot simply be moved to that retry path. Native
source-linked testkit v989 protects this boundary. A safe requester OOM
exit needs a valid owner continuation through any sleeping work, followed
by the current non-sleeping final MM release after leaving its stack.
No OOM killing is enabled by v989.

## v991: live-owner HUP stage

Ordinary EXIT now completes the potentially sleeping session-HUP preflight
on its existing private kernel stack after FILE closure and before switching
to slot zero. On refusal it returns -1 without discarding the owner's VM or
stack. The existing finalizer continues to recheck session state. A full
256K PDP-6 BRK runtime and the v991 source guard pass. The older v905
standalone HUP fixture failed to link (missing `proc_tty_line_try_reset_v897`)
and is not counted as a passing HUP test. VM destruction is not yet staged;
OOM termination remains disabled pending fault-injected exit acceptance.

## v992: native HUP fixture recovery

The historical native v905 HUP retry fixture has been repaired with a
proper text-section test-only TTY reset callback. Current deferred
VM_NONE+FINALIZE HUP retry PASSes, while the old-behavior negative
control fails as expected (exit 010). This is not yet a booted v991
EXIT preflight fault-injection test. Both existing OOM P0 gates remain open.

## v993: owner EXIT preflight refusal

The native PDP-6 testkit v993 executes the production v991 EXIT branch,
with test-only FILE/HUP helpers. The first HUP refusal returns -1 without
entering the permanent-stack exit path; a second attempt reaches normal
exit handoff. This control-flow proof passes, but does not certify the
full-system provider/VM cleanup path or activate automatic OOM termination.

## v995: fatal owner-side HUP retries

The APR/EXEC fatal exit performs session-HUP cleanup on the live private
stack following successful FILE close. A temporary HUP refusal retains the
owner continuation, sleeps with the pre-existing backoff, and retries.
Native PDP-6 v970 and fault-injected v995 regressions pass. No additional
resident state was introduced. VM-destroy failure after the permanent-stack
handoff still prevents automatic requester OOM termination; both existing
OOM P0 tasks remain open.

## v997: pre-VM EXIT release retry

Production EXIT now distinguishes pending pre-VM release from the existing
post-VM u-area finalizer using bit 33 of the otherwise-used finalizer
u-area word. A failed VM release retains TRANSITION, identity, and saved
exit status; slot zero retries only this marked, non-sleeping release path.
After VM destruction succeeds the PREVM mark is cleared and existing
u-area release/finalization resumes. No permanent retry table or extra
kernel stack is introduced. Full-system v997 injected VM rejection then
MM u-area refusal, plus unmodified v978, and native v881/v869 PASS. These
are teardown tests, not full physical-OOM requester-killing acceptance;
automatic OOM termination remains disabled until that proof is complete.

## v998: classify failed BRK before requesting termination

The allocator distinguishes insufficient total free words after reclaim
(-1) from fragmented free space (-2), descriptor exhaustion (-3),
invalid arguments (-4), and busy/pinned extents (-5). Current BRK merges
all allocation and validation rejections into its ordinary -1 return;
this is not sufficient evidence to kill its caller. Source contract v998
PASSes and booted v985 RUN/BRK failure-and-recovery regression PASSes on
the v997 kernel. The next isolated test must preserve the allocator cause
at the failed BRK call and terminate only for real physical shortage.
The production killer remains off and the two existing P0 gates remain.

## v999: retain the BRK allocator cause

On failed PDP-6 BRK growth, the VM helper now preserves the internal MM
failure code in AC2 while retaining the existing user-visible AC1=-1.
Local validation failures are -4, and old-extent release refusal is -5.
Only MM -1 indicates insufficient aggregate free memory after reclaim;
fragmentation, descriptor capacity and busy failures remain distinct.
This adds no permanent RAM and changes no syscall ABI. Source gate v998
(updated for v999) and the booted BRK/sbrk growth, data and shrink test
pass. Automatic OOM killing remains disabled pending a full-system
physical-OOM requester termination and reclamation test.

## v1000: default-off requester termination branch

PDP-6 BRK now retains its existing syscall ABI, with an optional
compile-time branch (`PROC_OOM_BRK_ENABLED_V1000`, default 0) that enters
the already-established fatal-owner cleanup only when both AC1 and AC2
are -1 (failed BRK and MM_ERR_NOMEM). The isolated native v1000 test
activates this branch and verifies successes, invalid and fragmented
failures return normally, while simulated physical shortage enters the
owner fatal path. The unchanged full-system BRK growth/data/shrink test
passes with production behavior disabled. Genuine physical shortage,
whole-process cleanup and unrelated-process survival are not yet proven
in one booted test; the automatic OOM path remains off.
