# PDP-6 K05 v1910: genuine parent EXIT with an unrepaired FULL child

Date: 2026-11-24. **Original 48K PDP-6 physical PASS (scoped).**

The optional RAW PURE recovery remains quarantined when its
original complete temporary PROCESS target is foreign-owned.
Previous v1906-v1909 tests proved its exact target-base escrow,
genuine user TERM refusal, capped pending retries and one real
PCLK PI6. The next ownership question is whether the parent
can exit normally without falsely tearing down that child.

The v1910 guest reuses the original five-child compressed
TSFS/BRK/PIPE/RUN workload with one **test-only parent USER APR
EXIT gate**; KCORE and PURE are unchanged. Original first
PACKED source `mm_free` actually returns -5. Only the provisional
target's MM metadata is test-corrupted owner3->owner2, leaving
its geometry, type, base and pins intact. A real original parent
TERM(3) syscall is refused (`TR`). The controller then opens
gate `135401` and the parent prints `E` and executes original
`dsys_exit(0)` from user mode; no kernel response or MM return
is fabricated.

Original fixed KCORE stops at `proc_exit_current=013655`,
native `proc_finish_slot_owner_v932=012610` on permanent slot0,
and `proc_idle_loop=017552`. Original parent slot1 descriptor
`045703` changes `045761400001` -> `045761600001` -> `0`,
proving the parent really finalizes rather than merely reports
an EXIT request.

The actual MM extent count decreases **18 -> 16**, removing
exactly these original parent resources:

| Resource | Physical span (octal) | Owner |
| --- | --- | --- |
| KERNEL_DYNAMIC u-area | `045761..046371` | `01001` |
| PROCESS APR | `130000..140000` | `1` |

There is NO other MM deletion or insertion. In particular the
authoritative child PACKED RAW `111200..120000` owner3 and the
unreleased temporary PROCESS target `120000..130000` foreign
owner2, both pin0, remain exact. Original child metadata at
`045711` changes `047017600401` -> `047017600001`: proper
parent1 -> parent0 orphan adoption with TRANSITION still held,
VMbase0 and unchanged original private executive CTX_PC
`340000016704`, FDCTL `000000004010`, and uarea word067
target escrow `040000120000`.

One actual post-EXIT clock PI6 enters original fixed KCORE
`proc_sched_pi_tick=017225` with hardware-saved word
`340000017562`, returning into the original slot0 idle
loop. MM16, ROOT, original RAW and target extents, transition,
escrow and saved executive context remain unchanged. The
simulator is then stopped by the **operator**: it is not a
native guest HALT or cleanup of the unresolved child.

The focused read-only source-linked physical audit passes,
including the real user syscall, parent EXIT/finalizer
breakpoints, exactly two freed MM extents, orphan reparenting,
saved child state and original PI6. Three targeted physical
unsafe mutants independently reject a missing slot0 finalizer,
premature child transition clearance and lost child escrow.
No broad repeated fault matrix is needed with unchanged KCORE.

Two discarded test-only launches are NOT acceptance evidence:
the first used an illegal simulator port above 65535; the
second used an incomplete guest build interrupted before its
compressed `/SYSTEM/COMP` TSFS media hook, and the user correctly
reported `!3` before any child workload. The corrected test
requires all three actual compressed TSFS members and `tsfscheck`
PASS before launching. Neither discarded trace is a kernel bug.

Source-built SHA256:

- Fixed KCORE `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- Optional PURE `09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`
- Test-only parent USER DXR `6204b31ec342a1fd406f64952ba74bbe4af442837d7ff587d13fef3e997d7338`
- Accepted physical transcript `262998e8a03dcad8b96247dd6761661cd5a8668226e0dd6db6332ea94bc10a70`

No production C/ASM, KCORE/PURE image, BSS, process storage or
default policy changed. Testkit:
`daimos-testkit/tests/validation/pure-k05-parent-exit-v1910/`.
DAIMOS design:
`DAIMOS/design/kcore-k05-parent-exit-v1910.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-parent-exit-v1910-physical-v3/`.

**K05 remains OPEN** for the live unrecoverable child's terminal
cleanup. Parent exit safely releases **its own** resources,
not the foreign MM claim. Other PI6/allocator ambiguities
remain; K06-K09 OPEN, RAW PURE/mode3 default OFF.
