# DLINK unused EXTERN and P10RUN SIMH expiry, v1812

Date: 2026-11-22. Host source correctness accepted with a source-linked
testkit matrix. Native DAIMOS DLINK has only compilation/link validation;
actual guest execution is still open.

## Undefined declarations versus relocations

DOBJ1 retains UNDEF symbols in its table, including EXTERN declarations
that are not referenced by the emitted code. A name with no symbol
relocation does not require archive extraction or a definition. Both
the standalone pdp10-tools DLINK and DAIMOS's host DLINK use the
existing DOBJ1 relocation table to test this, including both symbol
RH18 and LH18 relocation types. No resident index is allocated.

The native DAIMOS DLINK is file-backed. It reads each object's
relocations sequentially and builds a three-word temporary bitmap of
used symbol indexes; this avoids repeated filesystem scans for every
undefined declaration and adds no BSS allocation. Each relocation's
symbol index is one-based. Malformed referenced indexes are rejected.

## Simulator expiry must be a failure

P10RUN used to accept a stale initialized zero exit word after the
simulator exhausted a step or time budget. It now rejects `Step expired`
and `Time expired` as failures even when subsequent memory examination
matches the expected status. Successful execution with zero exit remains
accepted, as do existing error and low-core breakpoint refusals.

## Reproduction and test boundary

The versioned suite is in
`daimos-testkit/tests/host/toolchain-extern-expiry-v1812/`.
It builds the exact selected host sources, creates real DOBJ1 objects
with DAS, and compares direct and DARC archive links for only-needed
provider extraction. It also checks absent required providers and
both relocation halves. Direct P10RUN source evaluation uses completed,
step-expired, time-expired and unchanged-zero transcripts.

The old DAIMOS host linker fails the unreferenced EXTERN control; the
old P10RUN misclassifies both expiry controls as PASS. The corrected
DAIMOS and standalone host tools pass. Native KCC/DAS built the
file-backed DLINK target, and the production source linked into a DXR.
An independent bare-machine PDP-6 SIMH check using current P10RUN
source passes a completed GO guest and rejects a forced one-step stop.
The original P10RUN source falsely passed that same one-step guest;
this was verified by rebuilding the earlier source as a control.
The fresh default-OFF DAIMOS source build also passes.
Its DOBJ text grew 2650 to 2800 words, with BSS unchanged at 8438;
this is a userland executable, not fixed KCORE growth. An actual native
guest run and full DAIMOS system regression remain pending; no K05-K09
physical acceptance or RAW PURE enablement is implied.

The DAIMOS source-linked implementation report is
`design/toolchain-extern-expiry-v1812.md` in the DAIMOS repository.
