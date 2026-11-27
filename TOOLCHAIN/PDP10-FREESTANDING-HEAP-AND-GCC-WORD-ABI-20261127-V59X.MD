# Optional freestanding arena heap and GCC word ABI repairs (v59x)

## Installed implementation

- KCC `783e1db`: builds the GCC arena allocator's identical C source as an optional `libp10heap-arena-v1.a`. The allocator retains explicit `p10heap_init_v1` initialization. No implicit arena or DAIMOS dependency is added to the ordinary C runtime. KCC also defines `__pdp10__` alongside `__PDP10__`.
- GCC `2b862c27`: fixes native word displacements in bitfield assembly output and the unsigned division-by-one boundary. The unsigned quotient for divisor one is the numerator, and the remainder is zero; the signed DIV instruction must not be used for this case. No extra registers or stack slots are needed. The general division sequence grows from 19 to 24 static instructions. Literal-one expansion is also handled directly.
- Testkit `d21efeb6`: selects the optional allocator only for unresolved heap symbols. Automatic heap startup is used only when the program does not explicitly initialize an arena. Its separate CRT reserves 4096 words of heap (16384 nine-bit bytes) and 1024 words of stack. Ordinary CRT and core runtime are unchanged. The arena allocator retains its existing block metadata, splitting and coalescing design. This is a bounded freestanding test environment, not the DAIMOS process allocator.

## Validation

The final installed binaries and heap archives pass all 75 genuine original heap-link cells: 66 `20000914-1.c`, six `920810-1.c`, three `990628-1.c`. The former GCC bitfield output erroneously divided a native displacement by four and shifted the result; removing the ambiguous output-side heuristic preserves the word address in RTL. The KCC lowercase target macro makes the original 990628 source select its native byte-size expectation.

Both independent GCC defect regressions fail at assertion 1 with the old compiler and pass 44/44 with the repaired compiler. Explicit initialization, arena exhaustion, automatic startup, calloc, realloc, coalescing, data preservation and target macro tests each pass all applicable 66 profiles with final repairs. The division guard run reports 168 runtime passes and 36 compile-only cells, with no failures. The bitfield guard run reports 88 passes and 32 pre-existing failures (array writeback and signed field operations), with no new failure relative to the original sweep. Both heap archive acceptance scripts pass all eight CPU/optimization combinations and archive extraction; KCC also verifies negative linkage without its optional archive. The selector passes 46 controls.

## Scope and outstanding issues

The 113 original heap-link cells yielded 83 raw passes after the initial repairs. Eight of these are not valid evidence: all 24 original `comp-goto-1.c` cells assume flat byte-address integers. Linked-image auditing confirms the computed page access is outside the actual allocated payload in all 24 cases. Their historical registration is unchanged; they remain unresolved/unverified rather than silently being counted as successes. The native typed-page companion passes 57/66 (KCC 22/22, GCC 35/44). Nine GCC O2 failures exhaust the execution step limit at assertion 2; assembly suggests instruction-field loads are moved before the dispatch loop. Further investigation is required. The three `960521-1.c` cells request two 32768-word arrays; their payload alone exhausts a 64K-word machine. No original source or resource requirement has been weakened. Eleven other original cells now require console output (seven printf, four vprintf).

Original cohort ledger: 259 confirmed runtime passes, 24 previously corrected registrations, 121 unresolved/unverified of 404 original link failures. No full-matrix result is claimed. Native self-host and extended addressing are not newly verified.

## Reproduction and installed artifacts

Evidence directory: `$HOME/tmp/pdp10-heap-v59x.E89mr6CG`. See `final-75`, `replay-v2`, `compgoto-range-audit-v59x.tsv`, `native-goto-v1`, `bitfield-guards-v1`, `div-guards-v2`, `new-v1`, `baseline`, and `heap-{kcc,gcc}-v3.log`. The earlier `new-v1` GCC automatic-heap failures are superseded by `div-guards-v2` after rebuilding the actual GCC allocator.

| Package (relative to evidence directory) | SHA-256 |
|---|---|
| `install-v2/local.tar.gz` | `7101270ce72575196b99f52aa8715eac8bb817a72d62fd74e611824720bc344b` |
| `gcc-v2/local.tar.gz` | `3ca065fb5889cf581785190f1b98bbfdc57fab0cf73e37ec047086a24e45dc5f` |
| `gcc-heap-v2/local.tar.gz` | `e0d5fe0e7d707e50090520925b1d3170eb9f81a0c90673a9003c7f55de79d010` |

All three packages are installed under `$PDP10_PREFIX`. The core freestanding runtime was not rebuilt for this change. DAIMOS kernel code is unaffected.
