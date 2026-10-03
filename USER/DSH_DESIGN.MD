# DAIMOS DSH AND COMMAND-INTERFACE DESIGN

Status: pre-V1 implementation design and specification\
Date: 2026-09-19\
Scope: DSH, command utilities, SIXMD command manuals, getopt/help
support, completion interfaces, and userland overlay planning.

## 1. Purpose

This document is the implementation specification for the first substantial
DAIMOS userland port: DSH.

DSH is a native DAIMOS shell inspired primarily by ksh. It is not a
POSIX `sh` port and POSIX source compatibility is not a design goal. The
existing DAIMOSV1 DSH implementation is the starting point and source of
proven mechanisms, but it must be adapted to the current DAIMOS kernel
ABI and to the current PDP-10 documentation rather than copied
unchanged.

The design has four related parts that must be developed together:

1.  DSH itself: parser, evaluator, variables, functions, pipelines,
    redirection, jobs, interactive editing, completion, and eventual
    overlays.
2.  Core external utilities needed to make the shell useful and to
    bootstrap later userland.
3.  A small common getopt/help library generated from command manuals.
4.  SIXMD command manuals as the single authoritative description of
    command identity, options, operands, short help, completion classes,
    and manual text.

The exact SIXMD presentation language may still be revised later. The
command-manual architecture and the requirement that
completion/help/getopt metadata be derived from manuals are already part
of the design.

## 2. Sources and authority

Implementation work should consult these sources in this order.


### 2.0 Immediate implementation prerequisites

THE GCC/LIBGCC -> KCC/DAS USERSPACE CONVERSION, RUN/EXEC STARTUP ABI, DUP2, TTY
LINE DISCIPLINE, DESCRIPTOR ISATTY, INHERITED UMASK, LOGIN INITIAL ENVIRONMENT,
RECLAIMABLE D6FS BLOCK CACHE, AND DIRECT-SLIDE MM COMPACTOR/RELOCATOR ARE
COMPLETE.

THE PRE-DSH KERNEL-STABILIZATION PASS IS COMPLETE.  KEEP THE CONSOLIDATED
MONITORFS NAMESPACE, PROCESS-ONLY /PROC ALIAS, I/O-ONLY /DEV PROJECTION, AND
THE SEEK/CHOWN/RMDIR/MTIME/STAT/DATE/TOUCH ABI PASSING THEIR TARGET-SIDE
REGRESSIONS.  SUBSTANTIAL DSH MAY NOW BEGIN WITHOUT AN OPEN FILESYSTEM/SYSCALL
GATE.

DO NOT ADD DAIMOSV1 SYSCTL/DEVIO COMPATIBILITY INTERFACES.  INTROSPECTION AND
ADMINISTRATION TOOLS MUST TARGET MONITORFS.  COMPILER OR ASSEMBLER DEFECTS FOUND
DURING THIS WORK STILL TAKE PRIORITY AND MUST BE FIXED AT THEIR SOURCE.

### 2.1 Current DAIMOS source and ABI

The current DAIMOS kernel and libc are authoritative for available
system calls and process semantics. The old `USER/DSH.MD` contains
obsolete statements saying that pipes, `dup`, `chdir`, descriptor maps,
process groups, and job-control terminal handling are deferred. Those
statements must not guide the new implementation.

The current kernel already exposes, among other interfaces:

-   `SYS_RUN` / `dsys_run`
-   `SYS_WAIT` / `dsys_wait`
-   `SYS_DUP` / `dsys_dup`
-   pipe creation through `dsys_pipe`
-   `SYS_CHDIR` / `dsys_chdir`
-   `SYS_GETCWD` / `dsys_getcwd`
-   process-control operations through `dsys_procctl`
-   process groups
-   sessions
-   domains
-   process events
-   controlling-terminal attachment and foreground-process-group control
-   stopped and continued wait states
-   descriptor mapping in the current RUN structure

These facilities are baseline inputs to DSH, not speculative future
features.

### 2.2 `pdp10-doc/USER/SIXMD.MD`

This is authoritative for the current
command-manual/help/getopt/completion architecture.

Important rules already defined there include:

-   one manual is the source of truth;
-   `%SIXMD 1` identifies SIXMD source;
-   semantic headings use `@?`;
-   command identity comes from one anchored level-one heading;
-   `OPTIONS`, `OPERANDS`, `ENVIRONMENT`, `FILES`, `EXIT STATUS`, and
    `SEE ALSO` are semantic sections;
-   option declarations generate option-parser descriptions, synopsis
    text, HELP output, and DSH option completion;
-   operand metavariables define completion classes;
-   malformed manuals fail validation at build/install time;
-   malformed or missing installed metadata must never crash DSH or a
    command;
-   runtime consumers should use compact generated metadata instead of
    parsing complete manuals.

### 2.3 Existing `pdp10-doc/USER/DSH.MD`

This remains useful for the native SIXBIT language, namespace, parser,
fixed-memory, function, and command-policy concepts. It must be
corrected where it describes the older kernel ABI.

