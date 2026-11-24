# PDP-6 RAW PURE reclaim failure policy (v1916)

The optional PURE reclaim entry implements the permanent
proc_swap_reclaim ABI. Hard MM compaction failures (INVAL -4,
BUSY -5) must propagate to the BRK and general allocation callers,
before or after a RAW prefix DROP. Ordinary memory pressure may
still request another eligible sleeping/stopped owner.

A successfully committed DROP remains PACKED even if subsequent
compaction fails. A refused DROP may clear TRANSITION only when
the original resident or committed PACKED extent is proven exact
and unpinned. Otherwise leave the process quarantined and return
a hard refusal; do not initiate further victim reclamation.

The source-linked native PDP-6 regression reproduces the original
failure (0601 octal) and passes five targeted cases after correction.
No new persistent storage or fixed KCORE code is required. The
standalone native service region grows eight 36-bit words.

K05 implementation is complete. Persistent MM ownership uncertainty
(K06), executable backing lifetime (K07), scheduler integration
(K08) and production integration (K09) remain open. This scoped
native validation does not prove organic physical fragmentation,
descriptor exhaustion or broad platform acceptance. RAW PURE and
mode3 remain disabled by default.
