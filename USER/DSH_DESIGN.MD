# DAIMOS DSH AND COMMAND-INTERFACE DESIGN

Status: pre-V1 implementation design and handoff\
Date: 2026-09-13\
Scope: DSH, command utilities, SIXMD command manuals, getopt/help
support, completion interfaces, and userland overlay planning.

## 1. Purpose

This document is the implementation handoff for the first substantial
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


### 2.0 Immediate build prerequisite

Before new DSH feature implementation, convert the existing DAIMOS userspace
(libc, INIT, LOGIN, DSH, and current commands) from GCC/libgcc to KCC and pass
the real INIT -> LOGIN -> DSH acceptance path. Compiler or assembler defects
found by that conversion take priority and must be fixed at their source.

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
    text, `-H` output, and DSH option completion;
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
choosing one wholesale.

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

Command substitution is reserved as:

    $( LIST )

Variable disambiguation uses:

    $[NAME]

The shell must not attempt to force unavailable Unix punctuation into
the language merely to resemble Bourne shell syntax.

### 5.2 Grammar target

The parser should support a grammar equivalent to:

    SCRIPT   := LIST
    LIST     := ANDOR ( ; ANDOR )*
    ANDOR    := NOTCMD ( AND NOTCMD | OR NOTCMD )*
    NOTCMD   := [ NOT ] PIPELINE
    PIPELINE := COMMAND ( ! COMMAND )*
    COMMAND  := SIMPLE | IF | FOR | WHILE | UNTIL | GROUP | DEF | CASE
    SIMPLE   := WORD ( WORD | REDIR )* [ & ]
    REDIR    := < WORD | > WORD | >> WORD
    IF       := IF LIST THEN LIST [ ELSE LIST ] FI
    FOR      := FOR NAME IN WORDS DO LIST DONE
    WHILE    := WHILE LIST DO LIST DONE
    UNTIL    := UNTIL LIST DO LIST DONE
    GROUP    := BEGIN LIST END
    DEF      := DEF NAME DO LIST DONE

The exact `CASE` arm grammar remains to be frozen. The preferred native
form is:

    CASE WORD IN
        WHEN PATTERN [ PATTERN ... ] DO
            LIST
        WHEN PATTERN DO
            LIST
    ESAC

Preferred semantics are first matching arm wins, no fall-through, `*`
and `?` wildcard matching, and the same matcher used by pathname
globbing. Quoting should be able to make wildcard characters literal.

### 5.3 Quoting and escaping: still open

This is a language-level blocker that should be frozen before
substantial parser work.

The final specification must define:

-   available quote forms;
-   the escape character;
-   variable expansion inside each quote form;
-   command substitution inside each quote form;
-   wildcard suppression;
-   preservation of whitespace;
-   representation of an empty argument;
-   concatenation of quoted and unquoted word fragments;
-   quoting of reserved words and operators;
-   line continuation, if any.

One simplification should remain: variable expansion does not perform
POSIX/IFS-style implicit word splitting. A variable containing spaces
remains one shell argument unless an explicit language construct says
otherwise.

## 6. Variables and environment

The implementation should distinguish at least:

-   ordinary shell variables;
-   exported environment variables;
-   positional parameters;
-   special shell parameters.

Required special parameters include:

    $?
    $$
    $#
    $1 ... $9
    $NAME
    $[NAME]

The current RUN ABI should be used for environment inheritance where
available. The obsolete DAIMOSV1 assumption that RUN cannot pass an
environment must be removed.

Assignment syntax already has implementation precedent and should remain
available:

    NAME=VALUE

Shell-state commands should include at least:

    SET
    UNSET
    EXPORT

Function-local variables remain an open semantic question. A small
`LOCAL` or ksh-inspired `TYPESET` mechanism may be added if it can be
implemented with a bounded call-frame model. It should not be
implemented until the global/exported variable model is stable.

No IFS-style implicit field splitting is planned.

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

The remaining policy decisions are:

-   job numbering syntax;
-   job-reference spelling;
-   exact `JOBS` output;
-   notification timing for completed/stopped background jobs;
-   default stdin policy for background jobs;
-   retention/removal policy for completed job records.

The job table must be fixed-size. Exhaustion must produce a clear shell
error rather than corrupting state.

DSH should use kernel sessions and domains as defined by DAIMOS. It must
not create a second shell-specific session/domain abstraction.
No SessionFS is required: sessions and process groups are native kernel state,
and DSH consumes the normal kernel interfaces directly.

## 10. Command substitution

Syntax is reserved:

    $( LIST )

The implementation should eventually execute a complete DSH command list
and capture its standard output through a pipe.

