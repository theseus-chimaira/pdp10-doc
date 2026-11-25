# Direct MEMFS executable mutation admission, v1938

The MEMFS runtime is a singleton. Provider-local vnodes may omit the
public mount field, so backing ownership checks reconstruct the
canonical vnode from the published MEMFS root and actual file index.
An unpublished filesystem has no live VFS backing owners.

WRITE and TRUNCATE check executable ownership at the actual runtime
entry. UNLINK checks its resolved child before releasing any data or
namespace entry. PREPARE_UNMOUNT checks mount ownership before data
destruction. All use the existing PURE scanners, including transient
loader reservations and staged, resident and nonresident process
owners. Existing raw-XIP pin checks remain in place. Internal data
resize and teardown helpers retain their caller-owned admission
contract. No extra permanent data or process fields.

Implementation 3dbfb51b; testkit 5d78b7c. Native source-linked MEMFS
runtime, allocator and real backing scanners pass eight configurations
at mount IDs 2 and 18. Protected metadata, allocation and data remain
unchanged; unrelated writes and mutation after owner release succeed.
Five unsafe variants fail executed assertions. Historical MEMFS
allocator, fatal backstore/XIP and FIFO regressions pass after isolated
fixture linkage updates. Other historical runners not all rerun.
Controlled MM/backstore callbacks are not whole-guest PI6 or physical
media acceptance.

Folded lowmem KCC tape MEMFS MRES 1854 -> 1892 words (+38).
Initialized words 1841 -> 1879; BSS 13 unchanged. Baseline relink
uses the same fixed KCORE map and data/snapshot objects. Fixed KCORE
byte-identical at 025554 octal; PURE MRES 3879 unchanged. No extra
persistent state. This validates admission under the existing
non-sleeping MEMFS execution contract, not complete-guest concurrency.

K07 remains open for poisoned administrative retirement and integrated
lifetime validation. RAW PURE and mode3 remain disabled by default.
