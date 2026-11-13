# KCORE v1538: fail stop on selected-root I/O, isolate secondary failures

Date: 2026-11-13. Current KCORE feature policy, not optimization.

## Accepted source-linked integration

KCORE no longer needs to establish *permanent physical removal* before
stopping when its selected, published root filesystem cannot complete
an otherwise valid backing read or write. The ordinary physical driver
first performs its bounded recovery; if its selected-root provider
receives a failed operation, the fixed ten-instruction entry
`kcore_root_fs_io_failure_v1538` compares the requesting provider and
public mount ID with `vfs_namespace_root`. Only the selected root
enters the accepted v1515 root-specific no-return HALT. Unrelated
physical controller events are not interpreted as root loss.

The D6FS provider guards actual failed backing cache-miss reads and
write-through commits. TSFS guards real DTC reads of FILE/EXTENT
metadata and STORED/BLOCK file-data restarts. A secondary FS returns
I/O errors without HALT, retaining FILE, mount, dirty and cache owner
rules. Invalid arguments, missing TSFS records, EOF, MM/decoder
failure and failed boot probes are not promoted to fatal physical
root I/O events. No new KCORE permanent data words or syscall ABI.

Actual PDP-6 CPU source-linked testkit v1538 passes 11 D6FS cases and
five executed negative mutations, plus 12 TSFS cases and four negative
mutations. Each fatal case checks the actual linked root HALT address
against the BACKSTORE address. The v1513, v1515, v1516, v1517, v1537,
TSFS slab and multiextent native regressions PASS. Both full 256K
KCC disk-root/D6FS and tape-root/TSFS fixed+MRES links PASS; a full
tape-root boot image has been built.

## Real physical scope and remaining P0-A

The current 256K PDP-6 v1534 real booted guest PASSes detaching
noncritical DTC3 while independent DTC4, writable MEMFS and selected
TSFS root continue to work. The separate v1538 full-guest positive
TSFS-root test also PASSes: physical DTC0/DTC1/DTC2 detachment
followed by an actual `/SYSTEM/INIT` root data read causes a native
HALT at octal `003743`, the exact root-specific instruction derived
from the current fixed KCORE map. Metadata-only STAT cache hits
were rejected as insufficient physical fault evidence.

BACKSTORE remains independently critical and is deliberately **not**
changed here. Existing v1511/v1512 transient PI/PI6 watchdog failures
retain held-owner rollback and healthy swapped-process progress.
Determining when BACKSTORE is truly unserviceable without destroying
transient recovery is open. Physical D6FS disk/drum root and isolated
single-member TSFS-root losses also remain outstanding. KCORE P0-A
as a whole is not closed.

DAIMOS: `design/kcore-root-io-failstop-v1538.md`.
Testkit: `tests/validation/daimos-root-io-failstop-v1538.md`.
VERSION 0.5. Strict 32K and general optimization HOLD.
