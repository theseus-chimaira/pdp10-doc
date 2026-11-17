# P1-E v1653: 32K KINIT transient package size

Date: 2026-11-17. Original PDP-6 source-matched
32K PURE ON/OFF KINIT size and safety refusal PASS.
Neither candidate was booted: the original
host image tool rejected both before media.

The original source-linked minimal C89 PURE
user DXR has one child RUN, private writable
data check, EXIT-057, exact WAIT and parent
EXIT. It compiles, but it could not run in
the exact target 32K configuration because
the original KINIT payload is too large.

Both builds use the identical original
PDP-6 lowmem/GCC/TSFS three-member in-place
boot profile, physical core100000 octal
(32,768 decimal words), original stack
reserve0400 octal (256 decimal) and safe
image end077400 octal. Only the opt-in
PURE MRES flag differs:

| Configuration | KINIT end octal | Above physical core (words) | Above safe end (words) |
| --- | ---: | ---: | ---: |
| PURE OFF | 110052 | 4,138 | 4,394 |
| PURE ON | 113621 | 6,033 | 6,289 |

The two original KCORE maps and actual
KCORE streams are BYTE-IDENTICAL, with
fixed KCORE low end024253 octal. The
PURE ON build adds its expected removable
KINIT package and requires another
03547 octal (1,895 decimal) transient
staging words. Crucially, PURE OFF already
exceeds the total available physical
32K core by 4,138 words. This is not
an additional KCORE, assembler or MM
regression attributable only to PURE.

The original mkimage gate refuses both
source-built images correctly. The
testkit v1653 checker verifies the two
original KINIT symbols, source-matched
KCORE image identity, exact stack/core
bounds and genuine build diagnostics.
Three forged map/log/KCORE-stream
witnesses are all rejected.

DAIMOS source-backed report:
design/kcore-p1e-32k-kinit-staging-capacity-v1653.md.
Testkit: tests/diagnostics/pure-32k-process-v1653/,
commit 9b5a512.

To complete the current opt-in 32K profile,
reduce the true simultaneous transient
KINIT footprint by at least 6,289
decimal words or redesign removable
package staging while preserving the
bootstrap stack, required storage
providers and source validation. Merely
shrinking the stack or changing compressed
media size does not prove the decompressed
KINIT fits. A new actual 32K boot/RUN/MM
acceptance must follow such a change.

No production source/ABI/RAM change in v1653;
P1-E OPEN, PURE default OFF and strict
32K whole-KCORE optimization HOLD.
