# GCC PDP-10 KS10 exact signed byte-pointer loads v27

Status: compiler correction verified on 2026-11-24.

## Problem

Optimized KS10 code could identify a stack slot or register
holding an encoded byte pointer as though it were an aligned
scalar byte. The word/shift fast path lost the PDP-10 pointer's
position and size selector. A signed 9-bit `char9` dereference
therefore failed at octal assertion `0245` in O1.

Optimized signed 18-bit loads through constant `POINT 18` could
also choose a left-half load where the pointer selected the right
half. That failed at assertion `0344` in O2 and Os.

## Backend repair

`pdp10_scalar_mem_p` refuses to treat nested `MEM(MEM(...))`
pointer dereferences as direct local scalar slots and requires
object evidence for QI/HI register-pointer fallback recognition.
The signed-byte expander now uses the word/shift shortcut only
when the *accessed byte itself* is known scalar or static aligned
storage; dynamic encoded byte pointers remain exact LDB loads.

Constant native 16-bit `POINT` dereferences use LDB. Constant
native 18-bit `POINT` dereferences decode their P/S selector and
use the matching `HLRZ`/`HRRZ` or signed `HLRE`/`HRRE` half-load.
The fix does not add runtime code or resident kernel memory.

## Verification

The original source `runtime-byte-pointer-diff.c` passes **36/36**
GCC profiles under strict policy in an independent directory.
The permanent focused v27 nine-bit and eighteen-bit regression
also passes **36/36** GCC profiles under strict policy. No tests
were skipped or altered to suppress failures.

Six adjacent regression sources, including packed-byvalue field
extraction and four earlier byte-pointer/copy-loop cases, were also
rerun: **224/224 PASS**, strict exit policy. The three independent
selections thus pass **296/296 GCC cells** without skipped old greens.

The compiler change is in `gcc-pdp10/gcc/config/pdp10/pdp10.c`
and `pdp10.md`. Testkit evidence is in
`history/PDP10-KS10-SIGNED-BYTEPTR-LOAD-20261124-v27.md`.
