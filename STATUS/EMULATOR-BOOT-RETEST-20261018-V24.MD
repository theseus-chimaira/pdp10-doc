# DAIMOS PDP-6 CROSS-EMULATOR BOOT RETEST V24

Date: 2026-10-18

## Test inputs

- DAIMOS checkout bc22c27, with uncommitted Type-551/Type-136 PI3
  search changes in tape_io.s and storage_router.s.
- AAP: local pdp6/emu/pdp6_headless_v1, initially with diagnostics;
  then forcibly rebuilt from cleaned upstream AAP revision ac603e8.
- SIMH: installed $HOME/cross/bin/pdp6, which includes DPY, WCNSLS,
  and OCNSLS. The source-tree sims/BIN/pdp6 omits those devices.
- Separate 256K boot images rebuilt from this checkout for DTC/TSFS
  root and DSK/D6FS root, using diagnostic DSH_IMAGE_LIMIT=037000.
  Neither constitutes strict 32K acceptance.
- Testkit probe: probes/daimos-three-emulator-boot-v24.txt.
- Testkit AAP driver: tests/validation/daimos-aap-boot-retest-v24.py.
- Testkit SIMH device profile generator:
  tests/validation/daimos-simh-aap-hardware-v24.py.

## Results

| Configuration | Startup | External command |
| --- | --- | --- |
| AAP DTC/TSFS root | INIT, ROOT, DSH V1 | FAIL: ECHO echoed, no output/prompt within 240 seconds |
| SIMH AAP-like DTC/TSFS | DTC OK, no INIT | FAIL: incomplete Type-136 transfer |
| SIMH full DSK/D6FS, DCS0 | ROOT, DSH V1 | PASS: ECHO and PWD |
| SIMH full DSK/D6FS, fresh CTY | ROOT, DSH V1 | PASS: ECHO and PWD |

Both successful disk sessions execute ECHO RETESTED_OK_V24, observe a
separate RETESTED_OK_V24 output line, execute PWD, observe /, and
return to DSH.

## SIMH DECtape stall evidence

The AAP-like SIMH configuration disables DSK, DR, MTC, DCS, GE,
LPT, CR, CP and SLAVE. It retains CTY, PTR/PTP, Type-136 DCT,
Type-551 DTC, PCLK and graphics interfaces. The clean boot stalls
after DTC OK. A debugger snapshot recorded:

    PC                 023373         common PI return
    storage_state      777777777777   -1, DTC read
    storage_count      000000000200   128 requested words
    storage_iowd       777767107436   -9 words remaining
    PI3 DCT vector     264000023304
    transfer vector    720000024357   BLKI DCT
    DTC DTSA           220305

The IOWD proves 119 of 128 words transferred; nine are outstanding.
After resuming and stopping again, the same IOWD was observed.
The test was interrupted for debugging and terminated manually.
Follow up by checking the Type-551 DATA SYNC and Type-136
request/completion transition. Whether the remaining discrepancy
belongs to SIMH or the DAIMOS PI3 driver remains unresolved.

## Full SIMH disk boot observations

The first successful DCS0 run additionally logged INIT: RUN FAILED
on CTY; the failed login entry was not identified. A subsequent
CTY boot using the same writable media halted with ?RT, the KINIT
root-validation/mount failure code. After regenerating disk/drum
images, another CTY run passed ROOT/DSH/ECHO/PWD without the INIT
error. The correlation with media reuse does not prove causation.
Clean shutdown and repeated boot/persistence acceptance remain open.

## AAP post-login failure

A fresh AAP run reaches INIT, ROOT and DSH with the default INITTAB.
CTY receives the complete ECHO RETESTED_OK_V24 command, but the
external result and new DSH prompt do not appear within 240 seconds.
This reproduces the v23c open issue. Reaching DSH is not equivalent
to working external commands.
The second run with the rebuilt ac603e8 emulator independently reached
ROOT/DSH and timed out on the same ECHO command after 240 seconds.

## Evidence

Runtime reports: $HOME/tmp/daimos-retest-v24/.
AAP console: $HOME/tmp/aap-dtc-boot-v41/aap-boot-console-pi3-retest-v24.log.
Second AAP console:
$HOME/tmp/daimos-retest-v24/aap-clean-v25/aap-boot-console-pi3-retest-v24.log.
Successful disk reports:

    simh-disk-installed-report-v24.md
    simh-disk-cty-fresh-report-v25.md

No DAIMOS, AAP, or SIMH runtime fix was made in this retest.
