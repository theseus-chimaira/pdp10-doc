# TSFS

## 1. Scope

TSFS is the read-only native DECtape filesystem.  V1.1 remains the current
compatible on-media format.  A set may contain one through eight members,
using the complete three-bit DTC unit namespace.  Mount and boot discovery
validate set identity, generation, member descriptors, table checksums and
extent structure transiently; the resident provider keeps only compact,
validated state.

TSFS V1.2 is a locality and latency optimization for immutable media.  It does
not change filesystem semantics.  The design deliberately spends host CPU,
host RAM and a small amount of tape capacity during image construction to
minimize PDP-6/KA10 runtime RAM, kernel code and mechanical DECtape latency.

The central observation is that a multi-member DECtape set is not merely extra
capacity.  Each physical transport is an independent positioning resource.
Several tapes may move toward useful locations concurrently even though the
shared Type-136 data channel can transfer only one block at a time.

V1.2 therefore treats metadata as a set of one-block physical shards placed
across members and along each tape.  A logical metadata station may require
one or more shards, but every individual shard is exactly one DECtape block.
The number of distinct shard classes depends on the metadata size and the
number of members in the set.  Surplus members and surplus longitudinal
positions are used for staggered replicas.

## 2. Existing V1.1 representation

V1.1 uses 128-word DECtape blocks.  Each member contains redundant member
descriptors.  Canonical metadata consists of a table directory plus fixed-size
file and extent records.  Regular file data uses fixed `0400`-word restart
extents; each extent is STORED or D6LZ36 only when compression saves whole
DECtape blocks.

V1.1 metadata is intentionally direct and simple, but repeated metadata access
can force long seeks back toward the canonical tables.  This is inexpensive in
fast simulators and expensive on mechanically faithful DECtape timing.

V1.2 keeps the V1.1 canonical metadata as the authoritative recovery and
compatibility representation.  The optimized shards are additional derived
metadata and may always be regenerated from the canonical form.

## 3. Design goals

V1.2 shall satisfy the following goals:

- every physical optimized metadata shard is exactly one 128-word block;
- metadata size is structurally bounded and never depends on a probabilistic
  compressor meeting a target ratio;
- the layout adapts to sets containing one through eight members;
- independent tape motion is exploited when several members are present;
- the Type-136 data channel remains globally serialized for actual transfers;
- normal metadata access minimizes mechanical seek distance and reversals;
- common operations should need as few shard transfers as possible;
- the image builder, not the resident kernel, performs expensive optimization;
- permanent kernel RAM growth must be negligible;
- the existing mount syscall size and VFS interface should remain unchanged;
- V1.1 media must continue to mount normally;
- damaged optimized metadata must fall back to another replica or to canonical
  V1.1 metadata without modifying the read-only media.

## 4. Terminology

**Member** is one logical tape in a TSFS set.  A set contains 1..8 members.

**Shard** is one 128-word optimized metadata block.  No shard spans multiple
physical blocks.

**Shard class** is the logical role of a shard, for example A, B or C.  All
replicas of one class encode equivalent logical information.

**Metadata station** is a complete set of one replica from every shard class
required to describe the filesystem.  A station is logical, not necessarily a
contiguous physical object and not necessarily resident on one tape.

**Replica** is another physical copy of a shard class.

**Anchor** is the physical block position associated with a shard replica.
Local physical locations may be encoded as unsigned distances from an anchor.

## 5. Adaptive shard width

Let `M` be the number of members and let `W` be the number of distinct shard
classes needed by the compact metadata.  `W` must satisfy:

    1 <= W <= M <= 8

`mktsfs` constructs the compact logical metadata and chooses the smallest `W`
for which the required information fits in `W * 128` words under the V1.2
structural encoding and policy constraints.

Examples:

    one-member set, metadata <= 128 words:      W=1
    two-member set, metadata <= 128 words:      W=1, second member is replica capacity
    two-member set, metadata <= 256 words:      W=2
    four-member set, metadata <= 256 words:     W=2, two members are replica capacity
    eight-member set, metadata <= 384 words:    W=3, five members are replica capacity

If the compact metadata requires more shard classes than the set has members,
the requested optimized profile cannot be represented.  `mktsfs` must either
choose a less aggressive profile explicitly permitted by the caller or fail
the build.  It must not silently create a multi-block physical shard.

This gives the format a hard physical bound: one optimized metadata read from
one member never exceeds one block.

## 6. Split by function, not by byte range

The compact metadata must not be divided mechanically as words 0..127,
128..255, and so on.  Shard classes are semantic so common operations can
avoid unnecessary transfers.

The preferred ordering is:

### 6.1 Class A: namespace and routing

