# v1619: physical DECtape reverse-read completion

Date: 2026-11-16.
Machine: real SIMH PDP-6 48K,
three-member TSFS root, no AUXST1.
PURE MRES: experimental, default OFF.

The direct physical controller test
uses the generated TSFS DTC0 root image,
not a synthetic tape with known blocks.
Its read-only private copy passes
both near block-4 reading and
reading block 0341 then reversing
to block 4, using the original
Type-551/Type-136 driver and PI3/PI5.
The 128-word read buffer is nonzero;
tape images remain byte-identical.

The original full KCORE PURE workload
also reached physical breakpoint
031046 (Type-551 RBN match) with
unit=0, direction=reverse, requested
block 000004 and last mark 000004.
It then reached 031106 (PI3 read
command) and 030555 (Type-136
DCT read completion) in the same run.
At the DCT boundary, storage_state
was -1 and storage_count was 0200
octal. The following event publication
was not yet executed at that stop.

The DCT completion branch was verified
using a temporary test-only assembler
symbol export, and every original
linked tape image instruction remained
unchanged. Both an incorrect RBN
mark and a corrupted completion-state
test mutation were rejected.

Important diagnostic correction:
tape_pi_done on PI5 is not the normal
Type-551 read completion; that read
finishes through tape_dct_done_keep_dtc.
This required correcting the test,
not the driver. No production
code, KCORE, or MRES growth.

The next gate remains physical
PURE-prefix VFS completion and
the actual MM commit or rollback,
release of the PACKED owner pin,
READY_ONCE, PIPE/WAIT/EXIT and
exact net RAM acceptance.
Do not interpret an RBN/DCT
success as completed process RUN.

Authoritative source:
DAIMOS design/kcore-p1e-physical-dct-read-v1619.md.
