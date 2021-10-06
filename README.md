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
  - Record holding two co-expression factories
    - `lines` === tabularLines | iniLines
    - `fields` === tabularFields | iniFields
#### procedure `FieldedDataFactory(format, filePath)` : FieldedData
  - Produce a `FieldedData` record for `filePath` corresponding to format.
  - `format == ("tabular" | "ini")`
#### procedure `tabularLines(f)` : C
  - Factory for a co-expression producing logical lines of a tabular file `f`.
#### procedure `tabularFields(line, sep)` : C
  - Factory for a co-expression producing fields from a logical line of a tabular file:
    - `line` is a logical line produced by `tabularLines`.
    - `sep` is the field separator; if omitted or &null, TAB is used.
#### procedure `iniLines(f)` : C
  - Factory for a co-expression producing logical lines of an INI file `f`.
#### procedure `iniFields(line)` : C
  - Factory for a co-expression producing fields from a logical line of an INI file
    - `line` is a logical line produced by `iniLines`.
#### procedure `getIni(ini)` : T (two dimensional)
  - Parse an INI file at path `ini` into a table of tables

## fileDirIo.icn

Procedures to manipulate files, directores, and their paths.

#### procedure `alterExtension(fn, old_ex, new_ex)` : s1, ...
  - Produce modified `fn`, substituting `new_ex` for `old_ex`
    - If `new_ex` is "", the trailing period will be removed.

#### procedure `directory_seq(name)` : s1, ...
  - Produce name(s) that name a directory

#### procedure `prog_path_parts()` : s1, s2
  - suspend location then name of program file.

#### procedure `path_atoms(path)` : s1, ...
  - suspend root, subdirectories, filename for a directory path

#### procedure `path_constructP{expr}` : s1, ...
  - construct paths from sequences

#### procedure `cmd_separator()` : s
  - return platform-specific command separator

#### procedure `path_separator()` : s
  - return platform-specific path separator

#### procedure `pwd()` : s
  - return platform-specific path to the current directory

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