Class A contains enough information to perform pathname traversal and decide
which additional shard, if any, is needed.  It should include:

- compact directory topology;
- component names in packed variable-length SIXBIT storage;
- file type and immutable mode/owner information where it fits efficiently;
- file size or compact size class information;
- home member/home region or equivalent routing information;
- direct location information for simple files when that avoids an indirect
  metadata access;
- indexes needed to locate exceptional/multi-extent metadata in later classes.

LOOKUP, READDIR, PARENT and many STAT operations should normally need only A.

### 6.2 Class B: data mapping

Class B contains normal extent/data mapping information that does not fit
profitably in A.  Common extent records use implicit information aggressively:

- logical start is implied by the extent ordinal and fixed `0400`-word restart
  size;
- the local member is implicit where possible;
- the physical location is encoded relative to a nearby shard anchor;
- STORED/D6LZ is one bit;
- block count uses only the bits required by the bounded shape;
- the existing 36-bit integrity checksum is retained unless an equivalent
  hierarchical integrity design is separately proven.

The common extent target is two words: one packed location/shape word and one
checksum word.  A one-word extent is permitted only if integrity information
has been preserved elsewhere with equivalent detection guarantees.

### 6.3 Classes C..H: overflow and uncommon structure

Additional classes exist only when metadata cannot fit in A+B.  They contain
overflow names, unusual extent shapes, cross-member exceptions, additional
integrity information, or other infrequent data.  A must indicate whether a
later class is needed so the kernel does not transfer it speculatively unless
the placement/timing model says doing so is beneficial.

`mktsfs` should strongly prefer layouts in which boot-critical and commonly
used files resolve with A alone or A+B.

## 7. Relative physical location encoding

Absolute DECtape block numbers waste bits and ignore the placement guarantee.
V1.2 should encode local data positions as:

    direction bit + unsigned distance from the shard anchor

The magnitude is unsigned.  Forward/backward direction is explicit.  This is
simple on PDP-6 and avoids signed-distance corner cases.

The required magnitude width is determined by the maximum service radius of
the placement class.  For a 578-block DECtape, evenly partitioned regions give
approximately:

    2 stations/member: maximum local radius about 145 blocks -> 8 magnitude bits
    3 stations/member: maximum local radius about  97 blocks -> 7 magnitude bits
    4 stations/member: maximum local radius about  73 blocks -> 7 magnitude bits
    5 stations/member: maximum local radius about  58 blocks -> 6 magnitude bits
    6 stations/member: maximum local radius about  48 blocks -> 6 magnitude bits
    8 stations/member: maximum local radius about  36 blocks -> 6 magnitude bits

More replicas can therefore reduce both mechanical latency and metadata bit
cost.  This positive feedback must be included in the host-side optimizer.

The common local form omits the member number entirely.  Cross-member or
out-of-radius extents use a bounded escape representation containing an
explicit member and wider/absolute position.  The optimizer assigns a strong
cost to escape records so they remain uncommon.

## 8. Placement on one member

For one shard class on one tape, identical replicas that minimize worst-case
distance follow the one-dimensional minimax placement rule: divide the usable
tape into equal service regions and center a replica in each region, adjusted
for reserved blocks and the actual one-block shard extent.

Positions should be derived from a compact placement class rather than stored
as a resident list.  The descriptor/handoff needs only enough information to
reconstruct the chosen geometry, such as replica count and placement class.

## 9. Placement across several members

When `M > W`, surplus member capacity is used for replicas.  Replicas of the
same shard class must be staggered longitudinally rather than aligned at the
same block numbers.  Staggering produces a denser virtual set of service
points when the runtime may choose among transports.

Example for `M=4`, `W=2`:

    member 0: A replicas at phase 0
    member 1: B replicas at phase 0
    member 2: A replicas at phase 1/2
    member 3: B replicas at phase 1/2

A metadata operation may choose the nearest A from member 0 or 2 and the
nearest B from member 1 or 3 independently.  It is not bound to a fixed A/B
pair.

For `M=8`, `W=2`, an image may use four effective A carriers and four effective
B carriers, or a different weighted split when A is accessed substantially
more often than B.  Replica counts are therefore an optimization result, not a
fixed `M/W` rule.

## 10. Parallel seek model

DECtape transports can be useful in parallel even though the shared Type-136
data path cannot transfer from two units simultaneously.

For metadata requiring A+B, the desired operation is:

    choose nearest A replica
    choose nearest B replica
    start both transports positioning concurrently
    wait for both positions
    transfer A through Type-136
    transfer B through Type-136 only if A says B is required

Mechanical latency is therefore approximated by:

    max(seek_A, seek_B, ...)
      + transfer_A
      + transfer_B_if_needed
      + controller/switch overhead