### 2.4 DAIMOSV1 DSH

DAIMOSV1 contains two useful implementation lines.

The modular implementation provides reusable architecture:

-   `dsh_s6.[ch]`
-   `dsh_lex.[ch]`
-   `dsh_parse.[ch]`
-   `dsh_env.[ch]`
-   `dsh_exec.[ch]`
-   `dsh_sys.[ch]`
-   `dsh_host_text.[ch]`

It already contains fixed arenas, AST parsing, variables, functions,
control structures, command lookup, source callbacks, and host-test
boundaries.

`dsh_native.c` contains later practical native mechanisms, including
PATH lookup, aliases, globbing, redirection, pipe handling, environment
handling, interactive history/completion scaffolding, and native input
handling.

The new DSH should merge the useful concepts from both rather than
choosing one wholesale.  In particular, reuse the parser/SIXBIT/environment/
PATH/glob/redirection/alias/test architecture, but replace the old executor and
pipeline/job-control path.  DAIMOSV1 `dsh_native.c` can execute pipeline stages
sequentially through temporary shell-descriptor redirection; the current DAIMOS
requires concurrent pipelines grouped into one process group with foreground
TTY ownership transferred to the job.

## 3. Fundamental DSH rules

The following are frozen unless measurements or a demonstrated kernel
limitation require a change.

-   DSH is native DAIMOS software.
-   DSH is ksh-inspired, not POSIX-compatible.
-   Native shell-language values are SIXBIT.
-   Persistent shell strings use bounded/countable SIXBIT
    representations.
-   The resident shell must not depend on NUL-terminated byte strings as
    its native language representation.
-   DSH uses RUN/WAIT, not fork/exec.
-   Memory use is fixed and bounded.
-   No heap growth is required for the shell core.
-   Ordinary utilities are external programs.
-   Builtins exist primarily when the operation must alter the current
    shell state.
-   Pipelines and job control are baseline features because the current
    kernel supports them.
-   Completion is advisory and may never prevent command execution.
-   Manual/help/completion metadata must never be trusted without bounds
    validation.
-   Feature creep is avoided; ksh is a behavioral source, not a
    checklist that must be cloned.

## 4. Native namespace and command lookup

Initial native command trees remain:

    /SYSTEM/EXEC
    /OPTION/BASE/EXEC
    /OPTION/<NAME>/EXEC

Manual trees remain associated with the system/package trees, for
example:

    /SYSTEM/MANUAL
    /OPTION/BASE/MANUAL
    /OPTION/<NAME>/MANUAL

The initial PATH should remain:

    /SYSTEM/EXEC:/OPTION/BASE/EXEC

Command lookup order is:

1.  builtin
2.  shell function
3.  explicit path when the command contains `/`
4.  PATH search

Aliases are an interactive/source-language preprocessing facility and
therefore occur before ordinary command resolution.

Every installed normal command should have a matching validated command
manual once installation tooling enforces the policy.

## 5. Shell language

### 5.1 Operators and keywords

DSH deliberately uses SIXBIT-friendly word operators.

Logical operators:

    AND
    OR
    NOT

Pipeline operator:

    !

Grouping:

    BEGIN ... END

Functions:

    DEF NAME DO
        LIST
    DONE

Control structures:

    IF ... THEN ... [ ELSE ... ] FI
    FOR ... IN ... DO ... DONE
    WHILE ... DO ... DONE
    UNTIL ... DO ... DONE
    CASE ... IN ... ESAC

Background execution:

    COMMAND &
    A ! B &

`&` applies to the completed pipeline/job and terminates that asynchronous job.
An AND/OR chain is not backgrounded implicitly; use `BEGIN ... END &` to run
the grouped compound command as one asynchronous job.

Command substitution is reserved as:

    $( LIST )

Variable disambiguation uses:

    $[NAME]

The shell must not attempt to force unavailable Unix punctuation into
the language merely to resemble Bourne shell syntax.

### 5.2 Grammar target

The parser should support a grammar equivalent to:

    SCRIPT   := LIST
    LIST     := ANDOR ( SEP ANDOR )* [ SEP ]
    SEP      := ; | RECORD-END
    ANDOR    := NOTCMD ( AND NOTCMD | OR NOTCMD )*
    NOTCMD   := [ NOT ] PIPELINE
    PIPELINE := COMMAND ( ! COMMAND )* [ & ]
    COMMAND  := SIMPLE | IF | FOR | WHILE | UNTIL | GROUP | DEF | CASE
    SIMPLE   := WORD ( WORD | REDIR )*
    REDIR    := < WORD | > WORD | >> WORD
    IF       := IF LIST THEN LIST [ ELSE LIST ] FI
    FOR      := FOR NAME IN WORDS DO LIST DONE
    WHILE    := WHILE LIST DO LIST DONE
    UNTIL    := UNTIL LIST DO LIST DONE
    GROUP    := BEGIN LIST END
    DEF      := DEF NAME DO LIST DONE

