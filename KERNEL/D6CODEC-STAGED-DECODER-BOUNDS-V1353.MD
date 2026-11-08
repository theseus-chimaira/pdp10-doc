# v1353: native PDP-6 staged grouped-decoder bounds

Date: 2026-11-08. Scope: completed validation and staged-source repair,
not production grouped-codec format activation.

## Verified implementation

The DAIMOS `d6codec_decode_v053.s` staged decoder now validates all
source and destination pointer/count geometry in the physical 18-bit
word-address space before reading or writing. Nonempty overlapping
source and destination ranges are refused. Unused group-control flag
bits and an unused right halfword in a final odd descriptor count are
required to be zero. Invalid buffers can leave partial output when
the error is found after decoding starts; callers must publish data
only after an exact complete success.

The 18-bit address-space endpoint is the *full-word* octal 01000000.
Using `CAILE` with that literal silently loses its nineteenth bit;
the v1353 implementation uses a full-word comparison. The initial
version of this check was rejected by the native tests and corrected.

## Acceptance and negative witness

- Testkit commit `97e222d` on
  `test-d6codec-bounds-v1353-20261108`: native
  `tests/validation/d6codec-bounds-v1353/run-v1353.sh` PASS.
- Existing staged `d6codec-kcore-abi-v055` native full-DXR golden
  roundtrip and saved-register test PASS with the hardened decoder.
- Running the v1353 negative fixture against unmodified DAIMOS
  `fa776f44` fails with code 1 because unused control bits were
  previously accepted. This is the expected negative witness.
- The unchanged D6LZ36 linked baseline records KCORE `023216`
  (9870 decimal) and the full optional package sum `046426`
  (19734). These are not grouped-codec linked sizes or a loaded-module
  physical RAM peak. The full package limit check still reports an
  overrun of octal 040000.

## Remaining gates

This does not establish arbitrary-input fuzz resilience, a complete
encoder capacity contract, worst-case linked stack bounds or a boot
image cutover. The existing Stage-1 resumable decoder, host producers,
EXEC/PURE and TSFS consumers still use D6LZ36. Continue from DAIMOS
`system/kernel/core/D6CODEC-INTEGRATION-TODO-v054.md` and
`design/d6codec-cutover-entry-v1352.md` in the isolated branch, with
no mixed-format production images or fallback codec.
