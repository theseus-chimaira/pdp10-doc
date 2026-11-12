# DAIMOS KCORE feature state, v1510

Date: 2026-11-12. Authoritative supplement to
`STATUS/KCORE-FEATURE-STATE-V1509.MD`. Feature acceptance, not
optimization or MRES scope. Production KCORE stays at v1507,
without new process state or permanent RAM allocation.

## Independent swapped contender: physical PASS

The full disposable 256K KCC PDP-6 F07 guest preserved the
original selected permanently failing DISK owner in octal slot
`010`, BACKSTORE first `02350`, count `0330`, through thirteen
physical fault-on snapshots (25..37). Its record `002350000330`,
bitmap-backed accounting, stable u-area, retry marker and pending
TERM were retained with 1..24 refused checked frees and no
false RAM/READY_ONCE, repair, fatal close or zombie.

The independently delayed real F07 child in slot `011` was
physically `SLEEP/DISK` at samples 25..31; it restored by a real
checked `proc_swap_mres_service_v481` success between samples 31
and 32 and then executed **331 real SLEEP syscalls** by sample 37.
The alternative checked exchange witness remained zero. Resident
PID 5 separately progressed from 42 to 770 SLEEP entries. Real
PI6 clock advanced from 36273 to 53794 decimal during the fault.

Strict inherited v1508-v1510 physical verifiers PASSed. Eight
incorrect disposable source variants and eleven false evidence
variants were rejected. An early v1510 verifier mistake requiring
slot `011` before RUN admission was fixed without weakening any
*fault-on* requirement. See testkit revisions `0de641c`,
`d79991e`, `0808309`; exact evidence and SHA256s are in DAIMOS
`design/kcore-backstore-swapped-progress-physical-v1510.md`.

The guest was deliberately stopped after the 37th sample; this is
not full F07 HALT or the release of the unrecoverable owner.
**Both resident and swapped bystander physical scheduler fairness
are now accepted.** No new quarantine state or resident KCORE RAM.

## Still open

1. Real storage DEVICE/media controller failure, timeout, failure
   propagation, provider recovery and safe long-lived backing
   ownership without phantom free or zombie.
2. Mixed DEVICE/VOLUME, BCACHE/DTFS tape-root, LOGSTORE/MEMFS
   failure and recovery paths.
3. F1 swapless PURE-text/MM/APR integration with checked ownership.
4. F3 dynamic FSINFO mount enumeration to 26 mount slots.
5. Only after features are accepted: optional compression with
   measured net RAM savings and final KCORE size optimization.

VERSION 0.5, strict 32K HOLD. MRES runtime unload/reload POST-1.0;
unified TSFS and online D6FS packer outside this KCORE scope.
