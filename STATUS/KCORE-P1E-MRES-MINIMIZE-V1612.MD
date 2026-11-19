# PDP-6 PURE MRES size reduction v1612

Date: 2026-11-16. Experimental MRES remains default OFF.
Measured KCC ROOT=tape, BOOT_PROFILE=full.

v1611 installed PURE module: 2275 PDP-10 words.
v1612 installed PURE module: 2196 PDP-10 words.
Net reduction: 79 words, with zero extra BSS and
zero new per-process state.

GCC lowmem v1611 installed PURE module: 1864 words.
GCC lowmem v1612 installed PURE module: 1758 words.
GCC reduction: 106 words. Compiler-specific folds are
also zero-cost. Fixed GCC KCORE remains byte-identical
at `024254` octal.

The new PDP-6 service policy uses one scan instead of
two for a retained PACKED owner, saving 30 TEXT words.
The existing p10fold generated plan folds 49 more words
between original v1381 DROP and bounded v1391 FULL.
The folded donor is relocation safe and needs no
extra JRST; hot paths have no extra branch.

An alternative shared-C preparation and extent helper
was tried and rejected after linking: 2356 module
words due to call handling overhead. It introduced
no valid net RAM improvement. The final tree retains
the original transactions without that extra object.

Source-linked PDP-6 tests verify DROP, fast prefix,
FULL fallback, source/refusal behavior, duplicate and
pinned MM owner failures and a mutated unsafe
READY_ONCE guard. The same-directory KINIT toggle
test checks module presence only when enabled,
2196-word footprint, a generated fold plan, and
byte-identical permanent KCORE.
An independent GCC test pins 1758 installed words.

Fixed KCORE low end stays 024534 octal. Standard disk
swap remains unchanged. Full physical guest and net
RAM acceptance are not complete. The opt-in flag
DAIMOS_PURE_MRES_ENABLED must remain zero for the
normal build. No KA10/KI10 or paging work is enabled.

Details: DAIMOS
`design/kcore-p1e-pure-mres-minimize-v1612.md`.
