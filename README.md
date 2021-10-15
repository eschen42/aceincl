# Icon "inclusion library"

Incorporate these files into Icon programs using the `$include` preprocessor macro:

- This may be readily achieved by including this directory
  in your `LPATH` environment variable.
- This may be require less preparation and updating than would translating
  to "ucode" and using the `link` directive.

## Testing program `runt.icn` and working examples

Working examples, named `test_*.icn`, are in the `tests` directory:

- `test_*.std` captures the corresponding test's expected output.
- `icon runt.icn tests` will run the tests and compare the results to
   their expected output.
- usage: `icon runt.icn [--continue] [--verbose] [<zero or more dirs>]`
  - By default, tests are located and run in the current working directory.
    - Otherwise, tests are located and run in the specified directory or directories.
  - Tests are not run unless an `.icn` file and a `.std` file share exactly
    the same name.
  - Use the `--continue` option to run all tests regardless of whether any fail.
  - Use the `--verbose` option to show both the expected output
    from the `.std` file and the actual output produced by the test program.

## fieldedDataFile.icn

Procedures to produce logical lines or fields from formatted data files.

#### record `FieldedData(lines, fields)`

Produce record holding two co-expression factories:

- `lines` === tabularLines | iniLines
- `fields` === tabularFields | iniFields

#### procedure `FieldedDataFactory(format, filePath)` : FieldedData

Produce a `FieldedData` record for `filePath` corresponding to format.

- `format == ("tabular" | "ini")`

#### procedure `tabularLines(f)` : C

Factory for a co-expression producing logical lines of a tabular file `f`.

#### procedure `tabularFields(line, sep)` : C

Factory for a co-expression producing fields from a logical line of a tabular file:
- `line` is a logical line produced by `tabularLines`.
- `sep` is the field separator; if omitted or &null, TAB is used.

#### procedure `getTabular(typeName, tsvPath, colL, sep, dflt)` : L

Produce L of RecTable from a tabular file

- `typeName`: the first result to be produced by RecTableType(T), s
- `tsvPath` : the path to the tabular data file, s
- `colL`    : (optional) columns to select, L of i
- `sep`     : (optional) separators, c
- `dflt`    : (optional) default value for RecTable fields, x

#### procedure `iniLines(f)` : C

Factory for a co-expression producing logical lines of an INI file `f`.

#### procedure `iniFields(line)` : C

Factory for a co-expression producing fields from a logical line of an INI file

- `line` is a logical line produced by `iniLines`.

#### procedure `getIni(ini)` : T (two dimensional)

Parse an INI file at path `ini` into a table of tables

## fileDirIo.icn

Procedures to manipulate files, directores, and their paths.

#### procedure `alterExtension(fn, old_ex, new_ex)` : s1, ...

Produce modified `fn`, substituting `new_ex` for `old_ex`

- If `new_ex` is "", the trailing period will be removed.

#### procedure `directory_seq(name)` : s1, ...

Produce name(s) that name a directory

#### procedure `prog_path_parts()` : s1, s2

Suspend location then name of program file.

#### procedure `path_atoms(path)` : s1, ...

Suspend root, subdirectories, filename for a directory path

#### procedure `path_constructP{expr}` : s1, ...

Construct paths from sequences

#### procedure `cmd_separator()` : s

Return platform-specific command separator

#### procedure `path_separator()` : s

Return platform-specific path separator

#### procedure `pwd()` : s

Return platform-specific path to the current directory

## iimage.icn

Procedures to transform data structures into includable Icon declarations and statements

#### procedure `iimage(x)` : s
  - Produce Icon code to reproduce value `x`, if possible
#### procedure `idump(f, x[])` : (writes to `\f | &errout`)
  - Write Icon code to reproduce values in list `x` to `f` if it is a file;
    otherwise to `&errout` and `f` is discarded.

## selectRecordFromListByField.icn

Procedure to produce records from a list of records (or a list of tables), matching specified criteria.

#### procedure `selectRecordFromListByField(Lfrom, sField, Ctest)` : R1, ...
  - Produce matching records (or tables) `X`
    - from list `Lfrom` (`type(Lfrom[i]) == "record" | "table"`)
    - where `X[sField] @ Ctest` succeeds

## wora.icn

Procedure to produce a value that can be read globally but can be reset only by the co-expression that set it it initially.

#### procedure `wora(id,del)` : x (lvalue or rvalue)
  - Set or read a globally visible read-only value,
    - which is resettable by the C that creates it.
  - `id` identifies the value; it is a key to a static table.
  - `del` signifies that the value is to be deleted, but only when specified by the creator.
    - Otherwise, this argument is ignored.

## LiComboP.icn

Procedures to suspend lists combining sequences.

#### procedure `LiP(A)` : L1, ...
  - Suspend lists combining infinite sequences. LiP:
    - evaluates in a "breadth first" manner to ensure that all values of finite
      sequences will eventually be produced even when some sequences are infinite.
    - uses memoization to avert the need to evaluate each sequence more than once.
    - uses wora(LiP) to determine whether to use LiFiniteP
      (the default) or nAltP to combine memoized results.
    - requires that `wora.icn` be previously included, for wora(id)
#### procedure `LiFiniteP(LofC)` : L1, ...
  - Recursively suspend lists combining finite seqs;
    - does not enforce "breadth first" evaluation.
#### procedure `nAltP(LofC)` : L1, ...
  - Recurrently suspend lists combining finite seqs;
    - does not enforce "breadth first" evaluation.

## RecTable.icn

Procedures to produce/manipulate record-like tables.

### procedure `RecTableConstructorC(rec_name_s, rec_fields_L, rec_default_x)` : C

Produce a C that, when receiving a transmitted list of values (of the
same length as `rec_fields_L`), produces a RecTable instance:

- `rec_name_s`    the "type" of the RecTable
- `rec_fields_L`  a list of the field names
- `rec_col_iL`    an optional list of column numbers to choose,
                  defaults to all
- `rec_default_x` default value for table members

### procedure `RecTable(rec_name_s, rec_fields_L, rec_data_L, rec_default_x)` : T

Produce a table with record-like aspects:

- `rec_name_s`:    the "type" of the RecTable
- `rec_fields_L`:  a list of the field names
- `rec_data_L`:    an optional list of values to assign to the fields
- `rec_col_iL`:    an optional list of column numbers to choose
                 defaults to all
- `rec_default_x`: default value for table members

### procedure `RecTableType(x)` : s1, S2, s3, ...

For RecTable, produce:

- name
- set of all fields
- each field

For non-RecTable, return type(x).

### procedure `RecTableFields(x)` : s1, ...

Produce RecTable's fields, or (likely) default value for non-RecTable:

- The default value will be produced for non-RecTable except when the
  key that is the table itself has been assigned another value.

### procedure `RecTableColTypeCheck(x, type_name, col_name, preamble)` : x

Return x, except abort when x is not instance of type_name:

- `x`        : value whose type is to be checked
- `type_name`: expected string for RecTableType(x)
- `col_name` : name of identifier-under-test
- `preamble` : initial string for error message; defaults value of name
             RecTablePreamble.


<hr />

## Git Submodule - some practical reminders

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
