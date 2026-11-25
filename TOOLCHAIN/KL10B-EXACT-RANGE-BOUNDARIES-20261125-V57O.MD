# GCC full-range exact-width pointer validation (v57o)

Testkit fed9aea; compiler remains a8b2545 (v57n). No compiler,
assembler, linker, emulator, production installation or DAIMOS change
was needed. The two new dedicated runners are outside the compiler
matrix. They validate the compiler's actual typed operations rather
than the independent guest integer-pointer model used in v56y.

## Dynamic arithmetic

`tools/test-gcc-global-exact-range-v57o.py` uses the independent
unbounded host linear-bit/slot oracle from the v56y test, adding
coordinates on both sides of address bit 29. The guest fixture
`gcc-global-exact-range-v57o.c` calls noinline typed exact16 and
exact32 addition, subtraction and comparison functions. Raw integer
pointer words are target ABI probes for a conceptual array, not
portable C pointers. No such high synthetic address is dereferenced.

Each of 4286 rows checks forward and reverse addition, subtraction
of an element count, both directions of pointer difference, and
ordering through materialized boolean expressions and branch code.
Rows cover all 36 packed positions, native slots, signed offsets,
section carries and borrows, high address bits and almost the full
30-bit coordinate span. Eight executions (O0/O1/O2/Os, SIMH/KLH10)
pass: 30002 checks per execution, 240016 in total. This is finite
boundary-focused coverage, not an exhaustive enumeration of every
30-bit coordinate or validation of all constant-folded expressions.

## Real memory accesses

`tools/test-gcc-global-exact-boundary-v57o.py` constructs 450
host-oracled signed/unsigned field cases at three actual boundaries:
5,,377777 to 5,,400000 (page), 5,,777777 to 6,,000000 (section),
and 036,,777777 to 037,,000000 (section). Coordinates are octal.
Every physical position at widths 16 and 32 and the three native
slot forms are included with both signed and unsigned operations.
The guest initializes the actual words, calls noinline functions
with volatile typed lvalues, checks the initial and post-store
values, both raw words and both neighboring guard words. The host
independently checks final physical memory as well as completion.

A further 58 fitting-field cases use the last word of section 7,
with section 010 intentionally unmapped. The guest touches only the
selected word and its preceding guard, and verifies the typed load,
store and reread. Success demonstrates no unwanted following-word
access in the tested non-crossing paths. All eight optimization and
emulator runs pass, totaling 3600 boundary rows and 464 guard rows.

The existing P10X1 fixture now accepts explicit extra physical test
words expressed as virtual coordinates outside the linked ranges
and stack. It rejects out-of-range and overlapping entries, maps
only required sections/pages to independent physical frames, and
leaves the image format and dlink unchanged. This is test-only
transport, not an implementation of cross-section object placement.
The original extended-link regression passes all ten runtime
observations and link rejection controls, including byte-identical
local DXR output. No new CPU/emulator matrix profile is introduced.

## Evidence and linked sizes

`pdp10-testkit/manifests/PDP10-GCC-GLOBAL-EXACT-RANGE-20261125-v57o.tsv`
records the 16 accepted executions, source/runner/oracle/object/image,
log and tool hashes. Use the staged v57n GCC, v56y DAS and v56m
linker. The compiler hash and installation remain unchanged.

| Fixture | O0 TEXT | O1 TEXT | O2 TEXT | Os TEXT | DATA | BSS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Arithmetic | 796 | 449 | 446 | 444 | 17145 | 5 |
| Physical boundary | 891 | 481 | 470 | 467 | 6098 | 5 |

All sizes are 36-bit words. The DATA counts are dominated by the
independent test oracle, not runtime helper or conversion tables.
They are not production RAM or speed measurements. Test inventory:
2400 unchanged matrix sources, 441 special entries, zero unowned.

## Remaining work

Full-range constant-folded arithmetic, ordinary architectural
byte/halfword/word pointer boundary tests, remaining aggregate layouts
and cast semantics are still open. Physical tests cover sections
supported by the existing KL10B fixture (0..037); the high 30-bit
coordinates are arithmetic-only probes. The tests do not establish
arbitrary object placement across sections, extended libgcc/runtime,
multiple TEXT sections, or KCC/GCC ABI compatibility. Continue with
these correctness gates before measured resident-size optimization
and final mixed-compiler acceptance. No DAIMOS changes.
