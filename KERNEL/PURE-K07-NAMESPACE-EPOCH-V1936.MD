# Executable namespace revision validation, v1936

The executable loader snapshots a single resident namespace revision
before pathname resolution, using temporarily unused STAT scratch.
After the v1933 backing reservation has been acquired, it compares the
revision before STAT, permission and image reads. A changed revision
refuses admission and releases the reservation through the normal
loader return. The physical provider lease drains earlier operations;
subsequent backing mutation remains subject to the owner scanners.

The register gateway brackets potentially mutating operations with
revision updates before and after dispatch. This includes failures
because a failed physical operation may have partially completed.
The completion update precedes physical lease release. MEMFS shares
operation classification without taking the physical lease. VFS mount
publication and retirement also advance the revision, including slot
reuse. The counter saturates rather than wraps; new executable walks
refuse a saturated revision. No additional loader frame words or
per-process state are needed.

The comparison deliberately permits conservative refusal on an unrelated
concurrent mutation, rather than an unbounded retry. Its scope is
executable admission against gateway-visible mutations. It does not
establish generic pathname linearizability or detect silent external
physical exchange. Direct MEMFS mutation admission still needs audit.

Implementation 456ff3bf, testkit 9bee952. Source-linked native gateway,
loader, reservation and VFS publication/retirement gates pass 50
provider/opcode cases, lookup and lease-wait change cases, saturation
and mount changes. Five unsafe controls fail executed assertions.
Earlier loader, mutation, provider lease, D6FS and DTFS unmount gates
pass. The v1560 auxiliary fixture was adapted but not rerun. Lookup
and physical callbacks are controlled; full recursive pathname walk,
fatal private-stack lifetime and physical PI6 acceptance remain open.

Folded KCC lowmem tape fixed KCORE low end 025444 -> 025500 octal
(+28 words), DXR image 025001, BSS 000417 (+1), PURE MRES 3879 unchanged.
RAW PURE and mode3 remain default OFF.
