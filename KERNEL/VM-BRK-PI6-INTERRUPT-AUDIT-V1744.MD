# BRK relocation PI6-only critical section v1744

Date: 2026-11-21. DAIMOS commit `d385fd6d`.

Following the Type 270 DRL root-read fix in MM allocation (v1742),
the interrupt audit found another long global PI-disabled region:
`vm_brk_resize` in `system/kernel/proc/vm_pdp6.s`. It covers the
descriptor/APR transaction and may call `fs_move_words`,
`fs_copy_words`, and `fs_zero_words` for an entire process extent.
Global PI inhibition could starve the Type 136 DCT while the disk
is transferring data. This is a source-derived hazard; no separate
reproduction of a BRK-triggered DRL was required or claimed.

The three BRK transaction lock entries now use `CONO PI,1002` to
inhibit scheduler PI6 while leaving disk PI3 and PI5 live. The four
corresponding restoration sites test the saved PI6 bit and use
`CONO PI,2002` only if that level was enabled on entry. This
includes reclaim/retry, allocation fallback, failure and APR commit.
No extra instructions or static/process memory were introduced;
physical copying, zeroing, descriptor ownership and error behavior
remain unchanged.

Validation: the v1744 source contract passes and rejects four unsafe
mutants (global mask, global restore, wrong saved-state test for
each restore register). All 33 native K01 acceptance tests pass.
The fresh 256K KCC full-profile guest passes DCS0 ROOT login,
DSH ECHO, external TRUE status 0, orderly HALT and CLEAN D6FS.
Testkit commit `970823f`. SHA256 of the native suite transcript:
`e4c133ebe39c8affe99368a80905140941c08303c449c12005ccc4fd7cf4aa1e`.
SHA256 of full-profile runner transcript:
`b5addef58278179f07221499187ed4fe190fc1d57b44bd91fad8d5370a6c6244`.

Next audit target: `mm_move_extent` still holds a global PI-disabled
transaction across `vm_extent_move`, which can copy a complete
resident process image. Other short MM pin/free/probe locks remain
global and should be measured before changing their semantics.
Strict 32K KCORE fit and K04 permanent-refusal policy remain open.
