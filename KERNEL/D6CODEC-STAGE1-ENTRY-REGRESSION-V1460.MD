# v1460: correct DTC/MTC Stage-1 KINIT entry register without code growth

Date: 2026-11-10. Status: **corrected and native/full cold-boot tested**.

This note supersedes the DTC/MTC early-boot timeout state in
`KERNEL/D6CODEC-BLOCK-CUTOVER-INTEGRATION-V1459.MD`. The decoder
core, VFS adapter and v1421 block compressed format are unchanged.

## Fault

Both DECtape Type-136 (DTC) and Type-516 magnetic tape (MTC)
Stage-1 loaders read a compressed KINIT header containing the
expanded image size and an entry offset. They added the load
base (octal 030000) and retained the resulting absolute entry
address in AC5. They then passed the compressed payload to the
fixed block decoder and finally jumped through `JRST 0(05)`.

The block decoder uses AC5 on its match path as a temporary
back-reference source address:

    movni 5,1(7)
    add   5,12

Thus any final match changes the value used for KINIT's entry
address. In a real failed boot, the header specified octal
`053537`, but the end-of-decode register snapshot showed AC5
equal to octal `120366`. The decoder completed the declared
payload and output; the loader then executed a jump into the
expanded data, eventually returning to the low-core paper-
tape bootstrap loop at octal `000024`/`000027`. No KINIT banner
was printed. The earlier v1452 tape loaders had the same
register-lifetime defect.

## Fix

Retain the validated KINIT entry address in AC16 instead of AC5:

    hrrz 016,03
    caml 016,013
    jrst bad_tape
    addi 016,030000

At successful handoff, use `JRST 0(016)`. The fixed 53-word
block decoder does not reference AC16. DTC's ordinary compressed,
raw and TSFS-hybrid paths use the same preserved entry register;
MTC's compressed path does likewise. No new registers are
allocated on the stack and no static word is introduced.

This corrects the source of the error; it does not add a retry,
delay, blind jump or simulator-specific conditional.

## Native size verification

The testkit assembles pristine v1459 and corrected v1460
Stage-1 sources with **identical** fixed-decoder includes and
compares every physical read-in word:

| Device | Physical words, old and new | Altered words | Growth |
|---|---:|---:|---:|
| DTC | 160 | 5 | 0 |
| MTC | 125 | 4 | 0 |

Only the expected register-reference instructions differ.
KCORE's fixed block decoder remains **53 words**; the VFS
adapter remains **58 words**, and the linked KCORE image
remains **9,615 initialized words**, with no BSS change.

## Real simulator acceptance

New isolated full-profile 256K PDP-6 boot tests now pass for
both previously failing tape Stage-1 devices. Each test
required the KINIT banner, ROOT login, DSH startup, and an
executed guest ECHO command. These add to the previous DSK,
DRM and PTR successful cold boots, completing the five-device
**disk-root** Stage-1 boot-source matrix.

The separate TSFS-root `INIT: RUN FAILED` result and strict
32K/48K lowmem gates are not solved or reclassified by this
Stage-1 correction. They require independent acceptance.

Reproducible testkit entry:
`daimos-testkit/tests/validation/d6codec-v1460-stage1/run-v1460.sh`
and its register-liveness test and SIMH snapshot utility.