CASE is frozen as:

    CASE WORD IN
        WHEN PATTERN [ PATTERN ... ] DO
            LIST
        WHEN PATTERN DO
            LIST
    ESAC

The first matching arm wins and there is no fall-through.  `WHEN * DO` is the
normal default arm.  `*` and `?` use the same bounded matcher as pathname
globbing.  Quoting makes wildcard characters literal.

### 5.3 Quoting and escaping

The first working DSH uses these fixed SIXBIT rules:

-   an unquoted physical record end is a list separator equivalent to `;`;
-   unquoted `#` begins a comment through the physical record end only when it
    appears where a new token could begin; inside a word it is an ordinary
    character unless quoted/escaped for some other reason;
-   unquoted whitespace separates words; keywords/operators are recognized only
    when the complete token is unquoted;
-   single quotes `'...'` are fully literal; no variable expansion, command
    substitution, wildcard interpretation, or backslash processing occurs;
-   double quotes `"..."` preserve whitespace and make wildcard characters
    literal while allowing variable expansion and `$(...)`;
-   inside double quotes, backslash only quotes `"`, `\`, `$`, or a physical
    record newline;
-   outside quotes, backslash quotes the next SIXBIT character; backslash before
    a physical record newline removes that record boundary and continues the
    logical command;
-   `''` and `""` create a real empty argument;
-   adjacent quoted and unquoted fragments concatenate into one argument;
-   quoted reserved words and operator characters are ordinary data.

Variable expansion never performs POSIX/IFS-style implicit word splitting.  A
variable containing spaces remains one shell argument unless an explicit DSH
construct says otherwise.

### 5.4 Invocation and script modes

The first DSH command interface is frozen as:

    DSH
    DSH -L
    DSH -C COMMAND [ ARG ... ]
    DSH SCRIPT [ ARG ... ]

With no script or `-C`, DSH is interactive only when a kernel/libc TTY query confirms that standard input is its controlling terminal.  `-L` selects login-shell startup policy.  `-C` executes
one command-list string noninteractively and is required by later MAKE use.
`DSH SCRIPT` reads the native SIXBIT script file noninteractively.

For a script, `$0` is the script path and `$1`... are its arguments.  For
`-C`, `$0` remains DSH and following arguments become `$1`....

    SOURCE FILE [ ARG ... ]

executes FILE in the current shell and temporarily replaces the positional
parameter frame for the duration of the sourced file, then restores it.
Noninteractive shells do not implicitly source profiles.

## 6. Variables and environment

The implementation should distinguish at least:

-   ordinary shell variables;
-   exported environment variables;
-   positional parameters;
-   special shell parameters.

Required special parameters include:

    $?
    $$
    $0
    $#
    $1 ... $9
    $NAME
    $[NAME]

Current DAIMOS implements the bounded startup contract as RUN V2 and EXEC V1.
Both caller-side launch descriptions are self-contained and bounded, use
counted SIXBIT argument/environment records, and contain no graph of raw user
pointers.  RUN retains its process-group and descriptor-map fields.  RUN and
EXEC share the PDP-6 child-image vector/record builder.  The child/replacement
image receives:

    AC1 = argc
    AC2 = argv vector
    AC3 = envp vector or zero

Each argv/environment string is a counted SIXBIT record in child memory.
Environment records use `NAME=VALUE`.  RUN V2 places the counted executable path first, followed by argv records,
environment records, and descriptor mappings.  EXEC V1 places the counted path
first, followed by argv and environment records.  INIT passes the configured
TTY id to LOGIN as argv[1]; the old private AC1=TTY convention is gone.

Assignment syntax is:

    NAME=VALUE

Assignment creates/changes a shell variable but does not export it by default.
Environment entries inherited at startup begin marked exported.

Shell-state commands include:

    SET
    UNSET NAME
    EXPORT NAME
    EXPORT NAME=VALUE

`EXPORT NAME` marks the existing value exported, creating an empty value if the
name did not exist.  `EXPORT NAME=VALUE` sets and exports it.  `UNSET` removes
the variable.  Expanding an undefined variable contributes an empty fragment.
There is no IFS-style implicit field splitting.

The first DSH has no `LOCAL` or `TYPESET`.  Function calls save/replace/restore
the positional-parameter frame, but ordinary variable assignments inside a
function affect the shell's global variable table.  `$0` remains the shell or
script identity; `$#` and `$1`... refer to the current function/source/script
positional frame.

## 7. Functions

Functions are a required DSH language feature.

    DEF NAME DO
        LIST
    DONE

Rules:

-   fixed-size function table;
-   fixed call-depth limit;
-   functions use normal command invocation syntax;
-   functions shadow external commands;
-   `RETURN [STATUS]` exits the current function;
-   `EXIT` exits the shell;
-   no closures;
-   no exported functions initially;
-   recursion is allowed only up to the fixed call-depth limit.

