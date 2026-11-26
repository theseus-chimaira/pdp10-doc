# Native record-array expansion for register aggregates (v59b)

GCC c0306f24; testkit d81fccd5. The November 25 full sweep
reported 33 GCC ICEs for `20000412-5.c` at O1/O2/Os. The
11 O0 profiles passed. GDB on the baseline compiler confirmed
that the two-word automatic array had DECL_RTL `(reg/v:DI 25)`.
The native record-array recognizer accepted it even though its
physical word-coordinate memory expander requires a MEM.

The shared read/write recognizer now excludes declarations whose
existing RTL is not MEM. The generic aggregate expander retains
register field extraction and insertion; the native record stride
path still handles memory objects. No forced stack allocation,
new target runtime helper or ABI change is introduced.

The installed GCC passed 132/132 strict semantic matrix cells:
44 original upstream cases, 44 existing native memory record-array
stride cases, and 44 new `gcc-native-register-record-v59b.c`
cases (live full-word and subword field reads, writes, signed
values and adjacent-field preservation). The new regression
reproduced the baseline ICE in its native PDP-6 O1 cell. No
timeouts or test expectations were changed.

The original sweep is preserved; these are targeted replay results,
not an updated full-sweep count. Evidence directory:
`$HOME/tmp/gcc-record-ice-v59b.37IcnRMR`. Installed incremental
`local.tar.gz` contains only cc1; SHA256:
`1ee8ab1b07075ed704fdaaea051f7d6b850ba33e86a548162ed32fa2a239a15b`.
No DAIMOS source changes or target size benchmark.
