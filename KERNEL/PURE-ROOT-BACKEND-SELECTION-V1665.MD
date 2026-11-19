# v1665: install one PDP-6 memory service after actual root selection

Date: 2026-11-18. Parent DAIMOS v1664:
`894615ddd040bf24631c5a8eb733051a48954f1e`.

## Rationale

Physical PDP-6 RAM is scarce, and disk-block swapping and a
swapless TSFS PURE suffix owner require different release and
restore policies. Keeping the two policies permanently resident
would waste words and introduce dangerous compatibility branches
in KCORE. Fortunately, DAIMOS already has a single service-entry
pair (`proc_swap_reclaim`, `proc_swap_service_one`) patched at
boot, and installs only one MRES package through transient KINIT
relocation. v1665 corrects **when and how that choice is made**.

Previously `proc_swap_minit_v481()` ran before root discovery and
treated D6FS compilation as a proxy for the actual root. A
ROOT=auto kernel with both D6FS and TSFS support therefore could
not select optional PURE even when it booted from DECtape with no
validated BACKSTORE. Physical DSK/DRM hardware may be present
only as auxiliary storage; hardware presence alone does not
prove usable process-swap backing.

The MINIT table now orders installed physical drivers first,
then `root_select_minit`, then `proc_swap_minit_v481`, then
`auxstore_minit`. The swap MINIT reads `root_select_class()`:

* Selected DTC/TSFS with absent/invalid AUXST1, or valid AUXST1
  containing zero BACKSTORE: install optional PURE if enabled,
  otherwise no swap service.
* Selected DTC/TSFS with **positively validated nonzero**
  BACKSTORE: install original disk-swap MRES and publish its seven
  existing exports.
* Selected DSK/DRM root with support compiled: install original
  disk-swap MRES without TSFS-only AUXSTORE discovery.
* Unselected or unsupported root: install no memory backend.

The PURE choice patches the same two fixed scheduler trampolines
already used by disk swap. KCORE receives no permanent runtime
root test, backend table or extra process field. ROOT=auto now
includes the PURE candidate when `DAIMOS_PURE_MRES_ENABLED=1`
and `KINIT_TAPE_BOOT=1`; ROOT=disk/drum does not. This only
changes transient package inclusion: a general auto-root image
must still carry both candidate MRES packages until KINIT chooses
one. The unchosen candidate is NOT installed as permanent MRES.

## Evidence

The testkit source-extracts the actual production KINIT selector,
compiles six strict C89 feature combinations, and checks selected
root class, successful/absent/zero BACKSTORE, exact MRES export
patches and exclusive installation. It validates the actual
MINIT order and the top-level Makefile candidate set for six
ROOT/PURE combinations. Inverse MINIT order and wrong-root
selection are independently executed mutations and rejected.
The original v1613 selector regression (including an unsafe
AUXST1 discovery mutation) passes.

The full source-built KCC ROOT=auto PURE=1 kernel links both
transient MRES candidates successfully, with `__kinit_image_end`
octal `140154` and fixed `__kcore_low_end` octal `024535`.
Current KCC FULL PURE package costs 2,217 installed MRES words;
disk-swap package costs 1,795 including 128 BSS, but these are
ALTERNATIVES, not additive permanent residency. These measured
values belong to the current image, not a normalized comparison
to historic releases.

An actual source-built 64K PDP-6 with GCC lowmem ROOT=tape,
three-member TSFS and no attached auxiliary disk/drum media
boots successfully: console witnesses `MEM 64 K`, `DTC OK`,
`PURE LOADED`, `INIT V1`. There is no `?RT` or INIT RUN failure.
The GCC fixed `__kcore_low_end` is octal `024255`, KINIT end
octal `113655`, and PURE package 1,773 installed words. The old
v1613 physical script hardcoded `024254` and thus fails BEFORE
the simulator; the new versioned v1665 physical runner checks
the current bound and successfully reaches INIT. Retain both
results: neither implies a new fixed-KCORE regression from v1665.

## Further RAM savings

v1665 does not remove the unused MRES **candidate** from a
general KINIT image, does not link compressed mode3 into PURE,
and does not alter disk-swap release or fixed `proc_swap_detach`.
Next move terminal release behind a patched backend entry so
the fixed kernel does not carry both process-owner scans. Then
integrate the v1659..v1664 bitmap-only memory path into the PURE
module, not the disk module. A dedicated `TSFS swapless-only`
build profile should omit the transient disk-swap package and
fail closed if a positive BACKSTORE is found. Such a specialized
image is NOT equivalent to general ROOT=auto, which must retain
the ability to choose either service. Measure KINIT peak and net
permanent MRES words separately in physical 48K/64K guests;
strict 32K deferred. PURE default OFF, mode3 production OFF.
