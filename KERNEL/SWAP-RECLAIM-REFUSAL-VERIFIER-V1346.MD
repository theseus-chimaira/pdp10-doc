# v1346: source-linked reclaim refusal is not a fatal swap transaction

Date: 2026-11-08. Scope: daimos-testkit postflight classifier correction,
not production KCORE/FS/MM/swap code.

In the accepted v1345 *intermediate* physical 256K PDP-6 sample,
the real slot-0 swap-in allocation encountered one `MM_ERR_NOMEM`,
one no-partner cooldown, and one entry through the assembly label
`proc_swap_out_fail`. The label is shared by preflight refusal
paths, including pinned process extents, unsuitable process state,
absent swap backing, and owner-side FILE teardown. It also handles
later failed transaction paths. **The label by itself cannot
distinguish a fatal transaction from an ineligible reclaim victim.**

The final v1303 classifier previously rejected any nonzero count
for this label as a fatal swap-out. This was an instrumentation
interpretation error that would reject the genuine no-partner
scenario even when its cleanup completed successfully. Testkit
v1346 retains the source-linked count as diagnostic data and
no longer treats it as an independently proven fatal error.

Final acceptance remains strict: the actual guest must reach HALT
with all original requester, witness and child TERM/WAIT milestones;
source-linked MM errors must correspond to real typed allocation
failures; the no-partner cooldown and restore accounting must
balance; no fatal swap-in may occur; both the swap-block and the
backstore allocation counters must be **zero**. A successful
intermediate sampler trace is still *not* an accepted final guest.

The versioned v1346 regression executes the original v1303 selftest
and a specific physical refused-victim positive scenario, with seven
negative controls for fatal swap-in, missing cooldown, unbalanced
restore, missing allocator evidence, fallback, and both independent
backing leak counts. This corrects the test without introducing a
production fallback or changing the kernel's resident footprint.

The full booted 256K no-partner recovery and leak-free postflight
acceptance remains open until a guest completes the original strict
tests. The previously validated v1345 physical subcase remains
documented in `SWAP-SLOT0-DRM-AND-INDEPENDENT-RELEASE-V1345.MD`.
