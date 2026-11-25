# PURE reconstruction: stamped DTFS physical media proof (v1925)

Stamped native DTFS mounts retain both original 36-bit volume-ID
halves in the existing per-mount SHAPE/EXTENT words. Previously the
physical RBN0 revalidator ran only after an observed read fault.
The shared PREFIX/FULL reconstruction bridge now invokes private
DTFS operation 030 octal via the existing serialized provider
lease before reading the executable and after payload application.
The ordinary DTFS vnode admission gate still precedes this request.

The private operation rejects WRITE_UNCERTAIN and QUIESCING, checks
both saved identity halves, sets the existing READ_SUSPECT barrier
for stamped volumes, invalidates directory identity and both clean
BCACHE tags, and uses the original physical RBN0 recovery leaf. The
original V1/V2 validators enforce identity, format, seal and valid
RO/armed state. A refusal keeps the barrier when physical verification
has been attempted, and the reconstruction transaction retains its
original RAW suffix. The v1923 complete root comparison follows the
final physical check before restoration can publish the APR.

Unstamped legacy volumes retain the approved v1572/v1575 operator-
trust contract, but cannot recover from READ_SUSPECT without saved
identity. This check is not a physical media lock or a file-content
hash. Same-ID clones, a physical exchange and reversal between checks,
and previously replaced mount metadata are not detected. Other
providers retain their current contracts. D6FS does not retain UUID
in its runtime reader. Known-unavailable backing admission before
DROP, D6FS/TSFS identity, EXEC/mutation lifetime and integrated unmount
are separate remaining K07 work.

Implementation b8361714, daimos-testkit 0fa4b8c/f502625. Native gates
pass 19 media-leaf and ten bridge cases; four executed unsafe controls
are rejected. The physical leaf uses actual formatted RBN0 records
with a controlled Type-551 raw-read boundary. The bridge uses original
PREFIX/FULL with controlled provider and MM calls. Prior native V1/V2
recovery, attachment, shared-loader and provider identity gates pass;
no full-guest physical exchange or PI6 acceptance claimed.

Folded lowmem tape KCC PURE MRES 3694 -> 3707 words (+13), DTFS MRES
2805 -> 2836 resident words (+31, BSS remains two). Fixed KCORE is
byte-identical at low end 025174 octal. No new permanent data or
per-process storage. RAW PURE and mode3 remain default OFF.
