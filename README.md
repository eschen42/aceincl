# Icon "inclusion library"

Incorporate these files into Icon programs using the `$include` preprocessor macro.
This should be easier than translating to ucode and linking.

## fieldedDataFile.icn

Procedures to produce logical lines or fields from formatted data files.

- record `FieldedData(lines, fields)`
  - record holding two co-expression factories
- procedure `FieldedDataFactory(format, filePath)`
  - produce a `FieldedData` record corresponding to format
  - `format == ("tabular" | "ini")` 
- procedure `tabularLines(f)`
  - factory for a co-expression producing logical lines of a tabular file
- procedure `tabularFields(line, sep)`
  - factory for a co-expression producing fields from a logical line of a tabular file
- procedure `iniLines(f)`
  - factory for a co-expression producing logical lines of an INI file
- procedure `iniFields(line)`
  - factory for a co-expression producing fields from a logical line of an INI file
- procedure `getIni(ini)`
  - parse an INI file into a table of tables
- procedure `alterExtension(fn, old_ex, new_ex)`
  - change extension of file name

# iimage.icn

Procedures to transform data structures into includable Icon declarations and statements

- procedure `iimage(x)`
  - produce Icon code to reproduce value `x`, if possible
- procedure `idump(f, x[])` 
  - write Icon code to reproduce values in list `x`

# selectRecordFromListByField.icn

- procedure `selectRecordFromListByField(Lfrom, sField, Ctest)`
  - select matching records (or tables) `X`
    - from list `Lfrom`
    - where `X[sField] @ Ctest` succeeds
