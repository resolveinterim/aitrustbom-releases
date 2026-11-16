# Growth reusable workflow

Run the Growth workflow from a vendor repository with a tracked
`trust/portfolio.yml`. `trustbom init --growth` creates any missing portfolio
example, CODEOWNERS example, `.github/workflows/ai-trustbom-growth.yml`
caller, the reusable `.github/workflows/growth.yml` it calls, and the
per-repository Action in `.github/actions/trustbom/` (the same `action.yml`
and `install/action.yml` that `trustbom init` writes); `--dry-run` lists the same
files without writing them. Nothing refers to the AI TrustBOM source
repository. Existing files
are **never replaced**. Edit the example `apps/example` path to a real tracked,
non-overlapping system path; replace the example CODEOWNERS teams and ensure
existing CODEOWNERS protects both `trust/portfolio.yml` and
`trust/systems/*/approvals.yml`. Create reviewed, separately merged approvals
for each system; init never fabricates approved graphs. This
read-only reusable workflow does not write approval state for Trust.

The engine is the published `trustbom` npm package pinned in
`.github/actions/trustbom/install/action.yml` to the version of the CLI that ran
`init --growth` (a development build refuses to pin itself; set
`TRUSTBOM_ACTION_VERSION` to a published version to override). No Docker or
.NET SDK is needed; the runner needs Node.js, npm and Git. The caller
uses the local reusable workflow (`uses: ./.github/workflows/growth.yml`), which runs
that Action for the pull-request review and the same install action for the
default-branch `pack`, `answer`, and `publish` steps. Upgrade with
`trustbom init --upgrade` (see [upgrading generated files](cli-reference.md#upgrading-generated-files))
or by changing the
version in the install action through a reviewed pull request; the pinned version
must support portfolios and the verbs listed below. Because pull requests run
their own copy of the workflow and Action, `init --growth` adds CODEOWNERS
rules for `/.github/workflows/ai-trustbom-growth.yml`,
`/.github/workflows/growth.yml`, and `/.github/actions/trustbom/`; require
code-owner review on those paths.
The caller listens to pull requests and pushes to the detected default branch.
It grants `contents: read`, `pull-requests: write`, and `checks: write`; the
called workflow confines write scopes to the PR review job. Fork pull requests
still scan and gate, but the Action skips privileged comment/check publication.
Do not use `pull_request_target`. The workflow pins checkout and artifact
actions to commit SHAs and checks out the caller's full history; it does not
check out any engine source.

`portfolio-path` is currently restricted to `trust/portfolio.yml`: the CLI
loads that tracked canonical declaration, not an arbitrary portfolio file. The
workflow fails on any other path rather than claiming to scan a different one.
`fail-on` defaults to `medium`; `questions-csv` is optional and must name a
tracked CSV with `id,text` (and optionally `section`). If supplied, unmatched
answers cause a nonzero exit and prevent buyer-ready upload; Trust must review
them. The imported-portfolio mode requires an explicitly named
`imports-artifact` uploaded by a caller job in the **same run**, containing a
bounded `portfolio-import.json` and its referenced sibling cards. The reusable
workflow downloads that exact named artifact; it does not discover or download
cross-repository artifacts on its own. The caller must prepare the bundle
and supply the separate `sibling-read-token` reusable-workflow secret. Use a
read-only token scoped to `contents: read` on **every** sibling source repository
(for example, an appropriately scoped GitHub App installation token or
fine-grained PAT); the run's `github.token` is not substituted for it.
The approved default-branch pack/answer/publish step maps this secret to
`TRUSTBOM_GITHUB_TOKEN` only when verifying imports. The Action separately
receives `GITHUB_TOKEN` for PR publication; neither token is passed to the
scanner. A missing or insufficient sibling token fails closed. Verification
reads each source repository's current default-branch
approval and derived approved graph. The artifact's approval metadata alone is never
authority. The current verifier calls `api.github.com`; it does not support a
custom GitHub Enterprise API URL. If the sibling token cannot read the source repositories, verification
fails; do not substitute unverified artifacts. `--from` selects the imported
systems for the buyer bundle, **not** a union with the local monorepo systems;
the local portfolio's gate remains a prerequisite. Use separate calls to
publish a monorepo pack and an imported pack rather than assume a combined pack.

For imported siblings, the caller prepares a directory (for example,
`prepared-sibling-bundle/`) in its own trusted job, uploads it with the pinned
`actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02`,
and passes that artifact's exact name as `imports-artifact` to a Growth job
that `needs` the preparation job. Map the separate secret on that caller job:

```yaml
secrets:
  sibling-read-token: ${{ secrets.TRUSTBOM_SIBLING_READ_TOKEN }}
```

The directory must contain
`portfolio-import.json` and every referenced `artifact_path`, for example:

```json
{
  "organization": "Example vendor",
  "systems": [
    {
      "id": "assist",
      "repository": "example/assist",
      "source_commit": "<40-character derived approved graph commit>",
      "artifact_path": "assist/card.json",
      "graph_path": "trust/card.json",
      "approval_path": "trust/approvals.yml"
    }
  ]
}
```

This manifest identifies inputs for verification, **not** an approval record.
The verifier refuses wrong hashes, a source commit that differs from the derived baseline, moved default-branch
tips, missing files, partial collections, or inaccessible repositories. Never
use untrusted network downloads as an implicit preparation step.

The approved default-branch step runs paid commands (`pack`, `answer`,
`publish`), so the reusable workflow takes an optional `license-key` secret and
passes it to that step only, as `TRUSTBOM_LICENSE_KEY`. `trustbom init --growth`
maps it from a repository secret of that name; see
[Create the `license-key` secret](licensing.md#create-the-license-key-secret).
Pull requests (including forks) run only the free review job and never receive
the key; an empty secret on a push fails with the CLI's `license-missing`
diagnostic.

On a PR, the portfolio Action scans and gates each declared system, publishes
one marked portfolio comment and bounded check when permitted, and emits the
`system-ids` JSON array, per-system `graph-shas`/`sarif-paths` JSON objects,
and `gate-status`. The workflow uploads individual
`trustbom-DRAFT-<system-id>-card` artifacts plus a clearly labeled internal
staging artifact, even if approval fails. It **enforces the gate after those
uploads**: an Action failure, missing output, blocked policy, or failed draft
upload fails the PR job. A break-glass soft pass remains a draft and cannot
authorize approved publication.

Only a push to the repository's default branch can run the buyer job. It
requires the Action's `passed` gate status (not `breakglass`), then runs
`trustbom pack --fail-on <threshold>`, optional
`trustbom answer --questions <csv> --portfolio --fail-on <threshold>`, and
`trustbom publish --fail-on <threshold>` using the same pinned Action image. With an
explicit import artifact it passes `--from .trustbom-imports` to each operation
instead of `--portfolio` to `answer`. Answering writes `export/answers.csv`
into the checkout before `publish` renders the site. Each operation must independently
validate the approved graph; a partial or failed run uploads no buyer-ready
artifact. A successful run uploads `trustbom-APPROVED-buyer-bundle` containing
the approved export, HTML/PDF site, and `approved-html-pack.zip` (a portable
ZIP of the site, approved system cards and optional questionnaire CSV).
It also uploads `trustbom-APPROVED-<system-id>-card` artifacts, each containing
the verified historical `card.json` used in the buyer pack, rather than a
newly rescanned or PR-draft graph. It never uploads PR draft cards as approved
bundles.

The workflow expects the CLI interfaces `pack --from <directory>`,
`answer --questions <csv> --portfolio [--fail-on <threshold>]`,
`answer --questions <csv> --from <directory> [--fail-on <threshold>]`, and
`publish --from <directory> --fail-on <threshold>` for imported siblings.
The pinned `trustbom` release must contain these interfaces and the GitHub
provenance verifier; older CLI releases cannot run the Growth caller.

The Action used here has the same inputs and outputs as the single-system
Action; Growth passes `portfolio: "true"`, its `fail-on` input (same `medium`
default), and `comment`, and reads the `system-ids` and `gate-status` outputs.
A test enforces this overlap.