not by the sum of all seek distances.

This is the principal reason the optimal layout depends on member count.

## 11. Current DTC limitation and required driver extension

The current DAIMOS DTC path serializes search and transfer with the global
`storage_state`.  `dtc_read_block()` refuses a new operation while another is
active, so V1.2 cannot yet overlap positioning requests even though per-unit
`dtc_motion[8]` state already exists.

V1.2 should not replace the Type-136 transfer serialization.  Instead the DTC
driver gains a small positioning interface separating mechanical motion from
data-channel ownership.  Conceptually:

    dtc_position(unit, target_block)
        start/continue search or prepositioning for one transport
        do not claim Type-136 for a block transfer

    dtc_position_status(unit)
        report reached/in-progress/error and update dtc_motion[unit]

    dtc_read_positioned(unit, block, dst)
        acquire the shared Type-136 transfer path
        transfer one block once the unit is positioned

The exact assembly ABI may differ, but the architecture must preserve:

- one small per-unit motion/position state;
- one global Type-136 transfer owner;
- ability for several Type-551 transports to be moving toward targets;
- no large per-request buffers;
- correct timeout/error accounting per unit;
- no global stop that unnecessarily destroys useful motion state on unrelated
  units except where the real controller semantics require it.

This DTC work is useful beyond TSFS because it removes the current assumption
that one tape's mechanical search makes every DECtape transport unavailable.

## 12. Speculative positioning

Class A is intentionally authoritative for deciding which later classes are
needed.  Nevertheless the runtime may preposition likely B/C replicas while A
is being sought or transferred.

For a common A+B operation:

1. choose A and probable B using current `dtc_motion` estimates;
2. start both seeks;
3. read A;
4. if A requires B, read the already positioned B;
5. otherwise leave B positioned for possible future use and avoid its block
   transfer.

Rare class C requests should not normally incur transfer cost, but the host
profile may justify speculative positioning when a known boot transition makes
C highly probable.

## 13. Host-side optimization problem

`mktsfs` owns all expensive layout choices.  The optimizer considers at least:

- member count `M`;
- candidate shard width `W`;
- compact metadata word counts per semantic class;
- replica count per class;
- longitudinal replica positions and phase offsets;
- file/data placement around metadata anchors;
- relative-distance bit width and escape-record count;
- tape-distance cost;
- reversal/turnaround cost;
- member-switch/controller cost;
- parallel seek cost using the maximum seek among concurrently positioned
  shards rather than the sum;
- expected transfer count per operation;
- boot/runtime transition weights from an external profile;
- capacity consumed by metadata replicas.

The optimizer should minimize a weighted objective combining runtime latency
and metadata size while obeying hard correctness and capacity constraints.
Because the filesystem is immutable, expensive dynamic programming,
branch-and-bound or other host-only search methods are acceptable.  Runtime
must not reproduce the optimizer.

For small images the builder should seek an exact optimum or prove a tight
bound where practical.  For larger images a deterministic bounded heuristic
may be used, but `mktsfs` must report the resulting measurable properties:

- chosen `W`;
- words used in every shard class;
- replicas per class;
- worst-case nearest-replica distance;
- expected weighted seek cost under the supplied profile;
- escape-record count;
- metadata capacity overhead per member.

## 14. Transition profiles

Workload knowledge must not be hard-coded into `mktsfs`.  The image build may
provide a separate transition/profile input.  A DAIMOS root profile can weight
the normal sequence, for example:

    INIT -> MOUNTALL -> FSTAB -> INITTAB -> LOGIN -> PASSWD -> DSH

The profile may also express access frequencies or probabilities for pathname
components and executable/data transitions.  `mktsfs` uses the profile to
choose file regions, shard replica weights and speculative positioning value.

Without a profile, `mktsfs` uses a deterministic generic objective emphasizing
worst-case metadata seek distance, low escape count and compact placement.

## 15. Integrity and recovery

Every optimized shard carries enough identity to reject a stale or unrelated
replica and has its own checksum.  Set ID, generation, class and replica/layout
identity must be validated before use.

On failure, the runtime tries another replica of the same required class,
preferably the next-lowest predicted latency.  If the optimized representation
cannot provide a complete valid metadata answer, the implementation falls back
to the canonical V1.1 metadata.

TSFS is read-only.  No in-place repair, replica synchronization or mutable
nearest-copy state is required.

## 16. Mount and boot handoff

The current mount ABI already reserves four TSFS-specific words in the 20-word
handoff, including one currently reserved word.  V1.2 should preserve the
syscall size and repack these words to describe:

- the logical-member to physical-DTC-unit map;
- V1.1/V1.2 runtime format selector;
- shard width `W`;
- compact placement/replica class parameters;
- canonical recovery locator as needed.

