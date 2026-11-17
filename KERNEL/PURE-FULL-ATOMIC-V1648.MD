# P1-E v1648: FULL MM release and pin refusal matrix

Date: 2026-11-17. Scope: source-linked native PDP-6 validation,
not a live multiple-process provider-scheduling acceptance.

The opt-in PURE MRES FULL restoration transaction (v1391)
preserves the authoritative uncompressed writable suffix in a
`PACKED_PROCESS` MM extent, allocates a separate full user APR,
loads validated immutable text under the exact destination bound,
then overlays the original writable suffix. The process must
remain nonresident/TRANSITION-owned until the old suffix can be
released and the new user APR is safe to publish.

The accepted native testkit v1648 checks these three additional
MM failures against the real compiled FULL C transaction:

1. `mm_free(PACKED source)` returns `MM_ERR_BUSY`.
2. The same owner release returns `MM_ERR_INVAL`.
3. `mm_pin(new full APR)` returns `MM_ERR_BUSY`.

For all three, the exact old unpinned RAW descriptor and its
word-identical contents survive without a second process owner,
the provisional target is completely freed, the saved executable
backing and PACKED storage mode remain authoritative, and
`PROC_TRANSITION` remains set. Removing the one injected failure
allows the same v1391 transaction to finish with one exact full
PROCESS owner, restored text and raw suffix, zero pins, valid
published resident APR and cleared transition.

The original v1395/v1397 source-linked native MM tests and
v1611/v1612 scheduler service tests still PASS. Two independent
test-only invalid FULL sources are separately compiled and run
on native PDP-6: omitted authoritative suffix overlay exits
`0706` octal and intentionally leaked target on source-free
invalid refusal exits `01004` octal. Both are rejected.

DAIMOS report: `design/kcore-p1e-full-mm-atomic-native-v1648.md`.
Testkit: `tests/validation/pure-full-atomic-v1648/`, commit
`500d023`, branch
`test-kcore-p1e-full-mm-atomic-v1648-20261117`.

This proves only the tested source-free and pin failure paths:
actual physical concurrent bounded FULL reload versus sleeping
DTFS I/O/mutation/unmount, permanently uncertain MM ownership,
32K/48K/64K peak RAM and distinct KA10 VM/APR behavior remain
OPEN. The v1647 real V2 quiescence/RUN admission and v1638
physical simultaneous pinned source/full target are independent
accepted gates. No permanent KCORE/MM source, ABI or RAM changed.
PURE default OFF and strict 32K optimization HOLD.