Function bodies should be stored as compact persistent parser/evaluator
structures. They must not contain pointers into an unloadable parser
overlay.

## 8. Pipelines and redirection

Pipelines are baseline functionality:

    A ! B
    A ! B ! C

The implementation must:

1.  allocate all required pipes;
2.  launch all stages before waiting;
3.  create one process group for the pipeline;
4.  join subsequent stages to that group;
5.  map the appropriate pipe ends onto child descriptors;
6.  close unused descriptors in the parent immediately;
7.  for foreground jobs, assign the pipeline process group to the
    controlling terminal;
8.  wait for the job/process group;
9.  restore shell foreground ownership after completion or stop.

Pipeline status is the status of the last stage unless a later explicit
feature changes that policy.

Pipeline transport is the native 36-bit word FIFO ABI.  Normal text pipeline
stages exchange S6REC/SIXBIT words, the same representation used by native
text files.  A stage which requires complete output must tolerate short
WRITE_WORDS results and continue until its record/buffer has been transferred;
IPC itself does not decode text or make multiword records atomic.

The implementation may use a fixed maximum number of pipeline stages,
but this is an implementation resource limit rather than a language
semantic. Start with a small measured limit such as 8 and adjust from
memory measurements.

Required initial redirections:

    < FILE
    > FILE
    >> FILE

Full Bourne descriptor-redirection syntax is deliberately out of scope
until a real use requires it.

### 8.1 Builtins/functions in pipelines

This requires an explicit implementation rule because DSH has no fork.

Initial rule:

-   shell-state-changing builtins execute only in the parent when used
    as ordinary commands;
-   pipeline stages should initially be external commands;
-   a later DSH worker image may permit functions or selected builtins
    in pipelines without cloning the interactive shell.

Do not implement a hidden workaround that gives misleading parent-shell
semantics.

## 9. Job control

Job control is a required DSH capability because the current kernel
already provides the necessary process-group, session, terminal, event,
stop/continue, and group-wait facilities.

Required user-visible concepts:

    COMMAND &
    JOBS
    FG
    BG
    WAIT

Ctrl-C should affect the foreground job. Ctrl-Z should stop the
foreground job. `FG` and `BG` should continue stopped jobs in the
requested mode.

A pipeline is one shell job and one process group.

Job references use `%N`.  `FG` or `BG` without an argument selects the most
recent eligible job.  The first DSH does not implement `%+`/`%-` aliases.

`JOBS` uses a compact stable form equivalent to:

    %N STATE [STATUS] COMMAND

where STATE is RUNNING, STOPPED, or DONE and STATUS is shown for DONE jobs.
A pipeline is one job and one process group.

Background completion/stop notification occurs only at safe shell boundaries,
never in the middle of interactive line editing.  A completed job record is
reclaimable after its completion has been reported once either by such a
notification or by `JOBS`.

Background standard input remains inherited.  If a background process attempts
to read its controlling terminal, the existing kernel foreground-TTY rule stops
the process group; DSH does not silently redirect stdin to a null device.

The job table must be fixed-size. Exhaustion must produce a clear shell error
rather than corrupting state.

DSH should use kernel sessions and domains as defined by DAIMOS. It must
not create a second shell-specific session/domain abstraction.
No SessionFS is required: sessions and process groups are native kernel state,
and DSH consumes the normal kernel interfaces directly.

## 10. Command substitution

Syntax is reserved:

    $( LIST )

The implementation should eventually execute a complete DSH command list
and capture its standard output through a pipe.

The first implementation is bounded to `DSH_WORD_MAX` captured SIXBIT
characters; start with 255 unless measurement requires a smaller bound.  Record
separators in captured text are normalized to single spaces and trailing record
separators are removed.  The result is one word fragment and is never subjected
to implicit field splitting.

Command substitution is allowed in ordinary unquoted and double-quoted word
fragments, not inside single quotes.  Nesting is allowed only up to the fixed
parser/evaluator depth.  It eventually runs a DSH worker/external DSH execution
context and captures stdout through a pipe; it must not recursively clone the
interactive shell state.

Capture overflow, malformed nested input, or read/launch failure aborts the
enclosing simple command with status 2 and a diagnostic.  The child status is
not assigned to `$?` in the middle of expansion; `$?` changes when the enclosing
command completes.

## 11. Globbing

DAIMOSV1 already provides useful precedent for `*` and `?`.

Initial DSH should support:

    *
    ?

Bracket expressions and more elaborate ksh pattern operators are
deferred.

Globbing is performed after variable expansion and after quote processing has
identified literal wildcard characters.  If a pattern has no matches, the
original pattern is left unchanged as one literal argument.

The same bounded wildcard matcher should be reusable for CASE patterns.

## 12. Interactive input, history, and editing

Login startup policy is fixed before interactive editing details:

-   LOGIN starts DSH with `-L`;
-   a login DSH reads `/CONFIG/DSH.PROFILE` and then `$HOME/.DSH.PROFILE` if
    present;
