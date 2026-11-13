# KCORE v1551: DTFS dirty-before-data raw arm service validated

Date: 2026-11-13. KCORE P0-B staged implementation, **P0-B OPEN**.

The v1550 standalone DTVOL2 record parser and opt-in host
formatter are now joined by a separate native PDP-6 raw RBN0
write-ahead service `dtfs_intent_arm_v1551.s`. It takes physical
DTC unit, both saved volume-ID halves, and an *independent*
caller-owned 128-word scratch buffer. The caller must already
hold a serialized provider lease and must have admitted an
undamaged DTVOL2/CLEAN volume.

The routine bypasses stale BCACHE for an actual physical RBN0
read, validates full DTVOL2/CLEAN and exact previous identity,
updates the two-word WRITE_INTENT state/seal, invalidates
both clean BCACHE identities, performs the raw physical RBN0
WRITE and then clears the scratch buffer before a fresh raw
readback. It returns success **only** for fully validated
DTVOL2/DIRTY of the same media. A failed or uncertain header
WRITE or readback marks the real mounted DTC unit with the
existing WRITE_UNCERTAIN bit, preserving all ownership. It
cannot silently accept a preexisting DIRTY/old DTVOL1 volume,
reset the dirty marker, or overwrite pending filesystem data.

Testkit v1551 executes genuine source-linked arm and validator
instructions under the native PDP-6 CPU with only Type-551 raw
I/O completions, BCACHE and volatile poison leaf test-controlled.
**14 native cases PASS and eight unsafe source mutations rejected**,
including ignored write status, lost cache invalidation,
readback bypass and wrong high/low identity. The pending DATA
buffer is checked unchanged in every case. No permanent
KCORE data/BSS allocation, ABI change or default media change.

Important: the service is NOT in the regular DTFS module link yet.
There is still no integrated MM scratch lease, versioned mount
gate, automatic first-write arming, final owner-safe CLEAN
seal or physical Type-551 full-guest cold-boot acceptance.
Therefore P0-B is **not complete**, regardless of native tests.

DAIMOS: `design/kcore-dtfs-intent-arm-v1551.md`.
Testkit: `tests/validation/daimos-dtfs-intent-arm-v1551.md`.
P0-A complete, P1-C open. VERSION 0.5, strict 32K/optimization HOLD.
