# Welcome to the CLI reference

This is the reference for `trustbom`, the AI TrustBOM command-line tool. Here
you will find every command with its options, exit codes,
examples and common errors. Use it when you know what you want to do and need
the exact command, or want to check what a command writes and when it fails.

New to AI TrustBOM? Start with the [tutorial](../tutorial/README.md), which
walks through a complete first journey, or the
[workflow overview](../workflow.md) for the journey at a glance. You will need
the CLI first: see [install the CLI](../installing.md).

Related guides: [YAML input reference](../yaml-input-reference.md) for the
editable files, [the GitHub Action](../github-action.md) for automating scans
and gates on pull requests, and [approved export](../approved-export.md).

## Commands

| Command | Purpose |
|---|---|
| [`trustbom init`](init.md) | Write preserved-file scaffolding; use `--interactive` for previewed single-system onboarding and optional read-only setup checks |
| [`trustbom scan`](scan.md) | Detect AI usage and write `trust/card.json`, `card.md`, `questionnaire.yml`, `findings.sarif` |
| [`trustbom gate`](gate.md) | Compare the current graph with the approved baseline and explain what blocks the PR |
| [`trustbom request-approval [--check-controls]`](request-approval.md) | Author an approval-file proposal for independent CODEOWNER review; optionally inspect read-only GitHub controls |
| [`trustbom export`](export.md) | Write the approved `export/card.md` and `export/questionnaire.yml` (or `--draft` for an internal snapshot); `--format cyclonedx` also writes `export/trustbom.cdx.json` |
| [`trustbom publish`](publish.md) | Render a self-contained static trust page under `export/site/` (index, per-system pages, `verify.html`, `index.pdf`, downloads and approved `manifest.json`) from approved graphs, or `--draft` for a watermarked internal pack |
| [`trustbom license`](license.md) | Print the license terms for the installed version; works offline |
| [`trustbom license activate [--github-owner <owner>]`](license.md#options) | Exchange a license key (prompted, hidden, or piped on stdin) for a signed lease bound to the resolved GitHub owner |
| [`trustbom license refresh [--github-owner <owner>]`](license.md#options) | Renew the license on this computer now, regardless of remaining lease time |
| [`trustbom license request-trial --email <email> [--github-owner <owner>]`](license.md#options) | Request a free 30-day trial key by email; the service emails the key (never returned) and the 30-day trial starts when the key is issued |
| [`trustbom license status [--json]`](license.md#options) | Report the cached license state offline (lease end, trial or subscription end, renewal due), without contacting the license service |
| [`trustbom readiness`](readiness.md) | Write an advisory `export/readiness/` checklist for evidence coverage, approval validity, unresolved findings, and freshness; use `--framework <id>` for free offline framework-alignment overlays (`nist-ai-rmf`, `iso-42001`, `eu-ai-act`) |
| [`trustbom init --upgrade [--dry-run]`](init.md#upgrading-generated-files) | Rewrite only unedited marker-carrying generated files to the running CLI's pinned version. Never combined with `--growth` or `--interactive` |
| [`trustbom init --growth`](init.md#options) | Create a portfolio template, CODEOWNERS guidance, a local reusable Growth workflow and caller, and the pinned per-repository Action without overwriting existing files |
| [`trustbom github-rules apply [--dry-run] [--yes]`](github-rules.md#apply-the-ruleset) | Create or update the `trustbom-approval` ruleset (written by `init` to `.github/rulesets/trustbom.json`) through your `gh` login; needs repository admin rights, otherwise nothing changes and the admin steps print |
| [`trustbom github-rules check [--github] [--require-enforced]`](github-rules.md#check-the-enforcement-level) | Report the enforcement level (`enforced`, `audited`, `violation`, `unverified`) with per-rule status; offline unless `--github`; exits `3` on `violation`, or below `enforced` with `--require-enforced` |
| [`trustbom pack [--from artifacts]`](pack.md) | Render independently approved monorepo or GitHub-verified sibling graphs into one portfolio pack |
| [`trustbom verify [<path>] [--repo <path>]`](verify.md) | Check a received pack against its `manifest.json`; with `--repo`, also confirm approved fingerprints against the named approvals |
| [`trustbom answer --portfolio --questions customer.csv`](answer.md) | Combine approved per-system answers with separate source citations into one row per buyer question |
| [`trustbom changelog`](changelog.md) | Print a Git-derived Markdown comparison of approved graph baselines |
| [`trustbom drift [--register <path>]`](drift.md) | Report which registered buyer answers may be stale against current graph evidence |

### Exit codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | Parser or usage error: unknown command, option or argument value (for example an unknown `--format` or `--migrate`, or a missing required option such as `license request-trial --email`). The usage text is printed and nothing is written |
| `2` | Operational or argument failure detected by the command (invalid `--fail-on`, incompatible option combinations, Git, input or file failures, unmatched answer rows, cancelled interactive setup) |
| `3` | Gate policy failure |
| `4` | Entitlement failure: a paid command's license check failed, or `license status` found no usable lease |

Treat any nonzero code as a failure in automation; parser errors exit `1`
rather than `2` and fail closed.
