# `trustbom export`

## Synopsis

```text
trustbom export [--draft] [--from-approvals <path>] [--fail-on <severity>]
    [--format markdown|cyclonedx] [--github-rules]
```

## Purpose

Resolves the repository's default branch, verifies the approval there, and
writes the approved, citation-backed pack:

- `export/card.md`
- `export/questionnaire.yml`
- `export/manifest.json` (integrity manifest; see [`verify`](verify.md))

This is always the last **approved** graph, never the current branch's graph.
If the approval is missing, invalid, stale, or blocked by an unwaived
finding or rejection, export refuses rather than silently substituting an
older or draft pack. `--fail-on` at export is the caller's threshold, not part
of the approval contract: a lower threshold does not change what Trust
reviewed.

Do not ignore `trust/card.json` or `trust/card.md`: commit them with the
approval, because approved export reads `trust/card.json` from the default
branch.

Every `scan` rewrites the tracked `trust/card.json` and `trust/card.md` (commit
id and assessment) even when the graph is unchanged. `export --draft` and
`publish --draft` need the card's commit id to match `HEAD`, so rescan after
each commit, including the commit that merges an approval, before a draft export.

See [approved-export.md](../approved-export.md) for the full refusal and
watermarking rules and the CycloneDX mapping.

## Options

| Option | Meaning |
| --- | --- |
| `--draft` | Export the current branch graph as an internal, explicitly unapproved, watermarked draft (never for customer use). Free. No manifest is written. |
| `--from-approvals <path>` | Default-branch repository-relative approval file path (default `trust/approvals.yml`). |
| `--fail-on <severity>` | Minimum finding severity that requires a waiver (default `medium`). |
| `--format markdown\|cyclonedx` | `markdown` (the default) writes the card and questionnaire only; `cyclonedx` also writes `export/trustbom.cdx.json`, a CycloneDX 1.6 JSON BOM of the same graph. The approved form is paid and `export --draft --format cyclonedx` is free. |
| `--github-rules` | Read GitHub (through `gh`) and state the enforcement level in an "Approval controls" section of `export/card.md`. The level never changes the graph, its fingerprint or the export result. Cannot be combined with `--draft`. See [`github-rules`](github-rules.md#check-the-enforcement-level). |

```powershell
trustbom export --draft
trustbom export --format cyclonedx
```

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Parser or usage error (for example an unknown `--format`). |
| `2` | Refused or operational failure: missing, invalid or stale approval, unwaived finding or rejection, default branch not provable, shallow clone, or invalid option combination. |
| `4` | Entitlement failure (approved export only; `--draft` never checks a license). |

## License

**Paid** for approved export. `export --draft` is free.

## Example

An internal draft of the current branch (real output from a repository whose
graph is approved on `main`; `<repository>` is the absolute path):

```powershell
trustbom export --draft
```

```text
Wrote <repository>\export\card.md
Wrote <repository>\export\questionnaire.yml
Draft graph fingerprint: 42ff94542d78
Full fingerprint: 42ff94542d7806435413a1de532e05b98f5cf7c5cdc150fd58d059c41902ac1f
Status: internal draft - unapproved - not for customer use
```

Exit code `0`. The approved form, `trustbom export`, checks the license first.
Without a usable license it writes nothing, prints `license-<id>: <message>`
on standard error and exits `4`; with one it writes the three files above and
labels the card `approved`.

## Common errors

- **Exit `4`, `license-missing`, `license-invalid` and the other `license-*`
  diagnostics**: see [`license`](license.md#license-enforcement).
- **Approval refused**: no approval, a stale approval (the tip graph does not
  match `approved_graph_sha`), an unwaived finding, or a rejection. Run
  [`gate`](gate.md) for the resolution.
- **Default branch cannot be proven**: run `git remote set-head origin -a`; a
  local `main` or `master` is not evidence. A shallow clone is also refused.
- **Draft refused because the card's commit id does not match `HEAD`**: rescan
  after each commit.

## See also

- [Approved and draft export](../approved-export.md)
- [`publish`](publish.md) for a static trust page, [`verify`](verify.md) for buyers
- [`answer`](answer.md) to answer buyer questions from the approved graph
- [`request-approval`](request-approval.md)
