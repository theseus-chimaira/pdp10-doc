# KCC DImode wide-literal zero comparison fix (2026-11-22 V1)

Status: fixed, with targeted PDP-6/KA10/KI10/KS10 runtime validation.

## Symptom

The exploratory test following the `950613-1.c` float-to-DImode fix
showed that `value != 34359738368LL` failed even when the same value
compared equal to a nonconstant `long long` variable. The conversion
and the stored two-word value were correct.

## Root cause

`ccgen2.c:gdimodezero()` selected its fast path for a zero literal
using only `Niconst`. For a wide `N_ICONST`, `Niconst` is the low word;
the high word is stored separately under `NF_WIDE`. Thus 2^35, 2^36,
and other nonzero multiples of 2^35 could be mistaken for zero.

## Fix

The fast path now requires a DImode integer constant and obtains
both words with `dimode_iconst_words()`. It selects the zero test only
when *both* words are zero. Nonzero wide constants follow the normal
two-word comparison path. No changes to the integer representation or
the general comparator were needed.

## Targeted verification

* Original exploratory 2^35 comparison reproducer: guest PASS on
  KA10, KI10, and KS10.
* Registered `kcc-dimode-wide-constant-compare-v1`: guest PASS on
  PDP-6, KA10, KI10 and KS10, with and without optimization (8 runs).
* Cases include operand order, positive and negative 2^35,
  comparisons against zero, and zero-to-zero comparisons.
* Testkit inventory, script ownership, raw p10run ownership, and
  fixture integrity audits: PASS.

No full compiler matrix was rerun.
