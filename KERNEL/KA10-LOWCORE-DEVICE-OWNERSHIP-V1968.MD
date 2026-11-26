# KA10 low-core device ownership (v1968)

Addresses are octal, counts are decimal unless stated otherwise.
The KA10 0.9 target uses ONESEG, MAOFF disabled. This audit covers all
DAIMOS drivers and all additional devices listed by the installed
pdp10-ka. Sources: `system/kernel/drivers`, `modules/module_io.s`,
`modules/module_minit.c`, `core/kcore_pi_pdp6.s`, `system/stand/pdp6`,
and simulator `PDP10` device implementations at abb7ee6. This is a
source-level ownership audit, not functional qualification of every device.

## Fixed low core

| Address | Ownership |
|---|---|
| 000..017 | Accumulators |
| 020..036 | PI AC saves |
| 037 | No active owner found beyond initialization; not an aligned pair |
| 040..041 | Monitor UUO / transient Stage1 handoff |
| 042..057 | PI vectors and AC saves; PI3 and PI7 special below |
| 060..061 | KA10 unassigned-instruction trap; PDP-6 APR and idle stack |
| 062..146 | Fixed 53-word shared decoder |
| 147 | KA10 APR shadow (moved from BSS) |
| 150 | KA10 DF10 channel initial control word |
| 151 | KA10 DF10 terminating control word / completion writeback |
| 152 onward | Remaining KA10 KCORE |

`arch/ka10_lowcore_v1968.s` is appended to the KCORE decoder object only
on KA10. The link rejects any movement of 147, 150 or 151. The decoder
end symbol is still 147 (exclusive). Stage1 does not contain the three
extra words. All three are initialized zero by the KCORE image copy;
only the APR shadow becomes live before any RM10 driver is installed.
The pair is permanently allocated even without an RM10. It costs two
additional fixed words, no alignment padding or per-process space.
PDP-6 images are byte-identical to v1963. KCORE 11122 words on KA10,
11109 on PDP-6. Both native KA10 roots reach INIT; the APR shadow
matches RL/PL and the unused channel words remain zero.

## Existing DAIMOS drivers

| Driver | I/O mechanism and vector ownership |
|---|---|
| CTY | Programmed DATAI/DATAO, ordinary PI4 |
| APR/CLK | Processor status, ordinary PI6 |
| PCLK / software clock | Programmed registers / software state, no special memory |
| PTR, PTP, LPT, CR, CP | Polled programmed I/O; no special runtime low-core words |
| DCS, GE | Programmed character I/O, ordinary PI2 / PI4 |
| TTY | Software dispatch, no hardware DMA |
| DPY (Type 340) | PI7-exclusive BLKO 056 and completion JSR 057; relocatable MRES IOWD |
| WCNSLS, OCNSLS | Programmed I/O to accumulators and caller buffers |
| DTC, MTC, DSK270 | Shared Type 136 programmed channel, PI3 046/047, relocatable storage_iowd |
| Type 167/236 | Autonomous DMA via programmed descriptor; ordinary PI2, PDP-6 only |
| SLV | Shared memory and ordinary interrupt; PI7 or PI6 if DPY installed |

The DCT is one shared channel, not three independent I/O engines.
Its current owner must be quiesced before changing vector 046 or the
IOWD. DPY's dedicated PI7 vectors must not be assigned to any other
ordinary interrupt source. Existing DCS/GE and Type167/236 support is
not a claim of KA10 compatibility. No other existing driver needs a
new fixed low-memory command-list allocation.

## All additional devices in pdp10-ka

