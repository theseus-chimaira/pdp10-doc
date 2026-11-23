# PDP-6 K05: real secondary DTC3 FULL rollback and later PI6 boundary

Date: 2026-11-23. Status: **scoped physical PASS; K05 OPEN**.
DAIMOS default build keeps PURE and optional mode3 OFF.

## Physical source and owner proof

DAIMOS v1839 integrated RAW PURE FULL restoration has a single
authoritative PACKED suffix owner during source-backed reconstruction.
The source-linked v1845 testkit boots a current-source 48K PDP-6
TSFS-root system with one root-backed and four secondary DTC3-backed
compressed PURE children. The sixth actual RUN under pressure fails,
while the first five have completed admission. A deliberately
disconnected DTC3 causes the original bounded FULL loader to return
-4. At loader entry, slot 3 owns pinned PACKED `111200..120000` and
pinned PROCESS `120000..130000` octal. The temporary PROCESS owner
is freed on failure, and the exact RAW suffix remains with zero pins.
The selected TSFS root is unchanged and no child receives a runnable
APR until a complete reconstruction commits.

The v1847 testkit performs the same accepted original physical
transaction but observes the real fixed KCORE PCLK PI6 entry before
reattaching the secondary DTC3 reel. The installed physical PC is
017205 octal, and the actual instruction word is 350040024510
(`AOS 1,proc_timer_clock`). Both are derived from original
`proc_pdp6.s` and the source-linked fixed KCORE symbol map; the
physical simulator independently reads the instruction and timer.
Clock, PURE cooldown and scheduler cursor at that interrupt entry are
021431, 0100 and 3 octal, respectively. The scheduler remains in
slot 0; all original MM extents, owners, pins and TSFS mount identity
are unchanged from the post-FULL-failure rollback. Only after this
clock checkpoint does the test return the exact same private DTC3
reel. All five original PIPE/writable-data witnesses, EXIT/WAIT and
normal HALT complete. The new physical verifier runs the complete
v1845 original owner/loader/source audit as a prerequisite and rejects
independently corrupted PI6 opcode and PACKED pin transcripts.

Image SHA256 values (identical to current-source v1845):

* Fixed KCORE: `33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`
* Opt-in PURE MRES: `ad246ce14079a821cd69f4c4f7b7661f5d1a0f58c118bdfa5cfa7adf2b2bfe3f`
* v1847 physical trace: `67e462afa59f3e760440b01d6e9b2057a0a4e89147fb59918c73d740b62843b1`

## Diagnostic exclusions and remaining requirements

An earlier attempt to wait for a second FULL callback at an old PURE
MRES address did not establish a second transaction. Its observed
registers had zero target/capacity and MM lacked a second pinned FULL
APR. Removable MRES code locations cannot be treated as permanent
identities across subsequent scheduler activity. Another preliminary
test-oracle mistake interpreted raw KCORE DXR IMAGE offsets as
installed physical KCORE addresses. The accepted v1847 test instead
authenticates its PI6 instruction from the original assembler and
linked installed address; the unmodified physical trace was reaudited
and both independently mutated witnesses were rejected.

The verified interrupt is **after** the FULL rollback. It does NOT
show PI6 interrupting the pinned FULL MM transaction itself, nor
permanently corrupt pins, owner uncertainty, arbitrary source loss,
continuous original physical MM high-water, 32K/KA10 acceptance, or
mode3 production readiness. K05-K09 remain OPEN; K01-K04 CLOSED.
The production MM owner/quarantine rule remains in
`KERNEL/PURE-MM-UNCERTAIN-V1658.MD`.
