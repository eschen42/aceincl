# Icon "inclusion library"

Incorporate these files into Icon programs using the `$include` preprocessor macro.
This should be easier than translating to ucode and linking.

Working examples, named `test_*.icn`, are in the `tests` directory.
`runt.icn` will run them all and compare the results to the `*.std` captures of their expected output.

## fieldedDataFile.icn

Procedures to produce logical lines or fields from formatted data files.

#### record `FieldedData(lines, fields)`
  - Record holding two co-expression factories
    - `lines` === tabularLines | iniLines
    - `fields` === tabularFields | iniFields
#### procedure `FieldedDataFactory(format, filePath)`
  - Produce a `FieldedData` record for `filePath` corresponding to format.
  - `format == ("tabular" | "ini")`
#### procedure `tabularLines(f)`
  - Factory for a co-expression producing logical lines of a tabular file `f`.
#### procedure `tabularFields(line, sep)`
  - Factory for a co-expression producing fields from a logical line of a tabular file:
    - `line` is a logical line produced by `tabularLines`.
    - `sep` is the field separator; if omitted or &null, TAB is used.
#### procedure `iniLines(f)`
  - Factory for a co-expression producing logical lines of an INI file `f`.
#### procedure `iniFields(line)`
  - Factory for a co-expression producing fields from a logical line of an INI file
    - `line` is a logical line produced by `iniLines`.
#### procedure `getIni(ini)`
  - Parse an INI file at path `ini` into a table of tables
#### procedure `alterExtension(fn, old_ex, new_ex)`
  - Change extension of file name
    - `fn` is the file name.
    - `old_ex` is the extension portion of `fn`.
    - `new_ex` is the new extension to replace `old_ex`

## fieldedDataFile.icn

Procedures to manipulate files, directores, and their paths.

#### procedure alterExtension(fn, old_ex, new_ex) : s
  - Produce fn, substituting new_ex for old_ex

#### procedure directory_seq(name)
  - Produce name(s) that name a directory

## iimage.icn

Procedures to transform data structures into includable Icon declarations and statements

#### procedure `iimage(x)`
  - Produce Icon code to reproduce value `x`, if possible
#### procedure `idump(f, x[])`
  - Write Icon code to reproduce values in list `x`

## selectRecordFromListByField.icn

Procedure to produce records from a list of records, matching specified criteria.

#### procedure `selectRecordFromListByField(Lfrom, sField, Ctest)`
  - Select matching records (or tables) `X`
    - from list `Lfrom`
    - where `X[sField] @ Ctest` succeeds

## wora.icn

Procedure to produce a value that can be read globally but can be reset only by the co-expression that set it it initially.

#### procedure `wora(id)`
  - Set a globally visibe read-only value
    - which is resettable by the C that creates it.

## LiComboP.icn

Procedures to suspend lists combining sequences.

#### procedure `LiP(A)`
  - Suspend lists combining infinite sequences. LiP:
    - evaluates in a "breadth first" manner to ensure that all values of finite
      sequences will eventually be produced even when some sequences are infinite.
    - uses memoization to avert the need to evaluate each sequence more than once.
    - uses wora(LiP) to determine whether to use LiFiniteP
      (the default) or nAltP to combine memoized results.
    - requires that `wora.icn` be previously included, for wora(id)
#### procedure `LiFiniteP(LofC)`
  - Recursively suspend lists combining finite seqs;
    - does not enforce "breadth first" evaluation.
#### procedure `nAltP(LofC)`
  - Recurrently suspend lists combining finite seqs;
    - does not enforce "breadth first" evaluation.
