# Portfolio reference

A **portfolio** is several AI systems under one Trust. This page lists the
contract of the generated portfolio workflow. For setup, start with
[Portfolio in one repository](portfolio-one-repository.md). Importing from
other repositories ([Portfolio across repositories](portfolio-across-repositories.md))
is not recommended; a monorepo pack and an imported pack are separate and
cannot be combined.

## Generated files

| File | Role |
| --- | --- |
| `trust/portfolio.yml` | Declares the organization and its systems. |
| `.github/workflows/ai-trustbom-portfolio.yml` | Caller: triggers, permissions and secrets. |
| `.github/workflows/portfolio.yml` | Reusable workflow the caller uses (`uses: ./.github/workflows/portfolio.yml`). |
| `.github/actions/trustbom/` | Per-repository Action and install action, the same files `trustbom init` writes. |

## Inputs and secrets

| Name | Notes |
| --- | --- |
| `portfolio-path` | Currently restricted to `trust/portfolio.yml`; the workflow fails on any other path rather than scanning a different one. |
| `fail-on` | Defaults to `medium`. |
| `questions-csv` | Optional tracked CSV with `id,text` (and optionally `section`). |
| `imports-artifact` | Optional; exact name of a same-run artifact for [imported siblings](portfolio-across-repositories.md). |
| `license-key` (secret, optional) | Passed only to the default-branch step that runs paid commands, as `TRUSTBOM_LICENSE_KEY`. Pull requests never receive it. |
| `sibling-read-token` (secret, optional) | Read-only token for imported siblings; see [Token](portfolio-across-repositories.md#token). |

## CLI interfaces

The workflow expects `pack --from <directory>`,
`answer --questions <csv> --portfolio [--fail-on <threshold>]`,
`answer --questions <csv> --from <directory> [--fail-on <threshold>]` and
`publish --from <directory> --fail-on <threshold>` for imported siblings. The
pinned `trustbom` release must contain these interfaces and the GitHub
provenance verifier; older CLI releases cannot run the portfolio caller. With an
explicit import artifact the workflow passes `--from .trustbom-imports` to each
operation instead of `--portfolio` to `answer`. Answering writes
`export/answers.csv` into the checkout before `publish` renders the site.

## Outputs and artifacts

The portfolio Action emits the `system-ids` JSON array, per-system
`graph-shas` and `sarif-paths` JSON objects, and `gate-status`. On a pull
request the workflow uploads `trustbom-DRAFT-<system-id>-card` artifacts plus a
clearly labeled internal staging artifact, even if approval fails. A successful
default-branch run uploads `trustbom-APPROVED-buyer-bundle` (the approved
export, HTML/PDF site and `approved-html-pack.zip`) and
`trustbom-APPROVED-<system-id>-card` artifacts holding the verified historical
`card.json` used in the pack.

## Action contract

The portfolio workflow uses the same inputs and outputs as the single-system
Action: it passes `portfolio: "true"`, its `fail-on` input (same `medium`
default) and `comment`, and reads the `system-ids` and `gate-status` outputs. A
test enforces this overlap. The workflow pins checkout and artifact actions to
commit SHAs, checks out the caller's full history, and checks out no engine
source.

## Licensing

Scan and gate are free for any number of systems. `pack`, `publish` and
`answer` use the organization license; see
[licensing](licensing.md#create-the-license-key-secret).