-   nested interactive shells and noninteractive shells source no profile
    implicitly;
-   a missing profile is ignored; a profile read/syntax error is diagnosed but
    does not terminate an interactive login shell, and leaves a nonzero `$?`;
-   profile files use native SIXBIT text records.

The CTY baseline is transcript-oriented. DSH must not require cursor
addressing, curses, ANSI escape sequences, or a terminal database.

DAIMOSV1 provides precedent for:

-   line editing;
-   small history;
-   TAB completion;
-   history recall;
-   Ctrl-D handling.

The first implementation should keep editing deliberately small and
bounded.  The accepted kernel TTY design also provides a minimal cooked mode
for ordinary programs, but interactive DSH deliberately selects raw/no-echo
input while its own editor is active because history and TAB completion remain
shell policy.

TTY mode is terminal state and must participate in job-control handoff.  DSH
keeps its own raw/no-echo mode, gives a newly started ordinary foreground job
canonical+echo mode, and restores the shell mode after the job exits or stops.
If a foreground job changes its mode and then stops, DSH records that compact
mode in the job entry and restores it before `FG` continues the job.  This is
required for full-screen programs such as Elvis; merely restoring the
foreground process group is insufficient.

Richer serial-terminal editing can be added later without changing the
shell grammar.

## 13. Completion

Completion is required, but the exact future completion specification is
secondary to getting DSH running.

The architecture is already fixed:

-   command manuals are the source of command option/operand metadata;
-   DSH does not maintain hand-written per-command completion tables;
-   DSH does not parse complete SIXMD manuals for every TAB;
-   build/install tools generate a compact validated completion index;
-   DSH reads only bounded generated metadata;
-   invalid, missing, stale, or unsupported metadata causes safe
    fallback.

Generic completion works without command metadata:

-   command position: builtins, functions, aliases, PATH commands;
-   path position: filesystem names;
-   variable position: shell variables;
-   job position: current jobs.

Semantic completion uses standard manual metavariable classes currently
including:

    FILE
    DIR
    PATH
    COMMAND
    USER
    DEVICE
    PROCESS
    NUMBER
    STRING

The exact compiled completion-index format may be finalized later with
SIXMD. Its parser must be bounded and defensive.

## 14. Overlay architecture

DSH is a strong candidate for userspace overlays because it is
long-lived but has mutually exclusive phases.

Provisional decomposition:

    DSHCORE
        permanent shell state, command execution, variables,
        RUN/WAIT, descriptor handling, job table, overlay dispatcher

    DSHPARSE
        lexer/parser, compound grammar, function definition,
        CASE/IF/FOR/WHILE/UNTIL parsing

    DSHEDIT
        interactive editor and history

    DSHCOMP
        semantic completion and command-description lookup

This split is not yet frozen. Actual boundaries must be chosen from
linker-map and call-frequency measurements.

Rules for overlay-safe design:

-   persistent shell structures contain indices/offsets, not pointers
    into overlays;
-   parser output must survive parser-overlay removal;
-   scripts should not require interactive overlays;
-   simple interactive commands should avoid loading the heavy compound
    parser when practical;
-   completion should not remain resident merely because the shell is
    interactive;
-   do not create many tiny overlays without measured memory benefit.

A possible later optimization is to parse functions/compound commands
into a compact evaluator representation, unload the parser, and execute
from the compact representation.

If the current userland lacks the required overlay loader, keep the
source modular and overlay-safe first; do not create a private
workaround in DSH.

## 15. Builtin policy

A command should be builtin when it must modify current shell state or
is tightly coupled to shell execution.  `.` is an exact synonym for `SOURCE`
with the same positional-argument save/restore semantics.

Expected builtins:

    EXIT
    CD
    SET
    UNSET
    EXPORT
    UMASK
    READ
    SHIFT
    RETURN
    SOURCE
    .
    WAIT
    JOBS
    FG
    BG
    ALIAS
    UNALIAS

`UMASK` with no operand prints the current three-digit octal mask; `UMASK OCTAL`
sets the low nine permission-mask bits for the shell and future children.

Possible later builtins:

    LOCAL or TYPESET
    TRAP

