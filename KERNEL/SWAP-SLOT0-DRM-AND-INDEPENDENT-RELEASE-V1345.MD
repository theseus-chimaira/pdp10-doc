# v1345: physical no-partner recovery and slot-0 device wait

Date: 2026-11-08. Scope: read-only source-linked diagnostics from a
genuinely booted 256K PDP-6 guest, with a disposable test-only MM
owner and held-provider fixture. The production kernel is unchanged.

## Exact observations

The original v1343 guest was stopped after 26 unchanged live
samples: PC remained in `proc_wait_boot` with a pending swap
request, clock advancing and no child lifecycle completion.
This is not sufficient evidence for a kernel scheduling defect.
Its original console, read-only SIMH transcript, linked addresses
and map were retained before interruption.

The v1344 rerun additionally reads PDP-6 AC1/AC17, linked DRM236
active and pending request words, storage state and scheduler kick.
The active DRM236 request descriptor contains the 300-tick watchdog
in LH and a pointer to its caller-owned three-word descriptor in RH.
At nine observed `proc_wait_boot` samples, AC1 was exactly the active
request pointer plus two words: the device completion event.
The active descriptor changed across successive samples as requests
completed, with real swap activity continuing. The source-linked
native DRM236 timeout and MINIT-binding regressions PASS.

At sample 15 the real production swap-in allocator returned
`MM_ERR_NOMEM`; the swap service chose its no-partner cooldown.
The guest had 31 swap-outs and 13 successful swap-ins. The genuinely
scheduled spinner saw the real cooldown, used the unchanged `mm_free`
to release all four owner-tagged 8192-word test extents and called
the unchanged `mm_unpin` on its own resident process extent.
Sample 16 recorded zero remaining test owners, exactly four freed
extents, no owner errors and cleared pin. Subsequent successful
swap-ins increased to 15, 16 and finally 18 by sample 18.

The same sample 18 showed `real_oom=1`, `fatal_busy=3`, provider
release published (`-1` as a 36-bit word), exactly one holder
entry and return, and the original provider owner executing the
normal production unlock path. No synthetic provider unlock,
fabricated MM allocation or forced process-state transition occurred.

`daimos-swap-slot0-recovery-evidence-v1345.py` independently checks
the linked, read-only samples and rejects six deliberately corrupted
sets of counters. Both the source-linked DRM236 300-tick watchdog
regression and its production MINIT binding test also PASS.

## Acceptance boundary

The v1344 diagnostic deliberately stopped after 18 samples. Its
ordinary full-guest strict postflight therefore reports failure
(zero of 18 child reaps), as required; this is not a malfunction
of the guest. A host wrapper path-parsing correction handles a
carriage-return prefix on the original `v1075 work:` marker.

The **complete** 256K no-partner guest liveness requirement remains
OPEN: all 18 children must execute TERM/WAIT, the requester/provider
and witness milestones must finish, the machine must HALT normally,
and all MM extents, private pins and backing blocks must have exact
final accounting. The v1345 intermediate evidence does not imply
that cleanup has been verified, and does not establish a production
DRM236 or PI6 defect.

Evidence on the Termux host:
`$TMPDIR/daimos-swap-slot0-device-recovery-v1345-evidence-r1.tar.gz`

SHA256:
`8045c4f8f79e4893f31a0d39aaa347c8573b1619603d1ac5807f10e22d70ccd4`

The archive includes the linked kernel map, complete original raw
SIMH trace, filtered guest transcript, live sampler, versioned test
driver and a SHA256-verified internal manifest. No production
KCORE/MRES words or speculative swap fallback were introduced.
