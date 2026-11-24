# PDP-6 RAW PURE K05 v1903: FULL retry to RESIDENT and typed WAIT

Date: 2026-11-24. **Scoped 48K physical PASS.** This document records
the original PDP-6 transaction completion after v1902's dual-owner
source-free refusal and autonomous slot-0 target rollback. It is not
RAW PURE production enablement, a mode3 result or K05-K09 closure.

## Ownership and safe publication sequence

The initial source-built five-child TSFS/BRK/PIPE workload enters
an original FULL restore with both PACKED RAW source and provisional
PROCESS target pinned. One external test-only source descriptor pin
causes the real `mm_free(PACKED)` return at `036031` to be -5.
The process remains TRANSITION-held, VMbase0, with its original
private executive continuation, FDCTL and backing retained.

After the controller releases only that external test pin, the
permanent kernel's original slot-0 backoff invokes the optional
PURE reconciliation routine. The actual original native target
`mm_free(PROCESS)` at `033263` returns MM_OK=0 at `033264`;
MM occupancy drops 18 -> 17. The recovery verifies a sole RAW
source before clearing TRANSITION at `033313`. The saved CTX_PC
and FDCTL are unchanged through cleanup and independent parent
user heartbeat/PCLK progress continues.

Next, the scheduler retries the actual FULL, allocating a new
temporary target (MM17 -> 18). The second original `mm_free(PACKED)`
returns MM_OK=0 at `036031`; the original source is gone, and the
sole unpinned PROCESS target occupies `120000..130000` (MM18 -> 17).
The original service returns at `033574` with RESIDENT VMbase
`120000`, TRANSITION clear. This is a genuine committed APR,
not a fabricated READY_ONCE result.

A test-only gate in the original resident parent's USER APR at
`135335` remains zero throughout the transaction. Only afterward
the simulator sets it, allowing original parent execution to pass
from heartbeat into five typed WAIT checks. All five child output
markers `w`, parent `Z`, and actual PDP-6 HALT PC `000000` pass.
At `Z` there are 8 original MM extents; after HALT, 6;
no slot3 PROCESS/PACKED owner remains and the TSFS ROOT is unchanged.

## Current compiler and breakpoint controller integrity

The installed GCC differs from the accepted v1902 compiler.
From unchanged DAIMOS source, the current KCORE DXR IMAGE is
`024167` octal (+2 words), permanent BSS `000415`, and optional
PURE MRES IMAGE `004615`. The installed PURE base is now `032453`,
the runtime process table `045534`, and slot3 u-area `046653`.
These are independently matched to real native linker relocations,
MM extent records and physical process-table pointers. No compiler
or assembler semantic regression is inferred from the size change.

The first candidate simulator trace was NOT accepted: bare SIMH
`nobreak` leaves previously installed breakpoints, which led to a
second FULL source-free CALL being confused with its RETURN.
v1903 uses explicit `nobreak all` at ten stage boundaries. A
separate wrong-address trace was also stopped rather than accepted.
The final source-matched original 48K physical run passes its
complete read-only owner/PC/ROOT/WAIT/HALT verifier and ten
independent unsafe physical transcript mutations all reject.

Production image SHA256:

- KCORE `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- PURE `55be5113d4d15fe5c4a44409dca22afd463c75f05f661b3f8f71451a057c577f`
- Accepted original physical trace `03a3763c12f1da7a8dc6297e8857639d46f64ed63fe5517ec7f2dc16552e3796`

Testkit:
`daimos-testkit/tests/validation/pure-k05-full-commit-v1903/`.
DAIMOS implementation report:
`DAIMOS/design/kcore-k05-full-commit-v1903.md`.
Accepted scratch evidence:
`$TMPDIR/daimos-k05-full-commit-v1903-physical-v3/`.

**Remaining K05:** permanent source pin, foreign or uncertain MM
owner, terminal failed-FULL cleanup, actual PI6 restore interleavings.
K06-K09 and compressed mutable mode3 remain separately open.
RAW PURE remains OFF by default.
