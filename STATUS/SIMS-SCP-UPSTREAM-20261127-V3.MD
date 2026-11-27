# SIMS shared SCP upstream integration (20261127-v3)

Original upstream: rcornwell/sims, master 18b6343.
Local baseline: e150428; upstream integration: 4b5aa92;
PCRE2 native makefile detection: cf52e6c.

The remaining upstream shared-infrastructure patch is integrated, including
SCP command/help changes, PCRE/PCRE2 EXPECT support, portable format
specifiers, Ethernet helpers, file-write buffer advancement, IPv4-mapped
IPv6 address handling, LINC display definition, III whitespace, SDL
keyboard mapping and screenshot error handling.

Integration decisions:

- Type 340 calls the existing idempotent display_init() on reset. A separate
  one-time initialization flag would prevent reopening after display_close().
  Existing SDL lifecycle and no-display build behavior is retained.
- SET -F device DISABLED checks detach status before setting DEV_DIS. It
  detaches only attached units and cancels pending events, including those
  of unattached active units. Multi-unit forced removal is not transactional.
- EXPECT preserves bounded string copies, cleans up rejected rules and
  failed allocations, checks realloc results, releases temporary regex
  match state on unsuccessful matches, and uses the actual NUL-stripped
  subject length. Screenshot failure paths free allocated resources.
- The native makefile detects PCRE2 when headers and library are present,
  retaining the PCRE1 fallback. No new DAIMOS kernel requirement is added.

Validation on Termux, Clang 21.1.8, optimized C99 with -Werror:

- PDP-6, KA10, KI10, KL10, KS10 builds and register sanity: PASS.
- Existing PDP-6 GO/PC and interactive console/breakpoint/step tests: PASS.
- Existing SDL forced-open failure and repeated window lifecycle tests: PASS
  (SDL dummy driver).
- Direct simulator smoke test: Type 340 disable/re-enable both opened a
  1024x1024 window; normal disabling refused attached DTC media; forced
  disabling detached the media and disabled the controller; re-enable
  reported all units unattached.
- PCRE2 10.49 EXPECT with real PDP-6 CTY output A1 and separate captured
  groups A and 1: PASS.
- SCP syntax-only compilation without regex support: PASS.

No new DAIMOS guest boot, hardware fault-recovery test, real graphical
renderer test, PCRE1 build or non-PDP simulator rebuild was performed for
this integration. The installed binaries under PDP10_PREFIX are unchanged.
The original upstream master was checked again and still points at 18b6343.
The local integration is intentionally not byte-identical to that patch.
