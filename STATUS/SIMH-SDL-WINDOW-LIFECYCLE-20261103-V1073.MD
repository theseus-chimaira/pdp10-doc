# SIMH SDL window lifecycle and DAIMOS SCLK boot (v1073)

Date: 2026-11-03

## Symptom and scope

An earlier PDP-6 DAIMOS software-clock (SCLK) 50-Hz fixture exited with
status 139 after SDL window-event diagnostics.  The historical log also
records Mesa EGL/ZINK warnings.  A later full graphical SCLK boot on the
current simulator did not reproduce the segmentation fault; therefore the
original intermittent event has not been conclusively traced to an exact
instruction.  SCLK is a guest clock-provider choice; it is not evidence that
the 50-Hz software clock caused the host crash.

## Confirmed memory-safety defect

In `display/sim_ws.c`, `ws_init()` called `vid_map_rgb()` twice even after
`vid_open()` had failed.  The mapping operation requires a valid SDL pixel
format and dereferences the absent format on failure.  A regression that
forces `vid_open()` to fail demonstrates two illegal mappings before the
fix, and none after it.

## Related SDL lifecycle repairs

SIMH now initializes allocated `VID_DISPLAY` fields, handles failed window
creation without calling global `SDL_Quit()` while other windows are active,
cleans up per-window graphics objects (including pixel formats), rejects
stale queued window events without caching dead window pointers, and avoids
use-after-free during multi-window shutdown.  The Type 340 frontend releases
failed-init cursors/surface; the display frontend releases allocated points.

## Validation

- Simulator PDP-6 optimized and AddressSanitizer builds: PASS, with internal
  register sanity checks.
- `daimos-testkit/tests/validation/simh-sdl-window-open-failure-v1.sh`:
  pre-fix FAIL (two unexpected color mappings); post-fix PASS.
- `daimos-testkit/tests/validation/simh-sdl-window-lifecycle-v1.sh`:
  PASS with SDL `dummy` and `offscreen` drivers, including AddressSanitizer.
- Native 50-Hz SCLK DAIMOS full graphical boot, PCLK disabled, DPY GUI and
  WCNSLS color scope enabled: exit 0, SCLK OK, DPY OK, expected HALT;
  also PASS with the patched AddressSanitizer simulator.

This establishes a correction of the reproducible SDL-open failure and the
tested video lifecycle paths, but not statistical elimination of every
intermittent host/SDL fault.  No DAIMOS kernel code changed for this fix.

SIMH source commit: `edd19b1` (v1073).
Testkit commit: `0c794cf` (v1073).
