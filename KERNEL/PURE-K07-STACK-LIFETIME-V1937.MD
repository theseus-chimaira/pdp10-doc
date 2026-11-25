# Executable reservation private-stack lifetime, v1937

The transient executable reservation list contains live private-stack
addresses. Its lifetime must therefore be protected independently of
VM backing and ordinary FILE ownership. The new address scanner checks
all records for overlap with the target u-area extent. It requires no
new process fields, storage or MM pin. The scanner does not yield.

Ordinary and marked EXIT finalization and unpublished RUN discard
check before destructive operations. The final u-area release checks
again before MM free; an outstanding reservation returns typed BUSY
and preserves the descriptor and exact owner. Foreign fatal delivery
records a pending event rather than closing the target's FILEs when a
reservation remains, even if a saved user-mode PC appears admissible.

The owner-side nonreturning EXIT path must have unwound all loader
and transaction frames before entering the stack-switch boundary.
A remaining published record is a fail-stop invariant; silently
removing it could abandon an executable dependency or leave an
unfinishable transaction. Normal owner EXIT keeps the existing
PREVM publication and PI6-masked idle-stack handoff.

Implementation 4288a3e1; testkit 9909995. Eight source-linked PDP-6
native cases and six executed unsafe controls pass. The extended v956
fixture exercises actual MM release with zero MM pins and an active
reservation, retaining the extent until retry. v1097, v869, v1086
and v1933-v1936 regression gates pass. v925 has stale extraction and
lifecycle assumptions; it was not counted as passing. Other older
source-linked runners have not all been adapted or rerun.

Folded lowmem KCC tape KCORE low end 025500 -> 025554 octal (+44 words),
DXR image 025055, BSS 000417, PURE MRES 3879 unchanged. This is native
controlled teardown validation, not complete-guest fatal PI6 scheduling
acceptance. Direct MEMFS mutation, poisoned administrative retirement
and integrated physical/lifetime acceptance remain K07 work.
RAW PURE and mode3 remain default OFF.
