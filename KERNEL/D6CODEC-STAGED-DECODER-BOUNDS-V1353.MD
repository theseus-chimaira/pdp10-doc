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

## v1354: staged encoder geometry verification

The matching inactive v053 encoder now checks both 18-bit physical
source and maximum destination extents, rejects nonzero upper pointer
bits and overlapping nonempty source/output ranges before writing its
control header. Nonempty input with zero capacity fails without a
destination write; empty input with zero capacity returns zero.

Testkit `1d2b15c` adds native PDP-6 `d6codec-encode-bounds-v1354`:
PASS for the hardened encoder, expected negative witness code 1 for
the previous staged encoder. The full-DXR `d6codec-kcore-abi-v055`
roundtrip remains PASS. No KCORE linkage or compressed-image format
has changed, and maximum encoded-capacity arithmetic is not yet an
accepted production contract.

## v1355: native conservative encoded-buffer sizing

The staged assembler encoder now exports
`d6codec_v053_max_encoded_words`, an allocation-free helper accepting
the uncompressed word count in AC1. It returns AC1=0 for an empty
stream, otherwise a safe encoded capacity `N + ceil(N/30)`, or -1
when the count is invalid or the bound exceeds the 18-bit physical
address space. AC0, AC2 and AC3 are volatile; there is no stack or BSS.
The group payload is bounded by the source words consumed, while
each nonfinal group consumes 30 tokens, establishing the formula.

The last allowable input count under this conservative bound is
253687 decimal (0757367 octal), yielding 262144 decimal words;
253688 produces an unrepresentable bound and is rejected. This is
a capacity contract, not a requirement to allocate 262144 words.

Native testkit `d6codec-capacity-v1355` checks exact and short
capacities for 30/31 distinct literals and a one-word zero-run,
input/output preservation and helper boundary cases. Combined native
v1355/v1353/v1354/v055 regressions PASS. No production image,
resident code or packed-process behavior has changed. Caller-specific
stack and actual physical-memory high-water remain to be measured.

## v1356: staged codec ABI frame stack measurement

The v051 native full-darc DXR and stack-canary fixture was rerun with
the v055 register-saving kernel ABI adapter and current staged v1355
codec sources. The `d6codec-stack-v1356` test PASSes: `__test_exit=0`
and `__v051_first_changed=016` octal. The latter measures 14 decimal
stack words including the CRT's initial PUSHJ, the wrapper frame and
both codec engine calls. This is not a proven maximum for malformed
data, capacity faults or nested kernel consumers. Production D6LZ36
and KCORE remain unchanged.

## v1357: staged malformed/capacity failure-path stack

The independent `d6codec-stack-fail-v1357` PDP-6 native test uses
the same 512-word canary CRT and v055 saved-register ABI to check
malformed/truncated decoder control input, input address-wrap refusal,
encoder short-capacity literal/zero-run errors, and capacity exhaustion
after a repeated-word encoder match-search. Every call returns -1 as
expected. The measured canary depth is 016 octal (14 decimal) stack
words, including the CRT main call. This matches the full-DXR v1356
measurement but is not an absolute upper bound for linked, nested
VFS/EXEC/TSFS call chains. No production KCORE or stored image changed.

## v1358: independent host-reference golden stream comparisons

The native `d6codec-parity-v1358` test uses three original v045
host-generated 128-word real-DXR compressed golden fixtures:
configctl, darc, and d6lz. Their source and expected stream data are
unchanged; only the staged v053/v055 assembler entry names differ.
All three PDP-6 tests PASS byte-for-byte equivalent 36-bit encoded
words and exact native decoded roundtrips. This verifies those
specific independent host reference samples, not full producer
migration or a recreated 81-DXR corpus: the older manifest refers to
unavailable absolute build paths. No production format was changed.
