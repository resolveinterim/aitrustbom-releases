# `trustbom publish`

## Synopsis

```text
trustbom publish [--draft] [--fail-on <severity>] [--from <dir>]
```

## Purpose

Renders the same approved graph as a self-contained, deployable static site
under `export/site/`: `index.html`, a `verify.html` page with buyer
verification instructions, `index.pdf`, an approved-only `manifest.json`, and a
per-system `downloads/<system>/` folder holding `card.json`, the CycloneDX
ML-BOM (`trustbom.cdx.json`) and `questionnaire.yml`.
A single-system publication keeps the full detail on `index.html`; a portfolio
uses `index.html` as an index linking to one `systems/<system>.html` page each.
Every page and download is a projection of its cited `card.json`; the pages use
inline styles only, with no CDN, scripts or network access. Zip or serve
`export/site/` to deliver the pack, and a buyer confirms integrity offline with
[`trustbom verify export/site`](verify.md). `trustbom publish --draft` writes
the same tree from the current branch, watermarked and without a
`manifest.json`. See [approved-export.md](../approved-export.md) for all rules
and [trust-page.md](../trust-page.md) to host the directory with GitHub Pages.
A `answers.csv` included in the site remains user-provided content that needs
separate review.

For sibling imports, `publish --from artifacts` uses the same verified approved
snapshots as `pack --from artifacts`. Set a read-only `TRUSTBOM_GITHUB_TOKEN`
with access to each source repository and prepare `portfolio-import.json` as
described in the [Growth workflow guide](../growth-workflow.md). Unverified
artifact content is never an approved buyer claim.

## Options

| Option | Meaning |
| --- | --- |
| `--draft` | Publish an explicitly watermarked, current-branch internal draft. |
| `--fail-on <severity>` | Minimum finding severity that requires a waiver in approved mode (default `medium`). |
| `--from <dir>` | Directory of verified approved sibling artifacts. |

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success. |
| `1` | Parser or usage error. |
| `2` | Refused or operational failure (approval missing, invalid or stale; unwaived finding; default branch not provable; invalid option combination). |
| `4` | Entitlement failure. |

## Licence

**Paid**, including `--draft` and `--from`.

## Example

```powershell
trustbom publish
```

With a usable licence and an approved graph on the default branch this writes
`export/site/` as described above. Without one nothing is written; a build with
licensing not configured printed (real output, exit `4`):

```text
license-invalid: Licensing is not configured for this build: the license service URL and lease signing keys are embedded at build time and cannot be supplied at runtime.
```

A released build without a key prints `license-missing` and instructions.

## Common errors

- **Exit `4`**: a `license-*` diagnostic; see [`license`](license.md#license-enforcement).
- **Approval refused**: same checks as [`export`](export.md); `--draft` is the
  only way to render an unapproved graph and is watermarked.
- **`--draft` refused**: the card's commit id must match `HEAD`; rescan after
  each commit.

## See also

- [Publish the trust page](../trust-page.md) and [Approved and draft export](../approved-export.md)
- [`verify`](verify.md), [`export`](export.md), [`pack`](pack.md)
