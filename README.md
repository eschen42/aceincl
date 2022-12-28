# Icon "inclusion library"

Incorporate these files into Icon programs using the `$include` preprocessor macro:

- This may be readily achieved by including this directory
  in your `LPATH` environment variable.
- This may be require less preparation and updating than would translating
  to "ucode" and using the `link` directive.

development repo: [https://chiselapp.com/user/eschen42/repository/aceincl](https://chiselapp.com/user/eschen42/repository/aceincl)

mirror and release repo: [https://github.com/eschen42/aceincl](https://github.com/eschen42/aceincl)

Except where otherwise noted, this code is created (with others' inspiration) by Art Eschenlauer ([OrcID 0000-0002-2882-0508](https://orcid.org/0000-0002-2882-0508)).

The usual [Icon value type abbreviations](http://www2.cs.arizona.edu/icon/refernce/misc.htm#datatypes) apply on this page:

- cset(`c`)
- file(`f`)
- integer(`i`)
- list(`L`)
- null(`n`)
- procedure(`p`)
- real(`r`)
- string(`s`)
- co-expression(`C`)
- record types(`R`)
- set(`S`)
- table(`T`)

with one addition:

- VNom(`V`)
  - see `vnom.icn` below.

---

<a name="contents"></a>

## Contents

- [Testing program `runt.icn` and working examples](#testing-program-runticn-and-working-examples)
  - The `runt.icn` runs a selection of, or all, test programs in specified directories to validate their outputs:
  - In the `tests` directory are "working examples" of how to use the files in this directory.
  - In the `sl3tests` directory are sqlite3-dependent "working examples".

- [`baton.icn`](#batonicn)
  - Procedure to coordinate passing data from one process to another via a file-based buffer.

- `baton_main.icn` - Procedures to use facilitate creation of processes that exchange data
    with current process via batons.
    - [`baton_main`](#procedure-baton_mainargs--exit0--1) Procedure for creating executable to interface between a batons and a stream.
    - [`baton_flatware`](#procedure-baton_flatwareargs--fail--exit0--1) Procedure to access batons without translating an additional executable.
    - [`baton_crowbar`](#procedure-baton_crowbar--n--stop) Procedure to handle programming error by terminating program exection when `baton_flatware` is not linked.

- [`batonsys.icn`](#batonsysicn) - Invoke process using batons to exchange input and output

- [`fieldedDataFile.icn`](#fieldeddatafileicn)
  - Procedures to produce logical lines or fields from formatted data files.

- [`fileDirIo.icn`](#filedirioicn)
  - Procedures to manipulate files, directores, and their paths.

- [`iimage.icn`](#iimageicn)
  - Procedures to transform data structures into includable Icon declarations and statements.

- [`jsonparse.icn`](#jsonparseicn)
  - Procedures to parse and generate JSON, by Carl Sturtivant and Gregg Townsend.

- [`LiComboP.icn`](#licombopicn)
  - Procedures to suspend lists combining sequences.

- [`lindel.icn`](#lindelicn)
  - In-place delete or insert of a pseudo-section of L.

- [`RecTable.icn`](#rectableicn)
  - Procedures to produce/manipulate record-like tables.

- [`rpn.icn`](#rpnicn)
  - Procedures to embed RPN-based (Forth-like) interpreter into Icon programs; can also be run in REPL.

- [`runningStats.icn`](#runningstatsicn)
  - Support computing summary statistics for normally distributed data using "Welford's online algorithm".

- [`selectRecordFromListByField.icn`](#selectrecordfromlistbyfieldicn)
  - Procedure to produce records from a list of records (or a list of tables), matching specified criteria.

- [`sl3.icn`](#sl3icn)
  - Interface to exchange commands and results with `sqlite3`.

- [`vnom.icn`](#vnomicn)
  - "Nominal vector", i.e., a list whose elements may be accessed by rank (index) or name (key).
  - This construct is supported by a Lua-inspired metatable.

- [`wora.icn`](#woraicn)
  - Procedure to produce a value that can be read globally but can be reset only by the co-expression that set it it initially.

- [Legacy Source Code Control](#legacy-source-code-control)

---

<a name="testing-program-runticn-and-working-examples"></a>

## Testing program `runt.icn` and working examples

Working examples, named `test_*.icn`, are in the `tests` directory:

- `test_*.std` captures the corresponding test's expected output.
- `icon runt.icn tests` will run the tests and compare the results to
   their expected output.
- usage: `icon runt.icn [--continue] [--verbose] [<zero or more dirs or test names>]`
  - Tests are not run unless an `.icn` file and a `.std` file share exactly
    the same name; names must begin with `test_`.
  - Optional arguments may be:
    - paths to directories in which to locate tests, or
    - names of tests to run, without `test_` prefix;
      - if *any* nonexistent paths are present then *only* the named tests will be run.
  - By default, tests are located and run in the current working directory.
    - Otherwise, tests are located and run in the specified directory or directories.
    - As usual, LPATH is required for `$include` dirctives to succeed.
  - Use the `--continue` option to run all tests regardless of whether any fail.
  - Use the `--verbose` option to show both the expected output
    from the `.std` file and the actual output produced by the test program.

---

<a name="batonicn"></a>

## baton.icn

`baton.icn` provides a serverless way to pass data between
processes using five files, one for the message transmitted and the others
for coordinating transfer.

### procedure `baton(action:s, filename:s, file:f, warn:C, wait_secs:N) : s|n|fail`

Procedure `baton` provides four distinct modes of action:

```
        action    buffer    file      Cwarn   wait_secs
  baton("write",  buffer:s, file:f|C, warn:C, wait_secs:N) : s|fail
  baton("read",   buffer:s, file:f|C, warn:C, wait_secs:N) : s|fail
  baton("select", buffer:s, file:n|x, warn:C, wait_secs:N) : n|fail
  baton("clean",  buffer:s, file:n|x, warn:C               ) : fail
```

Usages:

#### `action == "write"`

`baton("write", buffer:s, file:f|C, warn:n|C, wait_secs:N) : s | fail`

Copy input from `file` to file named by `buffer`

- `file` is optional; default is `&input`
  - `file:f` is a file open for reading
  - `file:C` is a co-expression producing string values,
    - e.g.:  `file := create !&input`
    - `file:C` is permitted to produce values containing multiple newlines
- `buffer` argument is required
- `warn` argument may be `&null` or a co-expression that can handle
  transmitted warning strings, e.g.:
  - `create repeat @&source | @main # silence; is the default behavior`
  - `create repeat write(&errout, \@&source | "") | @main`
- `wait_secs` argument may be `&null`, integer, or real, specifying
  approximate number of seconds to wait for initial handshake
- produces an error string only if unsuccessful (fails otherwise)

#### `action == "read"`

`baton("read", buffer:s, file:f|C, warn:n|C, wait_secs:N) : s | fail`

Copy output from file named by `buffer` to `file`

- `file` is optional; default is `&output`
  - `file:f` is a file open for writing
  - `file:C` is a co-expression receiving string values,
    - e.g.:  `@(file := create while write(&output, @&source))`
- `buffer`, `warn`, and `wait_secs` arguments as for "write".
- produces an error string only if unsuccessful (fails otherwise)

#### `action == "select"`

`baton("select", buffer:s, file:n|x, warn:n|C, wait_secs:N) : n | fail`

- produces `&null` only when data are available for reading from buffer
- `buffer` and `warn` arguments as for "write".
- wait (approximately) `wait_secs` seconds for success.
- `file` is ignored even if not `&null`
- This action is experimental and may be of no practical value.

#### `action == "clean"`

`baton("clean", buffer:s, file:n|x, warn:n|C) : fail`

- cleans up coordination files that may be left behind when "write" or
  "read" is interrupted before they can delete these files
- `buffer` and `warn` arguments as above
- `file` is ignored even if not `&null`

See "case two" in the example under `baton_flatware` below for
a demonstration of use of "read" and "write" from within a program.

---

## baton_main.icn

See below for notes regarding usage of these procedures.

<a name="procedure-baton_mainargs--exit0--1"></a>

### procedure `baton_main(args) : exit(0 | 1)`

This procedure provides an implementation for `main` for a standalone
executable to interface a baton to or from a stream, as described below.

- Exit code 0 indicates ordinary termination
- Otherwise, exit code is 1.

Note that Icon does not catch `SIGPIPE`.
Consequently, if `baton("read",...)` is feeding a pipe,
and if the downstream process dies,
then the Icon process running `baton("read",...)` will die as well!
This is the motivation for running an output pipe from a separate process
and coordiating data-passing using a baton.

*Usage of baton program*

```
baton read   bufferfile [handshake_timeout_secs]
baton write  bufferfile [handshake_timeout_secs]
baton select bufferfile [success_timeout_secs]
baton clean  bufferfile

  write reads from standard input into bufferfile in coordination with
      read, which copies from bufferfile to standard output.
      When supplied, handshake_timeout_secs specifies number of seconds
      to wait for initial handshake.

  select returns a zero exit code only when data are
      available to read.
      When supplied, handshake_timeout_secs specifies number of seconds
      to wait for success.

  clean removes coordination files that may remain when
      read or write experiences abnormal termination.
```

*Building baton program*

Create a file `baton_main_build.icn` containing:
```
$define baton_main main
$include "baton_main.icn"
```

Then translate:

```
icont -u -o baton     baton_main_build.icn # on Unix-like OSs
icont -u -o baton.exe baton_main_build.icn # on MS Windows
```

The resulting program can be used as in the following (trivial) example:

```
  $include "fileDirIo.icn"
  procedure main()
    # Pass one line of data from output pipe to input pipe.
    fsend := open("baton write buffer", "wp")
    frecv := open("baton read  buffer", "rp")
    write(fsend, "hello world from " || &progname)
    write(read(frecv))
    close(fsend)
    close(frecv)
  end
```

However, the ordinary way to use this program is to have "baton write"
stream one baton to the standard input of a process and to have "baton read"
stream the standard output of a process to another baton,
as demonstrated in the example for `baton_flatware` below.

<a name="procedure-baton_flatwareargs--fail--exit0--1"></a>

### procedure `baton_flatware(args) : fail | exit(0 | 1)`

This procedure facilitates creation of a "multi-entry binary";
specifically, it eliminates the necessity to create a distinct executable
to use batons outside of the parent program's process.

See `tests/test_baton_main.icn` for examples.

This procedure diverts the control to `baton_main(args)` when:

- `*args = 3`
- `args[1]` is `"baton_main"`
- `args[2]` is `"read" | "write" | "select" | "clean"`
- `args[3]` is a string (to name a buffer)

Otherwise, the procedure fails.

This allows an executable program to have two behaviors:

- Its "ordinary" behavior when not diverted to `baton_main`
- The `baton_main` behavior, invoked by the "parent" instance of the executable.

The following exemple demonstrates invocation of `baton_flatware(args)` from within `main(args)`:

```
  # example usage of baton_flatware; requires that sqlite3 is on PATH
  $define BATON_TRACE if &fail then write
  $define BATON_CWARN create repeat write(&errout, \@&source | "") | @&main
  $define BATON_TIMEOUT_MS 200
  $include "baton_main.icn" # implies include baton.icn and fileDirIo.icn
  $define PLUGH  "A hollow voice says \"plugh\"."
  $define PLOVER "end of SQLite result list"
  procedure main(args)
    local baton_self # string to invoke child "flatware" via baton_flatware
    local chunk      # temporary holder for a string of data
    local status     # exit status of child process, captured from Cexit
    local Cexit      # co-expression producing exit code if child "flatware"
                     #   has terminated; producing &null otherwise
    local Crecv      # co-expression to receive input from child "flatware"
    local Csend      # co-expression to send output to child "flatware"
    # handle calls to baton_main; does not return; kind of like a fork...
    baton_flatware(args)
    # path to self is platform-specific; this has been minimally tested!
    baton_self := &progname
    if not (&features == "MS Windows" | path_separator() == baton_self[1])
      then baton_self := "." || path_separator() || &progname
    baton_self ||:= " baton_main "
    # Exchange data with external process via batons.
    #   Launch process in background; if process dies, no SIGPIPE
    #   can reach us (see: https://unix.stackexchange.com/a/84828)
    #   which is good because Icon does not catch signals.
    Cexit :=
      system_nowait(
        baton_self || " read buf_in | " ||
        "sqlite3 -batch -json | " ||
        baton_self || " write buf_out"
        )
    # Set up baton for standard output of process
    Crecv := create baton("read", "buf_out", &main)
    # Set up baton for standard input of process
    # and activate baton so that it can receive a value
    @( Csend := create baton("write", "buf_in", &main) )
    # Send a command to SQLite, producing output
    ".show" @Csend
    # Send a query to SQLite, not producing any output
    "select 'plover' where 1 = 0;" @Csend
    # Send a query to SQLite to mark end-of-output
    "select 'plugh' as xyzzy;" @Csend
    # Retrieve lines until end-of-output mark (or closed pipe)
    while chunk := @Crecv
      do
        if chunk == "[{\"xyzzy\":\"plugh\"}]"
          then break write(PLUGH)
          else write(chunk)
    # Send "[]" to SQLite to mark end-of-output
    ".print '[]'" @Csend
    # Retrieve lines until end-of-output mark (or closed pipe)
    while chunk := @Crecv
      do
        if chunk == "[]"
          then break write(PLOVER)
          else write(chunk)
    # Close the output baton
    char(4) @Csend
    # Close the input baton to clean up baton files.
    @Crecv
    # Wait (briefly) for process exit and retrieve exit code.
    every 1 to 5
      do {
        write("SQLite exit code: ", image(status := @Cexit))
        if /status
          then delay(BATON_TIMEOUT_MS)
          else break
        }
  end
```

You will find another example under [`sl3.icn`](#sl3icn).

<a name="procedure-baton_crowbar--n--stop"></a>

### procedure `baton_crowbar() : n | stop()`

Kill the program when `baton_flatware` is not linked (which is a
programming error) to avert "infinite forking" that consumes all slots
in the process table.

This approach is imperfect since one may circumvent it with
```
  invocable all
```
or with
```
  invocable baton_flatware
```

---

<a name="batonsysicn"></a>

## batonsys.icn

# procedure `baton_system() : V`

`baton_system(basename, cmd, inC, outC) : BatonSys` (a VNom extension)

- This procedure is a VNom initializer producing a VNom that implements
  BatonSys message handling; i.e., BatonSys extends VNom (from `vnom.icn`)
  such that BatonSys-specific messages are invoked via the vmsg procedure.
- This procedure is supported by `VNomBatonSysCtor` and  `VNomBatonSysMesg`,
  neither of which need to be invoked directly. The BatonSys-specific
  VNom messages (handled by `VNomBatonSysMesg`, as described below) are:
  - `create`
  - `send`
  - `receive`
  - `dispose`
  - `select`

# procedure `VNomBatonSysCtor() : V`

`VNomBatonSysCtor(Original:T, Type:s, ID:s, Metatable:T, Disposable:n|x, Kind:s):BatonSys` (a VNom extension)

- This procedure provides the implementation for `baton_system` initialiaztion.
  VNom initializer producing a VNom that implements BatonSys
  message handling; i.e., BatonSys extends VNom (from `vnom.icn`)
  such that BatonSys-specific messages are invoked via the vmsg procedure.
- `Original:T`
  - If not null, a plain table, or another VNom (or extension
  of VNom), from which values are copied.
- `Type:s`
  - "Type" property, if `&null` then Type of `\Original`;
  default is `Kind || typecount`.
- `ID:s`
  - "ID" property; this is required (in contrast to VNom,
    which synthesizes a default).
- `Metatable:T`
  - Table that maps message strings to message-handler procedures;
    when `&null`, the metatable from Original is assigned, if available; otherwise,
    a default metatable is created; in any event, the BatonSys messages.
    are added to the metatable.
- `Disposable:n?`
  - Disposability flag; when not `&null`, a "Disposable" property is added and
    set to "yes", to be switched to "done" when the VNom has been disposed.
- `Kind:s` "Kind" property, defaults to "BatonSys".


# procedure `VNomBatonSysMesg() : x`

`VNomBatonSysMesg(args[]):x`

- This procedure is a VNom message-handler extension for BatonSys messages.
- Extensions to VNom messages:
```
    vmsg(V, "create", s, C, C ) : C  # CreateProcess, producing result
                                     #   from system_nowait
                                     #   arg1: command string
                                     #   arg2: C providing stdin
                                     #   arg3: C receiving stdout
    vmsg(V, "send",   s       ) : n  # Send s to stdin of child pro-
                                     #   cess from system_nowait
    vmsg(V, "receive"         ) : s  # Receive s from stdout of child
                                     #   process; not assignable
    vmsg(V, "select"          ) : n  # Produce n if input is ever
                                     #   pending from stdout of child
                                     #   TODO determine when it's TRUE
    vmsg(V, "dispose"         ) : i  # Terminate child process; produ-
                                     #   cing exit code
```
- Extensions to VNom state:
```
    buf_in  - base filename for baton handling data to child stdin
    buf_out - base filename for baton handling data from child stdout
    cmd     - system_nowait command string, minus baton pipes
    status  - "running" | "closed" | &null (disposed)
    whichme - path to current executable
    Cexit   - co-expression producing child's exit code; otherwise, &null
    Csend   - co-expression providing stdin to child
    Crecv   - co-expression receiving stdin from child
```

---

<a name="fieldeddatafileicn"></a>

## fieldedDataFile.icn

Procedures to produce logical lines or fields from formatted data files.

#### record `FieldedData(lines, fields)`

Produce record holding two co-expression factories:

- `lines` === tabularLines | iniLines
- `fields` === tabularFields | iniFields

### procedure `FieldedDataFactory(format, filePath) : FieldedData`

Produce a `FieldedData` record for `filePath` corresponding to format.

- `format == ("tabular" | "ini")`

### procedure `tabularLines(f) : C`

Factory for a co-expression producing logical lines of a tabular file `f`.

### procedure `tabularFields(line, sep) : C`

Factory for a co-expression producing fields from a logical line of a tabular file:
- `line` is a logical line produced by `tabularLines`.
- `sep` is the field separator; if omitted or &null, TAB is used.

### procedure `getTabular(typeName, tsvPath, colL, sep, dflt) : L`

Produce L of RecTable from a tabular file

- `typeName`: the first result to be produced by RecTableType(T), s
- `tsvPath` : the path to the tabular data file, s
- `colL`    : (optional) columns to select, L of i
- `sep`     : (optional) separators, c
- `dflt`    : (optional) default value for RecTable fields, x

### procedure `iniLines(f)` : C

Factory for a co-expression producing logical lines of an INI file `f`.

### procedure `iniFields(line) : C`

Factory for a co-expression producing fields from a logical line of an INI file

- `line` is a logical line produced by `iniLines`.

### procedure `getIni(ini) : T` (two dimensional)

Parse an INI file at path `ini` into a table of tables

---

<a name="filedirioicn"></a>

## fileDirIo.icn

Procedures to manipulate files, directores, and their paths.

### procedure `alterExtension(fn, old_ex, new_ex) : s1, ...`

Generate modified `fn`, substituting `new_ex` for `old_ex`

- If `new_ex` is "", the trailing period will be removed.

### procedure `cmd_separator() : s`

Produce platform-specific command separator

### procedure `directory_seq(name) : s1, ...`

Generate name(s) that name a directory

### procedure `home() : s`

Produce platform-specific path to the HOME directory, if available

### procedure `path_atoms(path) : s1, ...`

Generate root, subdirectories, filename for a directory path

### procedure `path_constructP{exprs} : s1, ...`

Generate paths from sequences of results of exprs

- `exprs` are comma-separated and used to create a list of co-expressions.

### procedure `path_parts(qualname) : s1, s2`

Generate location then name from path `qualname`

### procedure `path_separator() : s`

Produce platform-specific path separator

### procedure `prog_path_parts() : s1, s2`

Generate location then name of program file.

### procedure `pwd() : s`

Produce platform-specific path to the current directory

### procedure `system_nowait(command:s, title:s) : C`

Run command, but do not wait for exit, producing result C

- `command`, command to be passed to shell
- `title`, title for background window, optional, for MS Windows
- `@result` produces `&null` before command exits; exit code, after termination.
  - Please invoke `@result` till it does not produce `&null` to
    delete the file that holds the exit code.

### procedure `tmpdir() : s`

Produce platform-specific path to a tmp directory

### procedure `tmppath() : s`

Generate platform-specific temporary file path(s)

### procedure `which(filename:s, all:n|x) : s1, ...`

Generate full path(s) for filename on PATH

- on Unix, results are first (or all when `\all`) for `which -a`.
- on Windows, results are first (or all when `\all`) for `where`.

---

<a name="iimageicn"></a>

## iimage.icn

Procedures to transform data structures into includable Icon declarations and statements.

### procedure `iimage(x)` : s

  - Produce Icon code to reproduce value `x`, if possible

### procedure `idump(f, x[])` : (writes to `\f | &errout`)

  - Write Icon code to reproduce values in list `x` to `f` if it is a file;
    otherwise to `&errout` and `f` is discarded.

---

<a name="jsonparseicn"></a>

## jsonparse.icn

Procedures to parse and generate JSON, by
[Carl Sturtivant](https://www-users.cse.umn.edu/~carl/) ([OrcID 0000-0003-1528-4504](https://orcid.org/0000-0003-1528-4504))
and [Gregg Townsend](https://www2.cs.arizona.edu/~gmt/).

### procedure `json(L|T|i|n|r|s) : s`

  - Takes data (list|table|integer|string|real|&null) and produces a JSON
    string defining that data.  It is an error to use another type, even
    in substructures. See http://json.org/.  To serialize other types,
    see codeobj.icn from the Icon Programming Library.

### procedure `jsonparse(s) : x`

  - Takes a JSON string and produces the corresponding Icon value or
    structure. Tables in such a structure will have default values of null.
    JSON text containing true and false (booleans) will have those converted
    to the strings "true" and "false" respectively.

### procedure `jsonIconstringencoding(s) : n`

  - Fixes the encoding of Icon strings (NOT quoted strings embedded in
    JSON text) for all subsequent calls of json and jsonparse until
    jsonIconstringencoding is called again. The initial behavior of
    json and jsonparse is as if `jsonIconstringencoding("UTF-8")` has
    been called. Fails unless `map(s)` is either "utf-8" or "utf8" or
    "latin1" or "latin-1".

---

<a name="licombopicn"></a>

## LiComboP.icn

Procedures to suspend lists combining sequences.

### procedure `LiP(A) : L1, ...`

  - Suspend lists combining infinite sequences. LiP:
    - evaluates in a "breadth first" manner to ensure that all values of finite
      sequences will eventually be produced even when some sequences are infinite.
    - uses memoization to avert the need to evaluate each sequence more than once.
    - uses wora(LiP) to determine whether to use LiFiniteP
      (the default) or nAltP to combine memoized results.
    - requires that `wora.icn` be previously included, for wora(id)

### procedure `LiFiniteP(LofC) : L1, ...`

  - Recursively suspend lists combining finite seqs;
    - does not enforce "breadth first" evaluation.

### procedure `nAltP(LofC) : L1, ...`

  - Recurrently suspend lists combining finite seqs;
    - does not enforce "breadth first" evaluation.

---

<a name="lindelicn"></a>

## lindel.icn

In-place delete or insert of a pseudo-section of L.

### procedure `Ldelete(L, i, j) : L`

Delete indexes `i` to `j` from `L` (in-place), producing `L`

### procedure `Linsert(L, i, Lins) : L`

Insert list `Lins` into `L` (in-place) before index `i`, producing `L`

- use i = 0 when L is empty

### procedure `Lfind(L, x) : i1, i2, ...`

Generate indices where `x` appears in `L`

---

<a name="rectableicn"></a>

## RecTable.icn

Procedures to produce/manipulate record-like tables.

See also: `vnom.icn` below for another, potentially more flexible approach.

### procedure `RecTable(rec_name_s, rec_fields_L, rec_data_L, rec_default_x) : T`

Produce a table with record-like aspects:

- `rec_name_s`:    the "type" of the RecTable
- `rec_fields_L`:  a list of the field names
- `rec_data_L`:    an optional list of values to assign to the fields
- `rec_col_iL`:    an optional list of column numbers to choose
                 defaults to all
- `rec_default_x`: default value for table members

### procedure `RecTableType(x) : s1, S2, s3, ...`

For RecTable, produce:

- name
- set of all fields
- each field

For non-RecTable, return type(x).

### procedure `RecTableFields(x) : s1, ...`

Produce RecTable's field names.

- This will fail for a non-RecTable.

### procedure `RecTableFieldsL(x) : `L

Return a list of the values produced by RecTableFields(x).

- This returns an empty list when x is not a RecTable instance.

### procedure `RecTableFieldVals(x) : s1, ...`

Produce RecTable's field values.

- This will fail for a non-RecTable.

### procedure `RecTableFieldValsL(x) : L`

Return a list of the values produced by RecTableFieldVals(x).

- This returns an empty list when x is not a RecTable instance.

### procedure `RecTableColTypeCheck(x, type_name, col_name, preamble) : x`

Return x, except abort when x is not instance of `type_name`:

- `x`        : value whose type is to be checked
- `type_name`: expected string for RecTableType(x)
- `col_name` : name of identifier-under-test
- `preamble` : initial string for error message; defaults value of name
             RecTablePreamble.

### procedure `RecTableConstructorC(rec_name_s, rec_fields_L, rec_default_x) : C`

Produce a C that, when receiving a transmitted list of values (of the
same length as `rec_fields_L`), produces a RecTable instance:

- `rec_name_s`    the "type" of the RecTable
- `rec_fields_L`  a list of the field names
- `rec_col_iL`    an optional list of column numbers to choose,
                  defaults to all
- `rec_default_x` default value for table members

---

<a name="rpnicn"></a>

## rpn.icn

Procedures to embed RPN-based (Forth-like) interpreter into Icon programs; can also be run in a [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop).

This file may be used to embed RPN-scripted access to Icon procedures
and operators, in a manner reminiscent of Forth.
- This needs full and user-friendly documentation, which deserves its own file.
- Words may be composed from existing words.
  - Words could be composed to conform to Forth standards...
  - New word definitions replace former definitions.
  - There is no `forget` yet.
  - The colon in a definition goes *after* the string naming the new word, e.g.,
    - `"hi" : "hello world" . cr ;`

See the following to get started:
- `tests/test_rpn.icn`
- `tests/test_rpn.rpn`
- `rpn_core.rpn`
- `rpn.icn`

This work was inspired by Steve Wampler's "A (small) RPN calculator" example
[https://sourceforge.net/p/unicon/mailman/message/6144067/](
https://web.archive.org/web/20211011204544/https://sourceforge.net/p/unicon/mailman/message/6144067/)
and R. G. Loeliger's *Threaded interprtive languages* (1981, BYTE Books)
[https://lccn.loc.gov/80019392](https://lccn.loc.gov/80019392).


### run rpm.icn with REPL

To run this "stand-alone" in a "read, evaluate, print, loop" (REPL),
I put the following in `~/bin/rpn`:

```
LPATH=~/src/aceincl icon -P '
# run rpn.icn

# Define procedure main(args) that imports:
#   - ~/.rpn/*.rpn
#   - any .rpn files specified as arguments
# and executes standard input if either:
#   - *args = 0
#   - "-" == !args
$define RPN_MAIN 1
$include "rpn.icn"

# required by rpn.icn
$include "fileDirIo.icn"
'
```

then I made it executable:

```bash
chmod +x ~/bin/rpn
```

so that I can run it with:

```bash
~/bin/rpn
```

If you have rlwrap
([https://github.com/hanslub42/rlwrap](https://github.com/hanslub42/rlwrap))
installed (or built), and you change the first line above to
```
LPATH=~/src/aceincl rlwrap icon -P '
```
then you can get proper interpretation of the arrow keys in the REPL loop.

---

<a name="runningstatsicn"></a>

## runningStats.icn

These procedures support computing summary statistics for normally
distributed data using "Welford's online algorithm", porting code
from Wikipedia.

ref: [https://en.wikipedia.org/wiki/Algorithms\_for\_calculating\_variance#Welford's\_online\_algorithm](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Welford%27s_online_algorithm)

### record `welford_running(count, mean, M2)`

- record accumulating online results without persisting raw data

### record `welford_cumulative(n, mean, variance, sampleVariance, SD, SE)`

- record of statistical results extracted from `welford_running`

### procedure `welford_new()`

- produce an initialized `welford_running` record

### procedure `welford_add(W, x)`

- produce an updated `welford_running` record
  - `W` a `welford_running` record
  - `x` the next value to add to the record

### procedure `welford_get(welford_running)`

- produce `welford_cumulative` record summarizing normal statistics
  for the series of x provided to `welford_add`
    - `welford_running` a `welford_running` record updated by `welford_add`

---

<a name="selectrecordfromlistbyfieldicn"></a>

## selectRecordFromListByField.icn

Procedure to produce records from a list of records (or a list of tables), matching specified criteria.

- This is currently NOT coded efficiently, so only use it on small lists or tables.
- Also, at present, the co-expression is refreshed and (as an apparent consequence) leaks memory.
- For a better way to do this, see [fix_selectRecordFromListByField.icn](https://sourceforge.net/p/unicon/mailman/attachment/CACb17F7MTimKsVuYS0LCmB6CpuE1pLvTZBtdCiNZRJEzFsFeKA%40mail.gmail.com/1/), which I will eventually apply here instead.
  - Even so, there is yet the need to incorporate a "fuzzy binary search" to speed things up immensely for larger lists or tables.

### procedure `selectRecordFromListByField(Lfrom, sField, Ctest) : R1, ...`

- Produce matching records (or tables) `X`
  - from list `Lfrom` (`type(Lfrom[i]) == "record" | "table"`)
  - where `X[sField] @ Ctest` succeeds

### procedure `selectRecordFromListByFieldL(Lfrom, sFieldL, Ctest) : R1, ...`

- Produce matching records (or tables) `X`
  - from list `Lfrom` (`type(Lfrom[i]) == "record" | "table"`)
  - where this succeeds:<br />
    `L := []; every put(L, X[!sFieldL]); L @ Ctest`

---

<a name="sl3icn"></a>

## sl3.icn

Interface to exchange commands and results with `sqlite3`, which must be on PATH.

`sl3.icn` defines an interface to the sqlite3 command line interface,
which is described in detail at https://sqlite.org/cli.html

No generalized process interface exists in the basic Icon implementation
and library that would allow access to both the standard input and the
standard output of a child process, this interface uses the `baton`
inteface via `baton.icn`, `baton_main.icn`, and `batonsys.icn` to hand
data back and forth between Icon and sqlite3 without resorting to such
platform-specific devices such as FIFOs or named-pipes.

For convenience, if the symbol `sl3` is not defined by the preprocessor
before `sl3.icn` is included, then it is defined as `sl3Msg`:

```
$ifndef sl3
  $define sl3msg sl3
$endif
```

`sl3new(path, options, errC) : VNom` (a SQLite3 connection)
  - This is a synonym for `sl3(&null, "open", path, options, errC)`
  - See below for details of the "open" message.

`sl3Msg(conn:SQLite3, message, arg1, arg2, arg3) : x`
  - takes as its first argument, a "database connection" that is a
    "SQLite3" VNom (or `&null` when the message is "open").
  - takes as its second argument an operation message string, which is:
    - either one of `"open" | "prepare" | "fetch" | "close"`
    - or "the default key", a prepared statement or SQL string.
  - The value produced depends on the signature invoked.

The signatures of sl3 messages are as follows:

```
  sl3(&null, "open",      path,    options, errC ) : VNom (a SQLite3 conn)
  sl3(conn,  "prepare",   stmt                   ) : VNom (a prep_stmt)
  sl3(conn,  stmt:s,      parmL:L, errC          ) : n|fail
  sl3(conn,  prep_stmt:V, &null|x, errC          ) : n|fail
  sl3(conn,  "fetch"                             ) : VNom (a result row)
  sl3(conn,  "close",     errC                   ) : &null
```

You can find some examples with (modest) error handling in `sl3tests/test_sl3_02.icn`.

In lieu of a detailed description of each signature (which may be found the header of `sl3.icn`), here is a working example, which emphasizes several ways of passing parameters to prepared statements.

```
$include "fileDirIo.icn"
$include "vnom.icn"
$include "jsonparse.icn"
$include "sl3.icn"
$define ERR_OFFSET [0, -1][2]
$define DISPOSE_ON_ERROR if write(trace_reset(err_off)) then dispose(cnxn, errC)
global g_exit_code
procedure main(args)
  local cnxn, path_s, options_s, errC, chunk, err_off, prep_stmt
  # don't forget this or infinite forks will fill the process space... 
  baton_flatware(args)
  # set exit code for premature termination
  g_exit_code := -1
  # create co-expression to handle error strings
  @(errC := create while write(@ &source))
  # open sqlite and establish baton
  path_s := ":memory:"
  options_s := &null
  # open database connection
  #   signature: sl3new(path, options, errC) : VNomSQLite3
  #     === sl3(connection, "open", path, options, errC) : VNomSQLite3
  if not (cnxn := sl3new(path_s, options_s, errC))
    then stop("failed to open connection")
  err_off := ERR_OFFSET

  # execute SQL immediately, without parameters
  #   signature: sl3(conn, stmt:s, &null, errC) : n|fail
  sl3(cnxn,
    "CREATE TABLE invent(desc text, number integer, amount float);", , errC)
  DISPOSE_ON_ERROR
  # execute dot command immediately
  sl3(cnxn, ".dump", , errC)
  DISPOSE_ON_ERROR
  # show result lines, which are not rows from a SQL query
  every chunk := sl3(cnxn, "fetch") do write_ordered_values(chunk)

  # initialize first prepared statement, which has named parameters
  #   signature: sl3(conn, "prepare", stmt) : VNom (a prep_stmt)
  prep_stmt := sl3(cnxn, "prepare",
    "INSERT INTO invent VALUES(@desc, @number, @amount);")
  DISPOSE_ON_ERROR
  # set parameters for first prepared statement
  vmsg(prep_stmt, "put", "@desc", "a description")
  vmsg(prep_stmt, "put", "@number", 42)
  vmsg(prep_stmt, "put", "@amount", 3.14159)
  # execute first prepared statment
  #   signature: sl3(conn, prep_stmt:V, &null, errC ) : n|fail
  if not sl3(cnxn, prep_stmt, , errC) then {
      write("test one: execute prepared statement with named parameters failed")
      dispose(cnxn, errC)}

  # initialize second prepared statement, which has named and unnamed parameters
  #   signature: sl3(conn, "prepare", stmt) : VNom (a prep_stmt)
  prep_stmt := sl3(cnxn, "prepare", "INSERT INTO invent VALUES(?, ?3, ?2);")
  DISPOSE_ON_ERROR
  # set parameters for second prepared statement
  vmsg(prep_stmt, "put", "?1", "another description")
  vmsg(prep_stmt, "put", "?2", 1066)
  vmsg(prep_stmt, "put", "?3", 1.414214)
  # execute second prepared statment
  #   signature: sl3(conn, prep_stmt:V, &null, errC) : n|fail
  if not sl3(cnxn, prep_stmt, , errC)
    then {
      write("test two: execute prepared statement with unnamed parameters failed")
      dispose(cnxn, errC)}
  DISPOSE_ON_ERROR

  # execute SQL with implicit prepared statement and with L of unnamed params
  #   signature: sl3(conn, stmt:s, parmL:L, errC) : n|fail
  if not sl3(cnxn, "INSERT INTO invent VALUES(?, ?, ?);",
    ["foobar", 2401, 21.0/7], errC)
    then { write("test three: execute implicit prepared statement failed")
      dispose(cnxn, errC)}
  DISPOSE_ON_ERROR

  # execute SQL without params
  #   signature: sl3(conn, stmt:s, &null, errC) : n|fail
  sl3(cnxn, "select * from invent;", , errC)
  DISPOSE_ON_ERROR
  # show results from previous SQL
  write("---")
  every chunk := sl3(cnxn, "fetch") do {
    write_vnom_fields(chunk)
    write("---")}

  # set exit code for normal termination
  g_exit_code := 0
  &error := 0
  # stop sqlite3 and shut down the batons
  dispose(cnxn, errC)
end
procedure dispose(cnxn, errC)
  # stop sqlite3 and shut down the batons
  sl3(cnxn, "close", errC) | write("sl3 \"close\" failed")
  write("  batonsys disposition: ", image(cnxn["disposition"]))
  exit(g_exit_code)
end
procedure trace_reset(error_offset)
  local result
  result := ""
  if &error = 0 then fail
  if (&error < error_offset) then {
      result ||:= "There were " || error_offset - &error || " errors; "
      result ||:= "last error " || &errornumber || " - " || &errortext
      &error := error_offset
      return result
      }
  &error := error_offset
end
procedure write_vnom_fields(chunk, f)
  local i
  /f := &output
  # row fields are ordered by VNom key
  every i := vmsg(chunk, "key") do write(f, "  ", i, ": ", chunk[i])
  return
end
procedure write_ordered_values(chunk, f)
  local i
  /f := &output
  # results of commands are ordered by a discardable integer key
  every i := key(chunk) do if i ~=== chunk then write(f, "  ", chunk[i])
  return
end
```

which produces as output:

```
  PRAGMA foreign_keys=OFF;
  BEGIN TRANSACTION;
  CREATE TABLE invent(desc text, number integer, amount float);
  COMMIT;
---
  desc: a description
  number: 42
  amount: 3.14159
---
  desc: another description
  number: 1.414214
  amount: 1066.0
---
  desc: foobar
  number: 2401
  amount: 3.0
---
  batonsys disposition: 0
```

---

<a name="vnomicn"></a>

## vnom.icn

"Nominal vector", i.e., a list whose elements may be accessed by rank (index) or name (key).

A use case for this construct might a dynamically-defined record
(i.e., an ordered list of key-value pairs),
such as might be used to represent a row returned by an SQL query.

Through use of a Lua-inspired "metatable", operations on this structure
may be defined or extended with a few message-handler-extension functions.
Note that a metatable may be shared among several VNom instances to give
them identical behavior; for this reason, the copy constructor copies
a reference to the metatable rather than making a copy of the metatable.
Thus, behavior of the set of instances may (even dynamically) be modified
by changing a single structure.

### procedure `vnew(Original:T, Type:s, ID:s, Metatable:T, Disposable:s, Kind:s) : V`

Construct a new VNom instance.

  - `Original:T`
    - `&null` | `table` | `VNom`
  - `Type:s`
    - Type property
  - `ID:s`
    - ID property
  - `Metatable:T`
    - table mapping message strings to message-handler procedures
    - By design, when `Original` is another VNom, `vnew` copies a reference
      to the metatable rather than the metatable itself.
  - `Disposable:n?`
    - if not null, set Disposable property to "yes"
  - `Kind:s`
    - Kind property

### procedure `vmsg(VNom:V, Message:s, args[]) : x`

Send messages to update or interrogate the VNom instance.

  - `vmsg(V, "!"                 ) : x1, ...  # generate values in order keys in L   `
  - `vmsg(V, "*"                 ) : i        # produce number of values             `
  - `vmsg(V, "get" | "pop"       ) : x        # pop value, discarding key            `
  - `vmsg(V, "copy"              ) : V        # produce copy of V with same metatable`
  - `vmsg(V, "pull"              ) : x        # pull value, discarding key           `
  - `vmsg(V, "push",       xk, xv) : V        # push (or replace) value x2 for key x1`
  - `vmsg(V, "put",        xk, xv) : V        # put (or replace) value x2 for key x1 `
  - `vmsg(V, "key"               ) : x1, ...  # generate keys in order keys in L     `
  - `vmsg(V, "keylist"           ) : L        # copy of L of ranked keys             `
  - `vmsg(V, "bykey",      x     ) : s        # value, assignable by key             `
  - `vmsg(V, "byrank",     i     ) : s        # value, assignable by rank (index)    `
  - `vmsg(V, "delbykey",   x     ) : n        # delete value by key                  `
  - `vmsg(V, "delbyrank",  i     ) : n        # delete value by rank                 `
  - `vmsg(V, "strings"           ) : s        # generate string showing each KVP     `
  - `vmsg(V, "disposable"        ) : s        # disposable property                  `
  - `vmsg(V, "id"                ) : s        # ID property, assignable              `
  - `vmsg(V, "image"             ) : s        # image property                       `
  - `vmsg(V, "kind"              ) : s        # Kind property, assignable            `
  - `vmsg(V, "metatable"         ) : s        # Metatable, assignable                `
  - `vmsg(V, "strings"           ) : s        # generate a string to show each KVP   `
  - `vmsg(V, "type"              ) : s        # Type property, assignable            `

---

<a name="woraicn"></a>

## wora.icn

Procedure to produce a value that can be read globally but can be reset only by the co-expression that set it it initially.

### procedure `wora(id,del)` : x (lvalue or rvalue)
  - Set or read a globally visible read-only value,
    - which is resettable by the C that creates it.
  - `id` identifies the value; it is a key to a static table.
  - `del` signifies that the value is to be deleted, but only when specified by the creator.
    - Otherwise, this argument is ignored.


---

<a name="legacy-source-code-control"></a>

## Legacy Source Code Control Problems

If you are still using Git rather than [the best thing since CVS](https://fossil-scm.org), then you may need the following to recover from the mess that Git Submodule can create if you are not *very* careful.

### Git Submodule - some practical reminders

Reference: [http://openmetric.org/til/programming/git-pull-with-submodule/](http://openmetric.org/til/programming/git-pull-with-submodule/)

- To add a submodule to a repo, do, e.g.:<br />
  `  git submodule add git@github.com:eschen42/aceincl.git`
- For a repo with submodules, pull all submodules using<br />
	`  git submodule update --init --recursive` <br />
  for the first time. All submodules will be pulled down locally.
- To update submodules, use<br />
	`  git submodule update --recursive --remote`<br />
- To "commit" new changes in a submodule to the client project:
	1. *Make **sure** that there are no unsaved changes in the submodule* (are any files in the sandbox different from the committed code?).
		a. If so, commit and push.
	2. Next:<br />
     `  git submodule update --recursive --remote`
	3. Now it's possible to add and commit changes to the client project.
- What's changed in the submodule? (or "Has the commit for the submodule changed?" or something):<br />
	`  git diff --submodule`

### Ichabod Crane and the HEAD-less Repository

Reference: [https://www.loekvandenouweland.com/content/head-detached-from-origin-master.html](https://web.archive.org/web/20181001042322if_/https://www.loekvandenouweland.com/content/head-detached-from-origin-master.html)

If a submodule is in a "detatched HEAD" state (and it is not 1789), there are two courses of action that to take, depending on whether commits require rescuing.

If there is no need to push commits to the submodule to a remote git repository, something like the following should work, e.g., to reattach origin/main from the remote for submodule aceincl:
```
cd aceincl
git branch -la
git checkout remotes/origin/main
git status
git pull
git status
```

If there are commits to preserve, the process is more involved; see the reference, the gist of which is something like:

1. Put the commits onto a branch:<br />
   `  git branch fix-detached-HEAD $(git log | sed -n -e '/^commit/{s/commit[ ]*//; p; q}; d')`
2. Get the `main` or `master` branch, as appropriate for your repo:<br />
   `  git checkout main` (or `git checkout master`)
3. Re-establish remote tracking:<br />
   `  git fetch`
4. Review changes if necessary:<br />
   `  git diff fix-detached-HEAD`
5. Merge the changes to the remote-tracking non-HEAD-less branch:<br />
   `  git merge fix-detached-HEAD`
6. Push changes to the remote:<br />
   `  git push --set-upstream origin master`<br />
   or
   `  git push --set-upstream origin main`