Still to define:

-   maximum captured output;
-   record/newline normalization;
-   treatment of trailing newlines;
-   nesting;
-   whether command substitution is allowed in every word context;
-   failure/status interaction.

The implementation must be bounded. No unbounded capture buffer is
acceptable.

## 11. Globbing

DAIMOSV1 already provides useful precedent for `*` and `?`.

Initial DSH should support:

    *
    ?

Bracket expressions and more elaborate ksh pattern operators are
deferred.

Globbing should be performed after variable expansion and after quote
processing has identified literal wildcard characters. No-match behavior
must be explicitly specified before implementation; it should not
silently depend on the DAIMOSV1 accident of implementation.

The same bounded wildcard matcher should be reusable for CASE patterns.

## 12. Interactive input, history, and editing

The CTY baseline is transcript-oriented. DSH must not require cursor
addressing, curses, ANSI escape sequences, or a terminal database.

DAIMOSV1 provides precedent for:

-   line editing;
-   small history;
-   TAB completion;
-   history recall;
-   Ctrl-D handling.

The first implementation should keep editing deliberately small and
bounded.

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
is tightly coupled to shell execution.

Expected builtins:

    EXIT
    CD
    SET
    UNSET
    EXPORT
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

Profile execution exists conceptually but requires a clean native
implementation.

Still to define:

-   system profile path;
-   user profile path;
-   login-shell versus every-interactive-shell policy;
-   error handling;
-   whether noninteractive shells source anything implicitly.

Profile files must use the native SIXBIT record representation. Do not
reintroduce a byte-text fallback merely to enable profiles.

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
-   implement standard `-H` handling;
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

### 19.3 `-H`

`-H` is common short help and may be implicit for compliant commands.

Generated short help contains at least:

-   command name and one-line summary;
-   generated synopsis;
-   option spellings and required metavariables;
-   short option descriptions;
-   direction to `MAN` for full documentation.

Programs must not carry a second hand-maintained `-H` text table.

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

Possible behavior:

-   `HELP COMMAND` displays the generated short help;
-   without a command, it explains how to use DSH/MAN or lists basic
    help entry points.

It should consume generated descriptors where possible.

### 20.3 APROPOS / WHATIS

These should use generated indexes rather than scanning every full
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

The recommended implementation order for a new chat/work session is:

### Stage 0 - establish the base

-   unpack the newest DAIMOS, DAIMOSV1, and pdp10-doc archives;
-   record git base hashes;
-   build and run the current testkit;
-   inspect current RUN/WAIT/process-control structures;
-   do not rely on obsolete ABI text in old DSH documentation.

### Stage 1 - import the DSH core architecture

-   bring over the modular SIXBIT, lexer, parser, environment, and
    evaluator structure;
-   adapt types and libc boundaries to current DAIMOS;
-   keep fixed arenas;
-   get simple external commands and shell-state builtins working.

### Stage 2 - freeze language blockers

Before expanding the parser, finalize:

-   quoting/escaping;
-   CASE arm syntax;
-   no-match glob behavior;
-   basic variable/export semantics.

Add parser tests for every frozen rule.

### Stage 3 - execution and redirection

-   current RUN descriptor mapping;
-   `<`, `>`, `>>`;
-   correct close/error paths;
-   PATH lookup;
-   environment passing;
-   statuses 126/127.

### Stage 4 - pipelines

-   arbitrary bounded pipeline chain;
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
-   implement `-H`;
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

### Stage 11 - profiles and command substitution

Implement after the core shell is stable because both exercise
nested/current-shell execution and native record input.

## 24. Testing requirements

DSH requires host and target tests.

### 24.1 Lexer/parser tests

Cover:

-   every keyword/operator;
-   quoting/escaping once frozen;
-   redirections;
-   pipelines;
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
-   `-H`;
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

## 25. Open decisions to settle during implementation

The following remain genuinely open and should be tracked explicitly
rather than decided accidentally in code.

A. Exact SIXBIT quoting and escaping rules.

B. Exact CASE arm syntax. Preferred form is `WHEN ... DO`.

C. Function-local variable semantics and whether `LOCAL`/`TYPESET`
exists.

D. Command-substitution capture and newline rules.

E. Profile paths and startup policy.

F. Job reference syntax and exact job-status display.

G. Trap/event language, after basic job control is working.

H. Whether a DSH worker image is worthwhile for functions/builtins
inside pipelines.

I. Final overlay boundaries, based on measured linked sizes.

J. Exact compiled completion/getopt/index record formats and any later
refinements to SIXMD. The architecture is fixed; binary/record layout is
not.

K. Globbing no-match behavior.

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