Exact bit allocation should be chosen only after the compact V1.2 encoder is
prototyped.  The design must prefer derived placement over resident location
arrays.

KINIT performs equivalent transient validation for a TSFS boot root and emits
the same compact runtime description.  KINIT code may grow modestly because it
is transient; permanent KCORE/MRES growth remains the stricter constraint.

## 17. Resident TSFS changes

The resident provider should remain mechanically simple.  Its new work is:

- decode the compact handoff;
- determine which shard class a metadata operation needs;
- choose the lowest-cost replica using `dtc_motion` and the derived placement;
- request prepositioning of multiple units when useful;
- transfer the required one-block shards through the shared Type-136 path;
- decode compact namespace/extent records into existing scratch workspace;
- fall back to another replica or canonical metadata on error.

LOOKUP, READDIR, STAT, PARENT, file reads, VFS semantics and D6LZ payload
decoding should otherwise remain structurally unchanged.

No permanent 256/384-word metadata buffer should be introduced.  Decoding must
operate from the existing block workspace or similarly bounded transient
storage.

## 18. Tool and component responsibilities

### `DAIMOS/tools/host/mktsfs`

Major changes:

- V1.2 compact semantic encoder;
- adaptive shard-width selection;
- relative-location and escape encoding;
- replica-count optimizer;
- staggered one-dimensional placement;
- multi-member parallel-seek cost model;
- data-region layout optimizer;
- optional external transition profile;
- V1.1 canonical metadata generation retained.

### `DAIMOS/tools/host/tsfscheck`

Major changes:

- validate every shard identity/checksum;
- decode V1.2 and compare it logically with canonical V1.1 metadata;
- verify derived placement and non-overlap;
- verify all local relative distances are representable;
- verify escape records and cross-member mappings;
- verify every required shard class has sufficient valid replica coverage;
- report chosen layout/cost properties.

### Userspace `MOUNT.TSFS` / scanner

Moderate changes:

- recognize V1.2 descriptors;
- validate the optimized metadata structure transiently;
- discover all eight possible members;
- construct the compact runtime handoff without retaining metadata tables.

### KINIT TSFS boot scanner

Moderate transient changes equivalent to the userspace scanner.

### DTC driver

Small but important architectural change: separate concurrent mechanical
prepositioning from serialized Type-136 block transfer.

### Resident TSFS provider

Small change relative to host tooling: shard selection, bounded decoding and
fallback.

### VFS and syscall surface

No semantic change and no mount-syscall size increase expected.

## 19. Recommended implementation order

Implementation should proceed in this order so the on-media format is derived
from measured packing rather than guesswork:

1. Add a host-only prototype compact encoder for the current root metadata.
2. Measure exact class A/B/C word counts and prove the minimum feasible `W`
   for 1..8 members.
3. Implement the one-block shard format and decoder in DAIMOS `tools/host` only.
4. Extend `tsfscheck` and require logical equivalence with V1.1 metadata.
5. Implement relative-distance encoding and the escape format.
6. Implement single-member longitudinal replica placement and measure seek
   bounds.
7. Implement multi-member staggered replica optimization using the parallel
   seek `max()` cost model.
8. Add transition-profile support and data placement optimization.
9. Extend userspace `MOUNT.TSFS` validation/handoff.
10. Extend KINIT boot-root validation/handoff.
11. Add DTC preposition/search-only support with per-unit state while retaining
    one global Type-136 transfer owner.
12. Add resident nearest-shard selection and compact decoding.
13. Test 1-, 2-, 3-, 4- and 8-member sets under SIMH and AAP, including damaged
    replicas, reversals, simultaneous positioning and canonical fallback.
14. Measure AAP boot/login latency against V1.1 before making V1.2 the default
    root-image format.

## 20. Acceptance criteria

V1.2 is complete when all of the following hold:

- one through eight members are supported;
- every optimized metadata shard is exactly one physical DECtape block;
- the builder selects `W` from actual packed metadata and available members;
- surplus members are used as staggered replica capacity where beneficial;
- multi-shard positioning can proceed concurrently on independent transports;
- actual Type-136 transfers remain correctly serialized;
- common pathname operations avoid unnecessary B/C shard transfers;
- current-root optimized metadata fits the measured shard plan produced by the
  prototype, with no undocumented size assumption;
- corrupted optimized shards fall back to replicas/canonical metadata;
- V1.1 media remains readable;
- no large permanent metadata buffer is added to the kernel;
- `mktsfs` and `tsfscheck` report enough layout metrics to compare optimizer
  changes objectively;
- SIMH and AAP boot-root regression tests pass;
- AAP demonstrates a material reduction in boot/login seek latency versus V1.1.

