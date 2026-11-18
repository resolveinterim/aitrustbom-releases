# `trustbom answer`

## Synopsis

```text
trustbom answer --questions <csv> [--portfolio] [--from <dir>] [--fail-on <severity>]
trustbom answer --workbook <xlsx> --sheet <name> --id-column <header>
    --question-column <header> --answer-column <header>
    [--reuse-map <path>] [--suggest-only]
```

## Purpose

Answers buyer questions from a default-branch approved graph. It maps an
`id,text` CSV to cited answers and leaves unmatched questions for Trust to
review, or suggests reviewed answer reuse for a configured `.xlsx` workbook.
The full mapping rules (question bank, aliases, `trust/question-map.yml`,
output columns) are in [Map a buyer questionnaire](../questionnaire-mapping.md);
workbook reuse is in
[reviewed workbook reuse](../questionnaire-mapping.md#reviewed-workbook-reuse).

`answer --portfolio` combines approved per-system answers with separate source
citations into one row per buyer question. For sibling imports,
`answer --from artifacts` and `publish --from artifacts` use the same verified
approved snapshots as `pack --from artifacts`.

## Options

| Option | Meaning |
| --- | --- |
| `--questions <csv>` | Buyer CSV with `id,text` and an optional `section` column. |
| `--workbook <xlsx>` | Buyer `.xlsx` workbook in which to reuse Trust-reviewed answers. |
| `--sheet <name>` | Workbook sheet name (required with `--workbook`). |
| `--id-column <header>` | Header for the stable workbook row identifier. |
| `--question-column <header>` | Header for the buyer question text. |
| `--answer-column <header>` | Header for the answer column to fill. |
| `--reuse-map <path>` | Tracked reviewed answer mapping path (default `trust/answer-reuse.yml`). |
| `--suggest-only` | Write draft match suggestions for Trust review without filling the workbook. |
| `--portfolio` | Answer from the approved portfolio rather than a single system. |
| `--from <dir>` | Directory of verified approved sibling artifacts. |
| `--fail-on <severity>` | Minimum finding severity that requires a waiver (default `medium`). |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Every row matched: no unmatched rows. |
| `1` | Parser or usage error. |
| `2` | Unmatched rows or an operational error (the error is written to standard error). |
| `4` | Entitlement failure (`answer` is paid in every mode). |

## Licence

**Paid**, in every mode, including workbook reuse and `--suggest-only`.

## Example

```powershell
trustbom answer --questions customer.csv
```

On success it writes `export/answers.csv` (`id,question,section,answer,cites,confidence`)
and `export/unmatched.csv` (`id,text,section`) together. Rows Trust must answer
separately appear in `unmatched.csv` and make the exit code `2`. Without a
usable licence nothing is written. A build without licensing configured
printed (real output, exit `4`):

```text
license-invalid: Licensing is not configured for this build: the license service URL and lease signing keys are embedded at build time and cannot be supplied at runtime.
```

A released build without a key prints `license-missing` and instructions.

## Common errors

- **Exit `4`**: a `license-*` diagnostic; see [`license`](license.md#license-enforcement).
- **Exit `2` with unmatched rows**: answer those rows separately or add an alias
  in `trust/question-map.yml`, then rerun.
- **Approval refused**: `answer` uses the same approval checks and threshold as
  [`export`](export.md); it never reads the current branch's draft graph.
- **Unknown is not a negative claim**: an `unknown` answer means Trust has not
  declared the value.

## See also

- [Map a buyer questionnaire](../questionnaire-mapping.md)
- [`export`](export.md), [`drift`](drift.md) and [`pack`](pack.md)
- [Growth workflow guide](../growth-workflow.md) for `portfolio-import.json`
