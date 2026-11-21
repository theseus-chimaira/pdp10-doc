# KCORE low-memory and permanent-residency gates v1748

Date: 2026-11-21. DAIMOS source `ecfbe674` (includes the PI6-safe
MM relocation and current-KCC DSH size recovery). Measurements use
fresh, isolated builds; unrelated working-tree edits were not staged.

## Three independent limits

1. The `kcore-32k-gate` target (`system/boot/pdp6/image/
   check-kcore-32k-v1.sh`) requires a GCC `BOOT_PROFILE=lowmem`
   KCORE map with `__kcore_low_end` strictly below **`020000`
   octal (8192 decimal words)**. This is the KCORE endpoint limit,
   not the amount of installed physical memory.
2. `report-permanent.sh` separately requires the highest KCORE+MRES
   resident address to be strictly below **`040000` octal (16384
   decimal words)**. It accounts for resident module text, data and
   BSS, not disposable KINIT packaging metadata.
3. The 32K hardware profile has **32768 decimal physical words**;
   it does not relax either lower-address residency constraint.

## Fresh low-memory GCC result

Command: `make -C system/boot/pdp6 kcore-32k-gate`
with `BOOT_PROFILE=lowmem`, `BOOT=dsk`, `ROOT=disk`,
`COMPILER=gcc`, `PDP10_PREFIX=$HOME/cross`, and a fresh isolated
`BUILD`. The build/link succeeded; the mandatory gate failed:

```
KCORE 32K FAIL: end 024451 >= 020000 (must be strictly below)
```

The KCORE endpoint `024451` octal is 10537 decimal, **2345 words
over** the strict endpoint. `dxrcheck` reports image `023754`
octal and BSS `000415` octal. The KCORE fold plan already uses
`p10fold -m 2`; lowering the threshold to two is not a new
optimization opportunity. The previous PI repair's +28 native
object text words do not explain the much larger baseline gap.

## Separate 256K full-profile accounting

The KCC 256K physical-compaction build uses `BOOT_PROFILE=full`;
it must not be confused with the GCC low-memory link. For its
disk-root resident package set, with DTFS omitted and BLOCKSET
included only in the multi-root comparison, the resident report
shows KCORE `024644` octal (10660 decimal), KCORE+MRES
`050355` octal (20717 decimal), permanent last `050434` octal
(20764 decimal). Its strict single-root permanent-address result
is **4381 free-word units over** the `040000` ceiling; multi-root
is 4414 over. This full-profile measurement is diagnostic, not a
claim about low-memory resident package totals.

The largest individual full-profile MRES residents include DTFS
2808 words (omitted for disk-root), D6FS 2184, MEMFS 1854,
SWAP 1796, and MONITORFS 1592. Removing any mandatory provider
to make the gate pass would be a feature regression, not a valid
optimization.

## Next optimization direction

Keep the three limits distinct. Profile the low-memory GCC KCORE
object and linked symbol ranges, then prioritize duplicate code
and real instruction reductions in large mandatory paths without
removing functionality. Rebuild the exact `kcore-32k-gate` after
each candidate; preserve the independent 256K KCC ROOT/DSH/CLEAN
and physical MM compaction acceptances. This audit changes no
kernel code or memory limits.