| SIMH device | Source | Physical memory requirements |
|---|---|---|
| FHA (RC10/RM10/RD10) | kx10_rc.c, kx10_df.c | DF10 pair; RM10 required for 0.9 |
| DPA, DPB (RP10) | kx10_dp.c, kx10_df.c | DF10 pair per active channel |
| MTA (TM10A/B) | kx10_mt.c, kx10_df.c | TM10B uses DF10 pair; TM10A programmed I/O |
| FSA, RPA/RPB/RPC/RPD, TUA | kx10_rs.c, kx10_rp.c, kx10_tu.c, kx10_rh.c | RH10 pair per active controller |
| PMP | ka10_pmp.c | Programmable physical DMA address and count; no fixed pair |
| DDC (RES-10) | kx10_ddc.c | Internal command queue; physical address per transfer; no fixed pair |
| AI (Systems Concepts DC-10) | ka10_ai.c | Programmable channel program and buffers, hardware writes status/results; no fixed pair |
| DPK | ka10_dpk.c | Programmable 32-word table (16 ports, count and pointer per port); hardware updates table and reads output data |
| III | ka10_iii.c | Programmable display program, reads and writes physical memory; no fixed pair |
| DD | ka10_dd.c | Programmable display program reads physical memory; no fixed pair |
| DT (TD10) | kx10_dt.c | Programmed data I/O; ordinary PI; no DF10 pair |
| DC (DC10) | kx10_dc.c | Programmed serial I/O |
| DK, PD | kx10_dk.c, ka10_pd.c | Programmable timer and DeCoriolis clock; ordinary PI |
| TV, VDS | ka10_tv.c, ka10_dd.c | Programmed camera/buttons and video switching |
| IMP, CH | kx10_imp.c, ka10_ch10.c | Native KA10 programmed packet I/O; no fixed DMA workspace |
| IMX | ka10_imx.c | Programmed analog input |
| STK, DKB | ka10_stk.c, ka10_dkb.c | Programmed keyboard input |
| TK, MTY | ka10_tk10.c, ka10_mty.c | Programmed terminal multiplexing |
| TEN11 | ka10_ten11.c, kx10_cpu.c | ITS mapped remote PDP-11 memory, default 03000000..03777777 |
| AUXCPU | ka10_auxcpu.c, kx10_cpu.c | ITS mapped shared auxiliary CPU memory, default 03000000..03037777, interprocessor PI |

DF10 uses `ICWA=0776` and writes completion to `cia|1`; RH10 on KA10
uses the same mask and writeback location. Their initial control words
can link to ordinary allocated command lists. The simple RM10 two-word
list uses the odd word as terminator before hardware completion.
The reserved pair may be reused by future channels only with exclusive
ownership through acknowledged completion or verified abort/quiescence.
A second independently active channel needs a separate even pair. No
pool, allocator, additional pairs or runtime arbitration across future
controllers is implemented now. The sole K17 consumer will be RM10.
Other DMA programs/tables belong in pinned, validated physical extents
owned by the corresponding MRES or request. They must not refer to
unrelated KCORE, freed buffers or user logical addresses. In particular,
III can write arbitrary addressed memory, and DPK updates its descriptors.
Several models directly index simulator memory; NXM and bounds behavior
need native testing before claiming driver support. No untested simulator
defect is fixed as part of this audit.

TEN11 and AUXCPU are not ordinary KA10 DMA peripherals. Their mapped
remote ranges are dispatched by `Mem_read_its`/`Mem_write_its`; they
require the ITS memory mapping path, outside DAIMOS ONESEG. They also
have overlapping default bases. No permanent low-core space should be
reserved for either.

## Device number conflicts (not memory addresses)

- RH10 assignment order is 270,274,360,364,370,374; its first controller
  conflicts with DSK270 at 270. Keep RH10 disabled for K17.
- DK and STK both use 070.
- PMP and PD both use 500 by default; PMP has a configurable device number.
- VDS and MTA both use 340.
- MTY uses 400; WAITS IMP mode also uses 400.
- AUXCPU uses 020, the PDP-6 SLV number; SLV is not compiled into KA10.

These are supported configuration choices, not grounds for adding
emulator compatibility devices. Additional controllers remain post-0.9.

## Verification

The native read/write isolation fixture loads the actual linked KCORE
and sentinel-filled BSS, then uses the new 150/151 pair for a 128-word
RC10 transfer. Every other one of the 11120 resident KCORE words is
unchanged in both directions. Completion is 000151102177 for buffer
102000..102177. All five Stage1 decoder identities and image embedding
checks pass on both KA10 root profiles and the PDP-6 build. This is
not yet a live RM10 driver, concurrent controller, or swap test.
