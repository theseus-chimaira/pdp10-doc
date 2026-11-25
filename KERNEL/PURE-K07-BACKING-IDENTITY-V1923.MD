# PURE backing reconstruction identity, v1923

The one-word executable record and existing u-area mount extension
retain the original mount slot and executable index. The common
backing decoder now requires a nonzero backing and stable u-area,
a published root slab, and an exact public mount ID in that root.
It reconstructs the canonical vnode and returns the complete root
word as AC5 alongside vnode AC1 and text length AC2.

The shared PREFIX/FULL adapter retains the root in its existing
stack frame. Both paths require DXR PURE and exact saved TX2 text
length. Before payload application and after successful provider
I/O the adapter checks the original vnode, text length and complete
root word again. Failed checks return a loader error; the K06
transaction retains its original RAW suffix, does not publish an
APR, and rolls back only its proven scratch. FULL preserves a typed
source-free refusal after exact rollback, converting only false
success to BUSY.

Native testkit 042601d passes ten PREFIX/FULL source-change cases,
five decoder checks and five unsafe assembly negative controls.
The v1387/v1390 shared loader, v1563 staged admission/ownership,
v1649 simultaneous pinned mount/backing guard, v1839 and v1921
FULL rollback gates pass. The new test has controlled VFS stat/read
and MM/PI fixtures. No physical PI6 or concurrent media acceptance.

Folded lowmem tape KCC PURE MRES 3649 -> 3673 words (+24), fixed
KCORE +8 words (low end 025136 -> 025146 octal), BSS 000415 unchanged.
No additional permanent data or per-owner allocation. K07 remains
open: this transaction snapshot cannot distinguish pre-existing
same-slot reuse or an identical-root ABA replacement. Persistent
provider/media identity and actual mutation/unmount lifetime must
be audited before closure. RAW PURE and mode3 remain default OFF.
