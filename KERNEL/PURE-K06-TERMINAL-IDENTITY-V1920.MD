# PDP-6 RAW PURE terminal identity (v1920)

The v1900 direct deletion of an inferred second full PROCESS extent
is superseded. An uncertain FULL/DROP/PREFIX transaction must first
be reconciled by the existing slot-zero service. Terminal cleanup
never infers scratch from the slot, length and alignment alone.

Terminal admission proves a sole exact RAW PACKED extent with the
expected suffix length and aligned original APR origin, under PI
exclusion. The complete sorted MM table must have nonzero spans,
nonoverlapping ranges, bounded physical ends and valid descriptor
count/capacity. MODULE and KERNEL_DYNAMIC retain independent owner
namespaces. The original RAW sector address is escrowed in existing
u-area 067 (terminal bit31 and source field) even on pin refusal.
The PREVM armer preserves the escrow payload and saved exit status.

Each retry checks that original range; a different same-slot extent
is not a substitute. No pinned extent is freed. A separate absence
scan after either MM-free return verifies no surviving same-slot
user extent or foreign intersection. After prior verified terminal
admission, a later exact absence proof permits mode publication
without a second free. The common detach retires backing; terminal
success clears escrow and normal VM/finalizer cleanup proceeds.
No provider lookup, speculative scratch free or forced unpin is
part of this path.

Native PDP-6 tests in daimos-testkit 8025153: 23 leaf cases, five
unsafe mutations and original MM/VM/finalizer integration. The
integrated test runs real MM deletion before injecting a foreign
interior claim; PREVM must refuse both that claim and a same-slot
replacement, preserving exact identity until the injected claim is
removed. Escrow-loss mutation fails at 05. Prior v1609, v1666,
v1680, v1681 and v1919 native regressions pass. The historical
v1900 direct dual-owner fixtures require their pre-v1920 source.

Folded KCC lowmem tape PURE MRES 3556 -> 3585 resident words (+29),
fixed KCORE +3 words (low end 025133 -> 025136 octal), BSS unchanged
000415. No extra per-process or MM descriptor storage. This stage
is not physical PI6 or production acceptance; final integrated MM
geometry review remains K06 work. RAW PURE and mode3 default OFF.
