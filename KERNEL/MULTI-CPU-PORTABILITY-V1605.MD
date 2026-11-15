# KCORE v1605: PDP-6, KA10 and KI10 source boundaries

Date: 2026-11-15. Accepted scope: source/build reorganization
on PDP-6 only; no KA10/KI10 boot, paging or process support.
No added resident words, per-process metadata or runtime dispatch.

## Processor and VM split

| Backend | User mapping | Physical representation | Minimum initial target |
|---|---|---|---|
| PDP-6 | One APR relocation/protection image | Aligned contiguous extent | Current accepted KCORE |
| KA10 | One low segment, optional KT10A high segment | One or two independent contiguous extents | Single low segment |
| KI10 | Hardware page map, 512-word pages | 22-bit physical page frames | Fully resident mapped pages |

KI10 has 18-bit user virtual addresses and 22-bit physical
addresses; scattered physical pages cannot be addressed through
the present PDP-6 `vm_state` 18-bit physical base. KI10 has
different fast-AC context blocks, processor modes, page-fault
handling, and PI/trap setup. Do not claim that selecting
`VM_BACKEND=ki10` on the existing PDP-6 boot Makefile suffices.

Primary source: DECsystem-10 Hardware Reference Manual
DEC-10-XSRMA-D, section 2.16 (KI10) and section 2.17 (KA10),
https://www.bitsavers.org/pdf/stanford/sail/SAILON-75A_DECsystem-1020_Hardware_Manual_DEC-10-XSRMA-D_Jan77.pdf.

## PDP-6 source map (implemented)

The versioned `system/kernel/arch/pdp6_source_map_v1605.mk`
groups machine-owned sources. The kernel and original PDP-6 boot
Makefiles both include the same list. It identifies actual PI
handler, process, trap, syscall, VM/contiguous-MM, swap, KINIT,
module-relocation, and other PDP-6 source owners.

The output assembly file names, order of object inputs and P10FOLD
eligibility remain unchanged. Tested with separate full KCC
`ROOT=tape` KCORE builds: `kcore.dxr` and `kcore.map`
are byte-identical to the original v1603 build. The initial
source-map-only VM/EXEC/process generated assembly was also
identical. Later source comments change the literal EXEC/process
assembly text but not the final linked machine image.

This does NOT mean that the files are all wholly CPU-specific.
`proc_pdp6.s` still combines generic scheduling, event/TTY,
WAIT/EXIT and low-core PI6/context switching. `proc_swap_pdp6.s`
combines process-pressure policy with whole-image physical transfer.
Those source-sensitive testkit fixtures should migrate only when
the genuine backend calls for a source split. Do not introduce
additional resident calls or tables to pre-empt that work.

## Stable common logic

Keep the compact process and u-area ABI, syscall selectors,
generic scheduler/event policy, VFS, executable admission,
immutable-backed PURE lifecycle, raw disk/drum swap semantics,
and physical kernel allocation. The current MM two-word descriptor
contains only an 18-bit physical base: keep it for contiguous
kernel/MRES/device memory. KI10 gets its own page-frame owner
allocator and page-map representation, rather than widening the
PDP-6 hot descriptor or embedding page maps in every process.

## Required later replacement points

* **CPU**: `kcore_pi_pdp6.s` low-core vectors, AC-save aliases,
  `mach_user_pdp6.s` UUO entry/return, `proc_pdp6.s`
  user/executive context save/restore, PI6 software requests,
  `kinit_io_pdp6.s`, initial stack/AC/privilege setup.
* **VM**: entire `vm_pdp6.s` and `vm_pdp6_swap.c`;
  KI10 adds 512-word mapped pages and map-activation. KA10
  validates installed single-/dual-segment relocation hardware.
* **Executable + buffers**: `exec_load.s` directly decodes
  into `base+020` and publishes pinned MEMFS XIP APR image;
  `syscall_info.s` and MonitorFS read user startup words
  through raw physical bases. Page-aware bounded reads/writes
  replace those operations when a paged backend is implemented.
* **Storage + PURE**: `proc_swap_pdp6.s`, RAW PURE MM
  rebase/expand, `proc_pure_backing_pdp6.s`, and MEMFS XIP
  geometry assume one contiguous process image. Common
  owner/immutable-backing/rollback policy remains portable.
* **Physical device/module memory**: do not infer that PDP-6
  MRES I/O, PI binding, block I/O DMA pointers, or module
  relocation automatically work in the KI10's broader physical
  address space. Audit these when a real target is available.

The initial KI10 port is **resident-only paging**, with every
user page mapped/present and page-failure handling safe before
scheduler reentry. Demand paging, page-granular swap, COW and
hardware shared-code optimization are separate future features.

## Work not performed

No source split of the large scheduler/swap/VM files, no new
per-process page table, no new physical memory metadata, no
runtime dispatch and no KA10 or KI10 machine build. No
full-guest paging tests. P1-E uncompressed swapless PURE is
in staged v1604 common lifecycle/PDP-6 geometry work only.
Compression and strict 32K optimizations remain deferred.
DAIMOS design/implementation list:
`design/kcore-multi-cpu-portability-v1605.md`.