Normal utilities should be external, including:

    ECHO
    TRUE
    FALSE
    TEST
    [
    MAN
    HELP
    PWD
    CAT
    LS
    STAT
    WC
    HEAD
    TAIL
    CMP
    STRINGS
    HEXDUMP
    SYSCTL
    DEVS
    MODS
    MOUNTS

The old DAIMOSV1 resident `ECHO`, `HELP`, and status-display
conveniences should not automatically be retained.

## 16. Status and diagnostics

Initial command status convention:

    0       success
    1       ordinary false/failure
    2       shell syntax or usage error
    126     found but cannot execute
    127     command not found

`$?` stores the shell-visible status of the most recently completed
foreground command/list according to language rules.

DSH itself should contain short diagnostics, not long help text. Long
explanations belong in manuals.

Diagnostics must distinguish at least:

-   syntax error;
-   command not found;
-   cannot execute;
-   redirection failure;
-   pipe/resource failure;
-   job-table exhaustion;
-   function/call-depth exhaustion;
-   parser/arena exhaustion;
-   substitution overflow.

## 17. Profiles and startup

The startup policy frozen in section 12 is implemented and target-validated:

-   `DSH -L` reads `/CONFIG/DSH.PROFILE` first and then
    `$HOME/.DSH.PROFILE`;
-   missing profiles are ignored;
-   a profile read/syntax error leaves nonzero `$?` without terminating the
    login shell;
-   ordinary interactive and noninteractive shells do not source either file
    implicitly;
-   profile files use native SIXBIT/S6REC text only.

Do not reintroduce a byte-text fallback merely to enable profiles.

## 18. SIXMD command-manual profile

The command manual is the authoritative declaration of command interface
metadata.

Current semantic structure:

    %SIXMD 1
    # @?COMMAND - ONE LINE SUMMARY

    ## @?OPTIONS
    ### @?-F - SHORT DESCRIPTION
    ### @?-O FILE - SHORT DESCRIPTION

    ## @?OPERANDS
    ### @?SOURCE FILE
    ### @?[DEST FILE]
    ### @?FILE...

    ## @?ENVIRONMENT
    ## @?FILES
    ## @?EXIT STATUS
    ## @?SEE ALSO

The exact SIXMD markup may be revisited later, but the semantic
principle should not change.

The build validator must reject malformed semantic metadata. Runtime
programs must safely degrade if installed generated metadata is absent
or corrupt.

## 19. Common getopt/help library

A small common getopt-like library is required for DAIMOS utilities.

### 19.1 Goals

The library should:

-   consume a compact generated per-command option descriptor;
-   recognize valid one-letter options;
-   know whether an option requires a value;
-   provide descriptors usable by the separate HELP command;
-   return option/value information to the application;
-   avoid embedding duplicate help tables in each utility;
-   use bounded storage;
-   remain small enough for ordinary PDP-10 commands.

The application remains responsible for semantic validation and
behavior.

### 19.2 Initial option model

V1 command-profile options are:

-   one dash;
-   one SIXBIT option letter;
-   optional required value metavariable.

Examples:

    -F
    -V
    -O FILE

Deferred until a real command requires them:

-   long options;
-   optional option values;
-   option aliases;
-   mutually exclusive option groups;
-   complex subcommand grammars.

### 19.3 HELP

DAIMOS DOES NOT RESERVE A PER-COMMAND `-H` HELP OPTION.  SHORT HELP IS
PROVIDED BY `HELP COMMAND` USING SIXMD OR ITS GENERATED COMPACT DESCRIPTOR.

Generated short help contains at least:

-   command name and one-line summary;
-   generated synopsis;
-   option spellings and required metavariables;
-   short option descriptions;
-   direction to `MAN` for full documentation.

Programs must not carry a second hand-maintained help text table.  Basic
command execution must not depend on help/manual metadata being available.

### 19.4 Proposed library boundary

Keep the library mechanical. A conceptual interface should provide
operations equivalent to:

-   initialize parser with argc/argv and generated descriptor;
-   fetch next option;
-   obtain option value when required;
-   identify end of options;
-   emit generated short help;
-   return the first operand index.

Do not make the library parse full SIXMD at runtime.

The exact C API should be designed when the first two or three utilities
are converted, so it reflects real call sites instead of speculation.

## 20. Manual-related utilities

### 20.1 MAN

`MAN` is the human renderer.

Responsibilities:

-   locate the requested manual;
-   parse/render SIXMD;
-   strip semantic `@?` anchors from display;
-   generate/display synopsis from semantic metadata;
-   format options and standard sections;
-   paginate appropriately for the terminal;
-   remain usable with a minimal CTY line pager.

`MAN` may parse full manuals because parsing manuals is its purpose.

### 20.2 HELP

`HELP` should be a small command-oriented entry point rather than a
second documentation database.

Required behavior:

-   `HELP COMMAND` displays the generated short command-line help: command
    summary, synopsis, options, and operands;
-   without a command, it explains how to use DSH/MAN or lists basic
    help entry points.

It should consume generated descriptors where possible.

### 20.3 APROPOS / WHATIS

`APROPOS WORD` searches the generated one-line command descriptions derived
from each SIXMD anchored level-one heading and prints matching
`COMMAND - SUMMARY` records.  It does not render the long manual or the HELP
option/operand view.

WHATIS, if retained as a separate command, uses the same one-line description
index for exact-name lookup rather than becoming another manual renderer.

These interfaces should use generated indexes rather than scanning every full
manual at runtime.

The index is derived data rebuilt from manuals.

### 20.4 Manual validator/compiler

A host/build tool, and eventually a native tool if useful, must:

-   validate SIXMD;
-   validate semantic heading placement;
-   verify command identity;
-   validate option spellings and uniqueness;
-   validate operand ordering;
-   validate known completion classes;
-   validate exit-status declarations;
-   generate getopt/help descriptors;
-   generate DSH completion indexes;
-   generate WHATIS/APROPOS indexes.

This tool belongs with userland/build tools, not inside DSH.

## 21. Core utilities related to the first DSH port

The first utility set should exercise the common interfaces without
causing feature creep.

Phase 1:

    TRUE
    FALSE
    ECHO
    TEST
    [
    CAT
    PWD
    MAN
    HELP

Phase 2:

    LS
    STAT
    WC
    HEAD
    TAIL
    CMP
    STRINGS
    HEXDUMP

System-inspection commands:

    SYSCTL
    DEVS
    MODS
    MOUNTS
    PS

Writable filesystem commands follow when needed for self-hosting:

    CP
    MV
    RM
    MKDIR
    RMDIR
    TOUCH

Stream-processing commands become especially useful once DSH pipelines
are operational:

    GREP
    TEE
    SORT
    CUT
    XARGS

Every newly adapted utility should use the common getopt/help descriptor
mechanism instead of inventing local option parsing.

## 22. Reuse plan from DAIMOSV1

Do not begin by rewriting everything.

### 22.1 Reuse/adapt first

From the modular implementation:

-   counted SIXBIT representation;
-   lexer organization;
-   fixed parser arena;
-   AST/evaluator separation;
-   fixed variable/function tables;
-   function call stack;
-   control-structure evaluator;
-   source callback concept;
-   PATH search concepts;
-   host-test conversion boundary.

From `dsh_native.c`:

-   native input handling concepts;
-   PATH lookup details;
-   aliases;
-   wildcard expansion;
-   redirection mechanics;
-   pipe mechanics;
-   environment handling;
-   small history/editor concepts;
-   completion frontend concepts.

### 22.2 Replace or substantially revise

-   obsolete RUN V1 assumptions;
-   lack of current descriptor maps;
-   single-pipe limitation;
-   four-background-child model as a semantic assumption;
-   lack of process-group job control;
-   resident ordinary utilities;
-   any byte-string native shell boundary;
-   ad-hoc help/completion tables;
-   profile disablement once a correct native record reader is
    available.

### 22.3 Current minimal DSH source

The current DAIMOS tree also contains a much smaller
`userland/dsh/dsh.c`. Treat it as evidence for the current libc/syscall
calling conventions and bootstrapping environment, not as the desired
final shell architecture. It currently has only a small parser, `CD`,
`EXIT`, output redirection, and command dispatch.

## 23. Implementation sequence

The recommended implementation order is:

### Stage 0 - establish and complete the platform prerequisites

-   unpack the newest DAIMOS, DAIMOSV1, and pdp10-doc archives;
-   record git base hashes;
-   build and run the current testkit;
-   complete the KCC-built INIT -> LOGIN -> current-DSH acceptance gate;
-   keep RUN V2 / EXEC V1 startup-ABI regression coverage current; the bounded
    counted-SIXBIT launch blocks, CRT0 AC1/AC2/AC3 convention, and INIT->LOGIN
    argv[1] TTY handoff are implemented;
-   keep regression coverage for the implemented `SYS_EXT_DUP2`/`dsys_dup2()`
    descriptor replacement operation; target probing already covers self-dup,
    invalid-source atomicity, and pipe-end replacement;
-   implement and test the minimal canonical/echo TTY ABI and job-mode handoff;
-   implement and pressure-test the reclaimable D6FS block cache;
-   inspect current RUN/WAIT/process-control structures;
-   do not rely on obsolete ABI text in old DSH documentation.

### Stage 1 - lock tests to the frozen language and invocation contract

Before importing substantial parser code, add parser/semantic tests for the
already-frozen quoting, empty-argument, CASE, variable/export, script/-C/-L,
SOURCE, glob no-match, profile, and job-reference rules.  Do not reopen these
rules merely because imported DAIMOSV1 code behaves differently.

### Stage 2 - import the DSH core architecture

-   bring over the modular SIXBIT, lexer, parser, environment, and
    evaluator structure;
-   adapt types and libc boundaries to current DAIMOS;
-   keep fixed arenas;
-   get simple external commands and shell-state builtins working.

### Stage 3 - execution and redirection

-   current RUN descriptor mapping;
-   `<`, `>`, `>>`;
-   correct close/error paths;
-   PATH lookup;
-   environment passing;
-   statuses 126/127.

### Stage 4 - pipelines

-   arbitrary bounded pipeline chain, limited by available process/file
    resources rather than by a one-pipe parser restriction;
-   process-group creation/join;
-   descriptor closure;
-   last-stage status;
-   pipeline failure cleanup.

### Stage 5 - job control

-   background jobs;
-   job table;
-   terminal foreground group;
-   Ctrl-C/Ctrl-Z behavior;
-   stopped/continued states;
-   `JOBS`, `FG`, `BG`, `WAIT`.

### Stage 6 - functions and compound language

-   IF;
-   AND/OR/NOT;
-   loops;
-   functions;
-   CASE;
-   SOURCE;
-   positional arguments.

### Stage 7 - common command interface

-   finalize the small generated descriptor;
-   implement getopt/help library;
-   implement HELP from SIXMD metadata;
-   adapt TRUE/FALSE/ECHO/TEST/CAT/PWD;
-   add descriptor and manual validation tests.

### Stage 8 - MAN/HELP/index tools

-   SIXMD renderer;
-   manual validator/compiler;
-   HELP;
-   WHATIS/APROPOS index;
-   generated synopsis.

### Stage 9 - completion

-   generic command/path/variable/job completion;
-   generated semantic command index;
-   option and operand completion;
-   safe fallback tests.

### Stage 10 - overlays

Only after the non-overlay implementation works:

-   collect linker maps and call-frequency data;
-   choose measured overlay boundaries;
-   make persistent structures overlay-safe;
-   implement the smallest useful overlay loader/interface if not
    already available;
-   measure RAM savings and execution cost.

### Stage 11 - command substitution

Implement the already-frozen bounded capture semantics after the core shell is
stable.  Profile loading belongs with login/script startup once SOURCE works.

## 24. Testing requirements

DSH requires host and target tests.

### 24.1 Lexer/parser tests

Cover:

-   every keyword/operator;
-   physical-record separators and token-boundary comments;
-   quoting/escaping and empty arguments;
-   redirections;
-   foreground and background pipelines;
-   compound commands;
-   malformed input;
-   arena exhaustion;
-   function nesting limits;
-   CASE patterns.

### 24.2 Execution tests

Cover:

-   lookup order;
-   PATH search;
-   status 126/127;
-   variable expansion;
-   environment export;
-   redirection;
-   descriptor closure;
-   pipeline EOF;
-   pipeline failure midway through launch;
-   background completion;
-   stop/continue;
-   foreground TTY restoration.

### 24.3 Manual/getopt tests

Cover:

-   valid semantic manuals;
-   malformed anchors;
-   duplicate options;
-   invalid option syntax;
-   invalid operand ordering;
-   generated synopsis;
-   HELP;
-   missing/corrupt descriptor fallback;
-   descriptor bounds;
-   completion classes.

### 24.4 Resource tests

Measure:

-   resident DSH words;
-   parser arena;
-   variable/function/job tables;
-   maximum pipeline memory;
-   completion working storage;
-   overlay sizes once introduced.

Resource-limit failures must be deterministic and recoverable.

## 25. Frozen decisions and deliberately deferred non-blockers

There are no remaining language or invocation decisions that block the first
substantial DSH implementation.  Sections 5 through 12 freeze quoting, CASE,
variables/export, invocation/script modes, profiles, jobs, command substitution,
and glob no-match behavior.

The following are deliberately deferred and must not be decided accidentally in
the first implementation:

A. Trap/event language.  Add only after basic job control and native process
   events are stable.

B. Functions or state-changing builtins inside pipelines.  The first DSH
   accepts external pipeline stages; a worker image is considered later if
   measurement and semantics justify it.

C. Final userspace overlay boundaries.  These are measurement-gated, not shell
   language decisions.

D. Exact compiled completion/getopt/index record layout and later SIXMD
   refinements.  The common semantic architecture is frozen; the compact binary
   representation may be tuned during Stage 7.

E. Advanced job aliases such as `%+` and `%-`, full termios, here-documents,
   full descriptor-redirection syntax, advanced glob operators, LOCAL/TYPESET,
   and exported functions.

## 26. Explicit non-goals for the first working DSH

Do not delay the first useful shell for:

-   POSIX shell compatibility;
-   complete ksh syntax;
-   long options;
-   arrays;
-   associative arrays;
-   arithmetic language beyond demonstrated need;
-   advanced ksh pattern operators;
-   bracket glob expressions;
-   process substitution;
-   here-documents;
-   full Bourne descriptor syntax;
-   exported functions;
-   unbounded history;
-   full-screen line editing;
-   terminal-control databases;
-   general Markdown compatibility.

## 27. Completion criteria for the first substantial DSH

DSH is ready to serve as the basis for the next userland/self-hosting
work when it can reliably:

-   execute commands by absolute path and PATH;
-   maintain shell variables and exported environment;
-   change directory;
-   redirect input/output;
-   execute multi-stage pipelines;
-   execute foreground/background jobs;
-   stop, continue, foreground, background, list, and wait for jobs;
-   run IF/loops/functions/CASE;
-   source native scripts;
-   provide bounded history/editing;
-   provide at least generic completion;
-   use common manual-derived getopt/help metadata for adapted
    utilities;
-   display command manuals through MAN;
-   survive malformed/missing completion metadata;
-   pass host and SIMH target tests without resource leaks;
-   remain within measured fixed memory limits.

At that point DSH can become the interactive environment used to
port/adapt the rest of the self-hosting chain, including MAKE, DAS, and
eventually KCC.
