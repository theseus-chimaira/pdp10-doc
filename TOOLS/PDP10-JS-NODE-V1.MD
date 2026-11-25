# PDP-10 JavaScript KI10 emulator: native Node.js host (v1)

The pdp10-js repository provides a headless Node.js host alongside the
original browser frontend. The CPU, floating-point arithmetic, device
controllers and built-in bootstrap are shared; no npm packages are required.

## Installation and operation

On Alpine Linux edge, install or upgrade the distribution package:

    doas apk update
    doas apk add --upgrade nodejs

From the pdp10-js checkout:

    node node.js --media-dir /path/to/images

The default media directory is the current directory. The boot monitor
starts automatically. Type HELP or BOOT DPA0. Ctrl-] exits the host on an
interactive terminal; Ctrl-C is passed to the guest. The CTY is a raw
seven-bit stream, not a VT52 display renderer. Use the browser for VT52
screen applications. The optional --printer FILE appends printer output
to a file (otherwise stderr).

RP03 disk files are named dpa0.dsk through dpa7.dsk, and TM10B tape
files mta0.tap through mta2.tap. The disk format is the existing SIMH
little-endian eight-byte representation of 36-bit words. Local images
are opened read-only. Guest writes remain in the original RAM cache and
are discarded when the emulator exits. Each cache miss reads at most
one 1 MiB block from the image. A partial final block is zero padded.
The media files are not shipped with the emulator.

## Host interface and verification

The classic scripts share a single Node VM context. The Node host supplies
terminal output, panel switch callbacks, asynchronous file reads and a
printer sink. CTY input uses the same guest input register logic as the
browser and queues input until the guest accepts it. Disk I/O errors now
release the paused CPU, including memory and comparison failures.

Validated using Node.js v26.13.1 from Alpine edge: syntax checks of all
runtime scripts; local block reads, EOF, missing file, cache padding,
printer and data switches; CTY HELP; missing RP03 recovery; synthetic
RP03 bootstrap executing a guest HALT. The regression resides in
daimos-testkit tests/host/pdp10-js-node-v1.cjs (run with the pdp10-js
checkout as its argument). No complete TOPS-10 system boot or graphical
browser session was validated in this change.
