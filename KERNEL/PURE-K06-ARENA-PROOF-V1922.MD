# PDP-6 PURE managed physical arena proof (v1922)

The contiguous allocator publishes up to three sorted managed arenas.
Boot-reserved core is removed from the managed ranges and their live
allocated descriptors. An 18-bit physical address is therefore not,
by itself, proof of RAM or allocator ownership.

The shared optional PURE assembly range proof validates installed
core size, all arena descriptors (including trailing entries), and
containment of an exact range in one managed arena. It returns 0 for
contained, -2 for an unmanaged range in an otherwise valid arena
set, and -4 for malformed geometry. The table proof also checks
allocated descriptor count/capacity, nonzero spans and sorted
nonoverlap, reusing the current containing arena end. Empty tables
still require valid physical arena metadata.

DROP, PREFIX, FULL, deferred owner recovery and terminal teardown use
this shared proof before MM ownership decisions. PREFIX claim only
returns fragmentation for a valid retained RAW suffix with unavailable
prefix; malformed tables refuse before the original MM mutation.
Escrowed absence at terminal retry requires the original range to be
managed. PURE reclaim validates before and after compaction, with
ordinary PI behavior preserved across the compactor itself. This does
not replace the general allocator's existing valid-table and PI
serialization contract or protect against arbitrary MM memory damage.

Implementation 225aa401; daimos-testkit a3ecdda. Native gates pass 18
arena cases, 12 table cases, seven original MM mutation-refusal cases,
five executed unsafe ASM controls, two reclaim cases and four extra
terminal arena cases. Prior native ownership/commit/terminal gates
pass. Native fixtures now explicitly describe managed physical core;
actual MM/finalizer integration remains source-linked. Physical PI6
and production acceptance have not been established.

Folded KCC lowmem tape PURE MRES 3565 -> 3649 resident words (+84),
BSS zero, fixed KCORE byte-identical to v1921, low end 025136 octal.
No additional per-process or permanent RAM. K06 implementation audit
is closed; K07 immutable executable backing lifetime is next.
RAW PURE and mode3 remain OFF by default.
