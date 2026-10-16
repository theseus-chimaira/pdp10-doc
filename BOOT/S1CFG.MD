# S1CFG - POST-0.9 BOOT POLICY DESIGN

STATUS: DEFERRED DESIGN, NOT A CURRENT BOOT ABI.

S1CFG IS RESERVED FOR A POSSIBLE POST-0.9 POLICY LAYER.  DAIMOS V0.9 DOES NOT
IMPLEMENT AN S1CFG RECORD, `S1CFG_V1.H`, BOOTINFO EXTENSION, DMANIF STREAM, OR
SEPARATE KRES/INIT STREAM.

## CURRENT V0.9 BOOT CONTRACT

THE CURRENT PDP-6 STAGE1 LOADERS USE THE SAME COMPRESSED DAIMON IMAGE CONTRACT.
DSK AND DRM ADD DBOOT MEMBER discovery; DTC, MTC, AND PTR ARE SEQUENTIAL
TRANSPORTS:

```TEXT
STAGE0 / RIM ENTRY
  -> DEVICE STAGE1
  -> DISCOVER OR RECEIVE THE DEVICE-SPECIFIC BOOT STREAM
  -> READ THE THREE-WORD DAIMON STREAM HEADER
  -> LOAD RAW LOWMEM IMAGE OR STAGE D6LZ36 PAYLOAD
  -> EXPAND THE COMPLETE IMAGE AT 030000 WHEN COMPRESSED
  -> ENTER KINIT AT 030000 + ENTRY_OFFSET
```

STAGE1 PASSES ONLY THE COMPACT TWO-WORD MEMBER-LOCATOR HANDOFF IN LOW CORE.
KINIT OWNS ROOT DISCOVERY, MODULE INITIALIZATION, FILESYSTEM MOUNTING, AND
NORMAL BOOT POLICY.

THE V0.9 ROOT CONTROLLER CLASS IS SELECTED BY THE LOW CONSOLE-SWITCH BITS.
AUTO PREFERS DSK, THEN DTC, THEN DRM.  KINIT DOES THE ACTUAL ROOT SET
DISCOVERY.  RAM ROOT IS RESERVED FOR A FUTURE INSTALLATION MODE.

## WHY S1CFG IS DEFERRED

THE PRESENT MECHANISM IS DELIBERATELY SMALL.  ADDING A GENERAL POLICY RECORD
BEFORE V0.9 WOULD CONSUME BOOT FORMAT, TEST, AND RESIDENT complexity without
closing a current requirement.  OPTIONAL MOUNT, SWAP, AND LOGSTORE POLICY IS
TO MOVE OUT OF KINIT INTO INIT AND `/CONFIG`; it is not a reason to grow
Stage1.

## POST-0.9 REQUIREMENTS IF S1CFG IS REVIVED

A FUTURE S1CFG DESIGN MUST:

- leave raw controller I/O and media geometry in Stage1;
- leave root filesystem discovery and mount policy above Stage1;
- preserve the existing DBC/DB0/DB1/DBX compatibility contract where useful;
- avoid a large copied boot-information structure;
- define versioning, bounds, checksums, and failure behavior before code lands;
- work for DSK, DRM, and DTC without embedding filesystem-specific code in
  Stage1;
- coexist with the runtime movable-module loader rather than inventing a
  second boot-only module format;
- justify every permanent or low-core word on the PDP-6 profile.

NO CURRENT SOURCE FILE OR BUILD TARGET SHOULD BE DESCRIBED AS IMPLEMENTING
S1CFG UNTIL THIS DEFERRED DESIGN IS explicitly revived.

## VERIFIED SIMH PAPER-TAPE/TSFS BOOT PATH (2026-10-16)

An isolated full-profile `make image ROOT=tape BOOT=ptr` generated the
Stage-1 PTR read-in tape, two counted compressed KINIT payload tapes,
and one TSFS root member (`tsfs-root0.dta`) attached as SIMH `dtc0.tap`.
The SIMH PDP-6 boot script uses `SW=2`, loads Stage-1 via the historical
low-core RIM reader, switches to the first payload and then automatically
mounts the second when the loader prints `TAPE2`. A bounded 85-second run
reported `DTC OK`, `DTFS LOADED`, `INIT V1`, and `LOGIN:`. This proves
startup through the login prompt, not a completed interactive session.

For an AAP comparison, the *same linked KINIT compressed image* was placed
on its own two paper-tape payloads. AAP completed both payload transfers
and executed KINIT's linked entry (`054540`) but later HALTed at
`kinit_halt` (`072040`) after 37 observed PI cycles. This does not yet
establish an AAP CPU fault: the AAP Type-555 DECtape model consumes raw
mark/data cells, whereas SIMH's attached TSFS root is stored in a
32-bit-halfword `.tap` container. The AAP comparison therefore did **not**
include the same mounted root medium. The AAP functional/timing gate remains
18/19 until equivalent TSFS media and clean ordinary-tick intervals are
verified. See DAIMOS `PROJECT-STATE.MD` for the build-specific evidence.
