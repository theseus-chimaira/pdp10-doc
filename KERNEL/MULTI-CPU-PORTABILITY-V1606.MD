# KCORE v1606: PDP-6-first modularity contract

Date: 2026-11-15. This document supersedes the *scheduling*
implications of v1605; its existing CPU source inventory and full
KCC PDP-6 binary-identity evidence remain valid. KA10 and KI10
hardware requirements are design constraints, NOT active port tasks.

## Decision

Finish PDP-6 KCORE before starting another processor. P1-E
uncompressed swapless PURE process reclamation, immutable backing,
PI/owner/EXEC/EXIT/WAIT rollback and real guest validation are
still open. Optional mutable RAM compression comes later only with
measured whole-system savings. The separate P0-B media-retention
requirement and strict 32K/optimization HOLD are unchanged.

The existing v1605 arch source manifest remains the only CPU
selector. It is verified against a byte-identical PDP-6 KCORE
DXR and map. Do not introduce a second machine build target,
segmented or paged VM state, runtime CPU function table, new
per-process fields, or extra branch/call in PDP-6 hotpaths.

## Keep policy, replace only machine mechanisms

| Common ownership and policy | PDP-6 machine implementation | Portability condition |
|---|---|---|
| Process slots, WAIT, TTY, events, scheduling | proc_pdp6.s | Hardware PI/context leaves remain recognizable |
| Kernel/user entry, syscall selectors | mach_user_pdp6.s, syscall_dispatch_pdp6.s | UUO trap and user-memory translation are distinct from selector policy |
| VM lifecycle and atomic runnable state | vm_pdp6.s, vm_pdp6_swap.c | Backend owns complete mapping and restore, not generic proc code |
| Physical allocation and pin ownership | mm_pdp6.s | 18-bit contiguous descriptor remains valid for PDP-6 kernel/device extents |
| Swap-pressure, backing and rollback | proc_swap_pdp6.s, proc_pure_backing_pdp6.s | Private mutable-state ownership is not equivalent to an APR suffix |
| Executable validation and source ownership | exec_load.s, exec_lowmem_pdp6.s | Physical compressed destination and MEMFS XIP are PDP-6-specific |
| Syscall INFO and MonitorFS accounting | syscall_info.s, monitorfs_runtime.s, monitorfs_swap_v63.c | Do not treat process vm_state RH as a portable physical base |
| Machine boot, PI save areas, I/O and module loading | kcore_pi_pdp6.s, KINIT, MRES drivers | Low-core vectors, fast ACs, DMA and machine privileges are not portable by name alone |

The current `VM_SPACE_ACTIVE` macro is a PDP-6-style RH-nonzero
check. An independently segmented or paged backend cannot claim
correctness merely by preserving that packed shape. Likewise,
`vm_user_words` returns a physically contiguous mapped pointer
today; logical user spans on paged hardware need not be contiguous
or writable. Common policy must not assume the current physical
representation. The source map identifies owners, not a universal
hardware ABI.

The present two-word MM extent descriptor contains an 18-bit
physical base. Do not expand it to fit wider page-frame addresses:
the contiguous kernel/device allocator and any later user frame
mapping are different responsibilities. The KI10's 512-word
pages, 22-bit physical addressing, executive/fast-AC/trap rules
explain why this separation matters, but this does not authorize
implementing KI10 paging. The KA10's optional protected high
segment likewise does not make shared text a prerequisite for
completing PDP-6.

Sources: DECsystem-10/DECSYSTEM-20 Processor Reference Manual
(1982), sections on KI10/KA10, and DEC-10-XSRMA-D hardware manual
(1976/1977); see v1605 for exact source locations and links.

## Source-organization decision

Keep complete `vm_pdp6.s`, `kcore_pi_pdp6.s`, and
`mm_pdp6.s` as replaceable source units. There is no value in
breaking them apart merely for a theoretical target.

The mixed scheduler/context `proc_pdp6.s` and
swap-policy/physical-transfer `proc_swap_pdp6.s` could eventually
be split. But many native regression tests extract labels and
mutate exact source files. A premature split risks breaking those
checks without improving the existing PDP-6. Retain their
machine-source-map identity and split only when a real PDP-6 change
benefits and full source-sensitive native/guest tests are updated.
Similarly, retain direct EXEC/INFO physical paths until a correct
logical-memory read/write path is genuinely required.

This is a source-organization contract, not a paged-MM interface
specification or a proposed set of future CPU implementation steps.

## Acceptance

Any preparatory PDP-6 source/header change must preserve the linked
KCORE and symbol map, `kcore_end` placement, PI save-cell aliases,
user/syscall ABI, per-process RAM, active swap and MRES behavior.
Place all new tests in daimos-testkit, prove no compiler/assembler
regression, and require real PDP-6 physical owners/PI/VM/MM gates
before closing P1-E. For the v1606 documentation/header pass,
the full image comparison is a source-preservation gate, not
a claim that P1-E is working in the production scheduler.

The fresh v1606 full KCC PDP-6 ROOT=tape KCORE DXR and map are
byte-identical to the original v1603/v1605 baseline. DXR SHA-256:
`830559842b15985c986b9145437d193cffacb7b8b654a7fb14f437788ce646ae`.
The existing daimos-testkit v1605 source-manifest check PASSes,
including fail-closed refusal to build as KI10.

See DAIMOS `design/kcore-pdp6-first-cpu-contract-v1606.md`
for the direct code inventory and `KERNEL/SWAPLESS-MEMORY.MD`
for the active RAW PURE acceptance conditions.
