# csv-reader-speed-01-run-20261121

A fast drop-in replacement for csv-reader 1.0.0, for the make.fun
competition "Read broken CSV files faster, 21 November 2026".

## What it is

`solve.js` exports `run(input)`, returning exactly what csv-reader 1.0.0
returns for every input `{text, header}`: `{columns, rows, errors}` under
the reference's broken-row policy (BOM dropped, LF/CRLF record ends, lone
CR outside quotes dropped as `stray_cr`, quoted fields with `""` escapes
and CR/CRLF inside read as LF, junk after a closing quote appended as
`quote_junk`, a never-closing quote read as a literal with the rest re-read
as unquoted as `unterminated_quote`, short rows padded and long rows cut to
the first record's width with `short_row` / `long_row` errors, in order met).

`main.ts` is the harness driver (unchanged from the kit's house baseline):
it reads every case from `/input/cases.json`, answers each with `run`, and
writes `/output/answers.json`.

## Why it is faster

The reference reads one character at a time and builds every field a
character at a time. This entry scans in bulk instead:

- unquoted fields are cut with a single regex scan to the next delimiter
  (comma, CR, LF) and sliced out whole;
- quoted fields are cut with `indexOf` to the closing quote, with doubled
  quotes folded by segment join;
- newline counting and CR-to-LF normalisation inside quotes only run when
  the quoted content actually holds a CR or LF;
- no per-field result objects are allocated on the hot path.

No dependencies, no clock, no randomness, no locale; nothing is kept from
one call to the next.

## Check

Run the kit's practice set through the driver and compare with the
reference answers in `practice/reference/expected.json`; the entry must
match every case before submitting.
