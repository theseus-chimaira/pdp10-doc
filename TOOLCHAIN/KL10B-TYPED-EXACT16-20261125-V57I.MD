# GCC typed global exact16 pointers (v57i)

GCC 62c1156, testkit dd01ab4; based on v57d. This is the verified
GCC exact16 implementation for native arrays, standalone scalars,
and continuous packed members of the tested record forms. It does
not freeze a mixed GCC/KCC ABI.

## Representation

All pointers remain one 36-bit word, with a 30-bit global word
address. High-six-bit tags (octal): 01..44 denote packed physical
bit positions 0..35; 045 and 054 denote the first and second native
16-bit array slots, at physical positions 0 and 16. A standalone
right-justified scalar uses tag 025 (decimal 21, position 20).
These are software tags, not hardware byte-pointer operands.
Symbolic addresses use %GADDR and the existing ADDR30 relocation.
Native exact32 and canonical architectural OWGBP selectors retain
their previously documented representations.

## Lowering

Native exact16 declarations carry plain storage word addresses.
The typed address helper forms the correct slot or packed tag,
including constant initializers, stack and global storage, and the
supported word-addressed indirect record member references. Typed
loads/stores decode the tag before accessing memory. The existing
physical scalar extraction/insertion routines handle crossing fields,
retain adjacent bits, and extend according to the declared precision.
Direct standalone scalar moves use a physical 16-bit field at bit 20.

Packed advancement splits n into trunc(n/9) and remainder: nine
16-bit elements span four words. Normalize the residual physical bit
position, correcting a negative remainder by a borrow. Native
advancement adds the second-slot indicator to n, takes floor division
by two and selects the resulting parity tag. Subtraction for a packed
same-array pair returns C storage units, (9*d + tagdiff/4)/2, where
d is the word-coordinate difference. Native subtraction returns
4*d plus twice the slot difference. Front-end exact division by two
produces the C element difference. The typed comparison uses the
same address-first rotation and sign bias as exact32. Dynamic
same-base subtraction retains typed operands; the front end must
not interpret already-scaled integer offsets as tagged pointers.
Constant positive and negative pointer offsets use typed folding,
not a hardware ADJBP expression in the initializer.

## Acceptance

The dedicated gcc-global-exact16-v57e runner executes five cases at
O0/O1/O2/Os on SIMH and KLH10. The new comprehensive fixture covers
native/packed pointer parameters, static pointer formation and
positive/negative constant initializers, signed and unsigned access,
standalone scalar access, dynamic positive/negative arithmetic,
25 ordered subtraction/ordering pairs for each array form, pre/post
increment, 16-bit wraparound, automatic storage and indirect record
member addresses. Four unchanged historical fixtures cover the
native pointer call, integer pointer round-trip, packed global
round-trip, and packed global pointer arithmetic. All 40 final v57i
runtime observations pass. Provenance and linked sizes are in
pdp10-testkit/manifests/PDP10-GCC-GLOBAL-EXACT16-20261125-v57i.tsv.

The v57h predecessor passed the previous 88 extended runtime
observations (16 exact32, 16 record, 8 narrow, 48 wide/direct-packed)
and four static object profiles. The final v57i initializer-only
change has 28 identical extended assembly guards against v57h.
The four historical exact16 fixtures have 64 byte-identical local
assembly comparisons between v57h and v57d; the final change is
extended-only. These are dedicated checks, not changes to the
historical compiler matrix. Inventory: 2400 matrix sources, 438
special entries, zero unowned sources.

Linked TEXT words for the new final fixture: O0 3249, O1 2050,
O2 2024, Os 1959; DATA 7 and BSS 12 for each. These counts are
not peak RAM measurements. No new runtime helper or table is
introduced, but inline code is large and the two residual division
operations merit sharing. No speed or RAM improvement is claimed.

Staged GCC: TMPDIR/pdp10-gcc-exact16-v57i/gcc, cc1 SHA256
716b6fd43fb2702d57da9996dbd4b36b89bb5523371243e75c5ddc2ff82e2903.
Use staged DAS v56y and linker v56m. Production tools unchanged.

## Remaining work

The accepted scope is the tested native/scalar and packed reference
forms. More complicated nested arrays, ordinary record members,
other packed aggregate layouts, non-word-addressed indirect record
bases, arbitrary cross-width casts and void-pointer round trips
remain to be validated or implemented. The full 30-bit address
arithmetic oracle was previously exercised by the v56y independent
model; it is not yet a compiler-level full-range acceptance test.
Physical cross-section data access, multi-TEXT placement, extended
libgcc/freestanding runtime, KCC support and mixed GCC/KCC ABI are
still open. Optimize the exact-width lowering for resident size and
execution cost after correctness coverage expands. No DAIMOS or
compiler matrix changes.
