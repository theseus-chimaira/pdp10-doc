# v1671: physical PDP-6 KINIT stack high-water at 48K and 64K

Date: 2026-11-18. Accepted DAIMOS baseline v1670:
`c3867558cd05d0d8be6a179529342cfcda1075f6`.

## The RAM question

The v1670 source-matched physical SIMH MM-event trace verified
that the dedicated TSFS swapless-only KINIT image reduced the
maximum observed occupied/reserved physical core by 1877 GCC
lowmem words in both 48K and 64K guests. This does not by
itself bound KINIT stack growth: the 0400 octal, 256-word
bootstrap pushdown list is a reserved range outside ordinary
MM dynamic owners. Deep KINIT nested calls could exhaust that
reserve despite a larger contiguous MM free arena.

DAIMOS already has a diagnostic stack-sentinel mechanism from
v639/v655, enabled only at build time with
`KINIT_STACK_WATERMARK=1`. It fills the stack above the live
pushdown pointer with octal `525252525252`, then scans the
highest modified index at late KINIT completion. v1671 uses
this existing code in a source-matched physical comparison; no
new KCORE production source or process metadata is needed.

## Exact physical setup and independent verification

Two independently linked GCC lowmem images: general ROOT=tape
TSFS/PURE and strict v1668 swapless-only TSFS/PURE. Both use
`KINIT_STACK_RESERVE_WORDS=0400`, bitmap mode3 OFF and an
identical filesystem/root workload. Each image runs at 48K
and 64K actual PDP-6 simulator core with copies of all DECtape
media and with all auxiliary DSK/DRM backing detached.

SIMH breakpoints are linked from the actual image symbols:
`kinit_stack_watermark_begin` must execute exactly once, and
the guest must reach `vm_enter_initial_user` before user code.
At that terminal point the testkit reads native
`kinit_stack_highwater`, all **256** physical stack words from
`__kinit_image_end`, and the patched low-resident immediate.
The independent host decoder computes the highest nonsentinel
stack index and verifies the original native stored watermark,
the complete unused high suffix and installed core bounds.
Missing, duplicated or inconsistent physical readings fail.

| Physical core | General high-water index | Strict high-water index | Intact upper reserve |
| --- | ---: | ---: | ---: |
| 48K | 129 | 129 | 126 words |
| 64K | 129 | 129 | 126 words |

In all four physical guests the highest touched stack address
is index **129 decimal** within the 256-word reserve. Thus the
exercised KINIT stack uses the first **130 words through that
index**, while the top **126 words** retain their sentinel.
The strict image does not require deeper KINIT stack usage, and
its instrumented linked KINIT still saves **1877 transient
words** relative to the instrumented general tape image.

The native diagnostic adds exactly **one** fixed KCORE word:
the original GCC lowmem fixed end increases from octal `024076`
to `024077`, and 48K/64K installed low-resident occupancy grows
from 17725/17741 to 17726/17742. These extra words are present
only in explicitly watermarked diagnostic images. Normal PURE
and KCORE production memory are unchanged by v1671.

## Unsafe variant rejection and remaining limits

The testkit runs four separate mutations against physical
48K simulator evidence and requires rejection by the identical
decoder/comparator: a forged high-water counter, one damaged
last stack sentinel, removal of the native initializer event,
and a forged summarized stack index. All four are rejected.

This validates the **tested successful TSFS/PURE boot path**
only. It cannot prove that DSK/DRM, auto-root, missing-media,
HALT/fatal diagnostics, unusual stacks or future initialization
paths are no deeper. Do **not** decrease the default 0400 stack
reserve on this evidence alone. A smaller reserve, if desirable,
requires independent worst-case error-path testing and physical
regressions before release.

Next: actual post-user 48K/64K process-memory pressure and
owner/pin accounting, bitmap-mode3 RUN/EXIT/WAIT/PI6, error
stack bounds and strict 32K physical KINIT acceptance.

DAIMOS source record: `design/kcore-kinit-stack-watermark-v1671.md`.
Executable testkit: `tests/validation/pure-kinit-stack-v1671/`.
