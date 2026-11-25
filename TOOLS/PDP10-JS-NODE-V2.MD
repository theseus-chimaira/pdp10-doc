# Native PDP-10 JavaScript host v2

Supersedes the execution architecture and boot-validation status in
PDP10-JS-NODE-V1.MD. Node.js v26.13.1 on Alpine edge was used.

The original Node host evaluated classic scripts in a contextified VM.
Boot tracing showed a progressing BOOTS memory scan, accepted CTY input,
completed RP03 I/O and an unhalted CPU. The virtual PC was frequently
in 743173-743712 octal. Representative adaptive CPU batch sizes were
roughly 350 instructions before and 30000 after replacing contextified
global lookups with lexical bindings (about 86x, not a controlled
benchmark). Use vm.compileFunction with explicit host parameters and
diagnostic exports; no guest instruction or disk format change.

Implementation commit 7204103; existing synthetic host regression
passed. All four original RP03 images were obtained from
https://skn.noip.me/pdp10/ and checked for valid little-endian
36-bit words in eight-byte containers. Local media directory:

    $HOME/pdp10-images/tops10-603a-ki10-v1

Actual boot acceptance reached KI603 04-21-78, completed quick startup,
logged in as 1,2 and executed SYSTAT, which reported one logged-in job
and 8002P core left. Startup reached the CTY system prompt after roughly
20 seconds in the observed run. Guest disk changes remain volatile.

No DAIMOS kernel modifications. The original browser path is unchanged
by the v2 performance correction; no browser graphical test was run.
