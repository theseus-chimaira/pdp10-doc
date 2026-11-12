# DAIMOS KCORE feature state, v1507

Date: 2026-11-12. Incremental authoritative update to
`STATUS/KCORE-FEATURE-STATE-V1506.MD`: no new process state or
persistent data RAM; newly accepted **native** owner-specific
failed-BACKSTORE retry scheduling. KCORE feature completion only.

## Native retry gate accepted

The source had a global 0100-tick delay for allocator pressure but
no retry delay for a refused `proc_swap_in`. A shared delay on fatal
errors would impede healthy swapped jobs; two failed jobs must also
not exchange immediate failure requests indefinitely.

Production v1507 writes a transient retry tag (LH bit 040000)
and the low-17-bit PI6 clock epoch into the owning stable u-area
word 067, *only while FINALIZE is clear*. The three-word process
descriptor and its RUN/DISK state remain unchanged; the authoritative
backing run, private executive continuation and pending TERM survive.
Normal and realtime selection omit only that tagged DISK owner
for 0100 octal clock ticks. Other swapped jobs are not blocked.
Expired tags are retired by the selector; successful direct restore
and successful swapped-pair exchange retire them too.

Existing global `proc_swap_pressure_ticks` remains the allocation
pressure mechanism, not a fatal-failure quarantine. In particular,
no separate STOP/QUARANTINE process state, per-slot array, new
resident RAM word or unsafe remote resource teardown is added.

Testkit `daimos-backstore-retry-scheduler-v1507.sh` joins
production PDP-6 slot-0 service, checked bitmap verifier/free,
swap-in, complete normal/realtime selection and PI6 clock.
Two distinct failing owners, healthy swapped and resident bystanders,
global-pressure compatibility, real 0100-tick deadline and 17-bit
clock-wrap are accepted with three executable negative mutations.
v1493/v1505 (six forced corrupt-refusal/repair requests) and
v1316/v1317 native checked-MM/rollback and three unsafe mutants
also PASS. Native v788 checked exchange publication and v854
full sector/PURE exchange validate retry-tag retirement and PASS.
One fresh isolated full 256K KCC PDP-6 boot image
build succeeded.

These are **native and build gates**, NOT a booted permanent-media
fault acceptance. An unrepairable DISK allocation still cannot be
safely released without a new verified recovery operation. No
automatic OOM victim killing or declared terminal owner reclaim.

## Remaining ordered KCORE work

1. Run a sustained physical BACKSTORE/DEVICE failure with real
   scheduler clock intervals, non-failing swapped and runnable
   bystanders, exact preserved DISK/bitmap/MM/u-area/TERM ownership,
   and the actual recovery or long-lived owned-failure outcome.
2. Complete mixed DEVICE/VOLUME, DTFS tape-root, BCACHE,
   LOGSTORE/MEMFS and provider error integration gates.
3. Integrate F1 uncompressed swapless PURE text reclaim and precise
   executable backing, mutable suffix, MM/APR restore and rollback;
   do not ship the earlier +180-word isolated candidate.
4. Complete F3 dynamic FSINFO enumeration up to 26 mounts, including
   the still-in-progress v1492 local branch and physical profiles.
5. Evaluate compression only after PURE with matched resident-RAM
   measurements; finish remaining KINIT/clock/device profiles,
   then undertake KCORE final size optimization.

The previously accepted v1506 **repaired** physical owner-return
TERM path is unchanged. 32K HOLD; VERSION 0.5. MRES unload/reload
POST-1.0 and unified TSFS/online D6FS packer separate scope.

Evidence and source inventory:
`DAIMOS/design/kcore-backstore-retry-scheduler-v1507.md`.
