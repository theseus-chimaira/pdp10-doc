# PDP-6 PURE integrated MM geometry and FULL replies (v1921)

The native MM table is sorted by physical base. Sole-owner discovery
now uses one pass and checks every nonempty, bounded and nonoverlapping
range, not merely ranges intersecting the requested owner. FULL reuses
this discovery, eliminating the separate weaker scanner. Its exact
two-owner proof also enforces the sorted-table invariant and preserves
independent MODULE/KERNEL_DYNAMIC namespaces and foreign-range checks.

FULL checks the original source pin before allocation and both pins
before the first destination write. Following target unpin, the
original source identity and exact remaining pins must be proven
before a base-only source unpin. Both original owners must be unpinned
before source deletion. All free replies require the independent
sole-successor proof, including reported success. Rollback also
revalidates the target after unpin before freeing it. Ambiguous owners
remain quarantined under TRANSITION with original escrow retained.

Native daimos-testkit c521176 passes six FULL reply faults and five
geometry cases; seven unsafe source mutations fail their assertions.
The v1395/v1397, v1612, v1839, v1855, v1902, v1906, v1918 and v1919
gates pass. These source-linked tests execute real MM rebase/claim/
release/commit ASM, with controlled allocation, pin/free, PI and loader
fixtures. No physical PI6 or production acceptance is claimed.

Folded KCC lowmem tape PURE MRES: 3585 -> 3565 resident words (-20),
BSS zero. KCORE byte-identical to v1920, low end 025136 octal. No new
per-process fields or permanent RAM. The managed-arena count and
containment audit remains open under K06. RAW PURE/mode3 default OFF.
