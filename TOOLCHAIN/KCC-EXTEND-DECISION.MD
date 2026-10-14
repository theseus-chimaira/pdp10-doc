# KCC KL10 / EXTEND implementation decision

# KCC KL10 / EXTEND audit v1

This audit covers the standard host KCC build used by DAIMOS.

## Supported KL10 target

`-x=kl10` means KL10B-compatible section-0 code generation.  The existing KCC
IR already models the section-0 instructions that materially change ordinary C
lowering versus early machines: `DMOVE`/`DMOVEM`, `ADJSP`, `ADJBP`, hardware
floating conversion, and double floating operations.  The regression
the migrated `kl10-section0-codegen` matrix fixture in pdp10-testkit checks representative emission.

The historical `klx` spelling means KL10B non-zero-section / extended-address
code.  The standard build does not define `MULTI_SECTION`.  The dormant
multi-section path has been incomplete since the repository import and also
expects historical loader/runtime support such as `ckx` and `$$SECT` that is
not present in the current runtime tree.  A standard build must therefore
reject `-x=klx` rather than silently emit section-0 code under that name.

## EXTEND profitability

The KL10 and KS10 implement `EXTEND`, including string compare/move/edit and
conversion operations.  KL10B also provides XBLT through `EXTEND`.

No low-cost automatic KCC lowering is justified for the current IR:

- fixed-size aggregate and ordinary memory copies already lower to BLT or the
  packed byte-stream paths; XBLT primarily solves extended-address copies and
  is not a smaller section-0 replacement for BLT;
- CMPS/MOVS/EDIT/CVT extended operations require string/decimal semantics that
  KCC currently does not represent as dedicated IR nodes or builtins;
- recognizing arbitrary C loops as those operations would require new dataflow
  and alias analysis, increasing compiler RAM and complexity for uncertain
  gain;
- adding compiler-only pseudo-builtins solely to reach EXTEND would be feature
  creep unless a real DAIMOS workload demonstrates a measurable benefit.

DAS already accepts `EXTEND`, so handwritten assembly or explicit assembly can
use it when required.  Automatic generation should be revisited only with a
specific common-source workload showing a code-size or runtime win large enough
to justify the compiler-state cost.
