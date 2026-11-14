# KCORE v1571: Type-551 silent-exchange negative physical proof

Date: 2026-11-14. **Two real 256K PDP-6 production guest boots PASS
the NEGATIVE oracle.** P0-B OPEN, DTVOL2 production READ-ONLY.

Two independent full 578-block, 128-word-per-block DTVOL2/CLEAN
images were freshly formatted on the host, with complete identity
complement, two different nonzero 36-bit ID halves, salted CLEAN
seal and zero padding checked BEFORE the guest. No V2 writable
test exception was compiled. The unchanged production v1555 gate
first refused the RW mount; DTC3 mounted cleanly READ-ONLY on the
original reel and FSINFO/STAT succeeded.

While real PID1 blocked in CTY READCHAR and issued no DTC3 I/O,
the host paused the real Type-551 simulator and executed DETACH
and ATTACH of the *different* valid CLEAN DTC3 reel in one
uninterrupted monitor episode. The guest made no read/write
request while unattached, and no DTC3 failure/READ_SUSPECT existed.
After resuming, the original VFS mount continued to accept FSINFO,
STAT and CHDIR and could UNMOUNT. It had NOT validated the actual
new physical RBN0 against its saved old 72-bit ID. The guest
reported `V2EXCHANGE V1571 BLIND_SPOT_CONFIRMED` and
`PASS_NEGATIVE` in both cold boots.

Separate complete-tape SHA-256 checks verified neither original
nor replacement was written or damaged. The established v1517
Type-551 SIMH CONI select/attached/detached/absent test was rerun
and PASSed; current availability cannot signal historical reel
replacement. In the TU55 original operating manual (1966,
chapter 3.3), removal requires setting REMOTE/OFF/LOCAL to OFF
to release brakes and then returning the switch to REMOTE/LOCAL;
there is no documented atomic reel-generation/write interlock
available to the current PDP-6 Type-551 driver.

This is an intentional fail-to-detect witness, not a safety
acceptance. Existing v1527 recovery requires an OBSERVED
physical read error, after which RBN0 is reread. If no request
fails, current controller/DTFS cannot infer a prior exchange.
Similarly, v1554's mandatory RBN0 check before V2 DATA/DIR write
does not atomically bind the subsequent physical transfer to
that checked reel. A genuine mechanical retention and interlocked
write path (or explicitly degraded operator-trust contract) must
be decided and independently accepted before any V2 RW cutover.
No fake software-only mount flag, extra RBN0 poll or simulator
`UNIT_ATT` bit is sufficient.

No production KCORE, physical driver, optional DTFS MRES,
mount-table RAM, ABI, fixed TEXT/DATA/BSS or footprint changed;
`__kcore_low_end` remains `024364` octal. P0-A CLOSED,
P1-C/D/E unchanged, VERSION 0.5 and strict 32K HOLD.

Primary design: `DAIMOS/design/kcore-dtfs-media-retention-v1571.md`.
Physical proof: `daimos-testkit/tests/validation/daimos-dtfs-v2-silent-exchange-v1571.md`.
