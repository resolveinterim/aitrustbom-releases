# AI TrustBOM GitHub Action

**A customer asked what AI you use. Answer with evidence a named person
approved.** AI TrustBOM keeps an approved record of the AI in your product,
and stops new AI from shipping until Trust signs it off. This Action is
where developers meet it: it runs on every pull request, and its comment is
the main interface.

1. **Open a pull request as usual.** If it adds or changes AI, the check
   fails and the comment lists what changed and what is missing.
2. **Fill the gaps and propose.** Follow the comment's steps, run
   `trustbom propose`, and push. When the comment asks for `propose`, it
   gives a copy-paste install line pinned to the version the Action ran,
   then the exact `propose` command, so nobody has to look up flags.
3. **Trust signs off.** Trust reviews the pull request and merges it. The
   record is now approved, and you can hand it to customers.

Running `trustbom scan` locally first is optional. The scanner reads
tracked files only; see
[what it can and cannot see](workflow.md#what-it-can-and-cannot-see).

## How it works

The Action is a composite action: it installs the exact pinned `trustbom` npm package into the runner temp directory (`--ignore-scripts`) and runs the same scan and gate implementations as the local `trustbom` CLI. It needs Node.js, npm and Git, which GitHub-hosted runners provide. It writes `trust/card.json`, `trust/card.md`, `trust/questionnaire.yml`, and `trust/findings.sarif`, then evaluates `trust/approvals.yml`. The gate adds a plain-language assessment and required actions to `card.md` without changing `card.json` or its hash. On pull requests it creates or updates one marked comment and publishes an `AI TrustBOM` check with up to 50 finding annotations. The comment's **What changed** section compares the graph with the approved default-branch graph, citing `path:line` evidence for each added, removed or changed row, and the rendered card follows in a collapsed block. A card that would push the comment past GitHub's 65,536-character limit is cut at a line and points to the full file in the run's `ai-trustbom` artifact.

For several systems, use the [Growth reusable workflow](./growth-workflow.md); `trustbom init --growth` writes the same per-repository Action described below.
The Action's `portfolio: "true"` input scans and gates the tracked
`trust/portfolio.yml`; it emits `system-ids`, `graph-shas`, and `sarif-paths`
as JSON plus an aggregate `gate-status`, and posts one marked comment and
bounded check showing each system's status and semantic delta. The existing
single-system inputs and outputs remain unchanged.

## Workflow

`trustbom init` writes the Action into your repository at `.github/actions/trustbom/` (an `action.yml` plus a nested `install/action.yml` that installs an exact pinned `trustbom` npm version; the install action holds the single version pin) and a workflow that uses it. No access to the AI TrustBOM source repository is needed. Because the Action runs from the pull request's own copy, `init` adds CODEOWNERS rules for `/.github/actions/trustbom/` and `/.github/workflows/ai-trustbom.yml`; require code-owner review on those paths. A development build of `trustbom` refuses to pin itself; `TRUSTBOM_ACTION_VERSION` can override it with an exact available package version.

To use an internal npm registry
or mirror, make the exact pinned `trustbom` package **and its matching RID
platform package** available there. Set `npm_config_registry` to that
registry's URL in the calling workflow/job environment (the Action's nested
install step inherits it); the root Action uses the same override. Supply any
required registry credentials through GitHub secrets or runner configuration,
never in the repository, workflow file, generated Action or logs. This
registry override changes only where npm installs from, not the version pin.
Do not give untrusted fork code access to registry credentials.

Every file `init` owns (`ai-trustbom.yml`, `ai-trustbom-growth.yml`, `growth.yml`, and the Action's `action.yml` and `install/action.yml`) starts with a marker such as `# trustbom-generated: version=1.2.3 sha256=<hash>` recording the generating `trustbom` version and a hash of the body. To move to a newer pinned version, first make the new exact main and RID packages available in the chosen registry. Install the newer `trustbom` CLI and run `trustbom init --upgrade --dry-run`, then `trustbom init --upgrade`: it rewrites only marked files whose body still matches the hash, never downgrades, and never touches unmarked or edited files, `trust/*.yml`, approvals or CODEOWNERS. Review the diff and merge it through a pull request; see the [`init` reference](./cli/init.md#upgrading-generated-files).

The generated workflow is equivalent to the following. Every `with:` value is
the Action's default, so a minimal workflow is the checkout step plus
`uses: ./.github/actions/trustbom`; keep the inputs only to change them. No
administrator rights are needed to add it. Review rules are separate: see
[Enforcement levels](enforcement-levels.md).

```yaml
name: AI TrustBOM

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  trustbom:
    name: AI TrustBOM
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository history
        uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          fetch-depth: 0

      - name: Scan and gate
        uses: ./.github/actions/trustbom
        env:
          GITHUB_TOKEN: ${{ github.token }}
        with:
          fail-on: medium
          baseline: trust/approvals.yml
          comment: true
```

`fetch-depth: 0` is required because the gate derives the approved graph baseline from default-branch first-parent history. A shallow checkout degrades the gate (no baseline diff) and makes export, pack and verify refuse.

## Recommended repository controls

The generated workflow runs on pull requests only; nothing runs after a merge.
Two repository settings therefore decide how strong the gate is:

- **Require the `Scan and gate` status check** (the job, not the annotated
  `AI TrustBOM` check). Without it the gate is advisory: a failing job does not
  block the merge.
- **Require branches to be up to date before merging** (strict status checks).
  Without it the merge result is never scanned or gated, so a graph change that
  landed on the default branch after the pull request's last run can merge
  unvalidated.

Both are recommended controls, not enforced by `init`. The seven review rules
are applied by `trustbom github-rules apply` and graded by `trustbom
github-rules check` as an [enforcement level](enforcement-levels.md).
`trustbom github-rules check --github` also reports whether `Scan and gate`
is a required status check (when the committed ruleset names one), as
`verified`, `missing` or `unverified` (when the token cannot read branch
rules). Do not add a push or post-merge job to compensate.

## Stale `card.json`

When `trust/card.json` is tracked, the Action compares its committed
`graph_sha` with the graph scanned from the pull request. A difference fails
the job (`gate-status` `failed`, exit `3`) with the blocker
`card-stale`, so a forgotten card is caught before the merge instead of at
export. The fix is: run `trustbom scan` (or `trustbom propose` when signing
off) and commit `trust/card.json` to the pull request. A card that is not
tracked never fails; in portfolio mode the same check applies to each tracked
`trust/systems/<id>/card.json`. The check needs the failing job to block the
merge, so require the `Scan and gate` status check (above).

## Inputs and outputs

| Input | Default | Meaning |
|---|---|---|
| `fail-on` | `medium` | Minimum unwaived finding severity that blocks |
| `path` | empty | Optional repository-relative scan glob |
| `baseline` | `trust/approvals.yml` | Approval YAML path |
| `comment` | `true` | Create or update the marked PR comment |
| `portfolio` | `false` | Scan and gate the systems in `trust/portfolio.yml` (used by Growth) |

The Action outputs `graph-sha`, `sarif`, and `gate-status`, plus `system-ids`, `graph-shas`, and `sarif-paths` in portfolio mode. Exit `0` means approved or an explicitly draft break-glass soft pass, `3` means a policy failure, and `2` means the Action could not evaluate policy.

## Permissions and fork pull requests

`contents: read` lets the scanner inspect the checkout and historical graph. `pull-requests: write` is used only for the marked PR comment. `checks: write` is used only for the `AI TrustBOM` check and annotations. The token is passed only to the publication adapter, never to repository detectors or renderers.

The workflow explicitly maps `${{ github.token }}` to `GITHUB_TOKEN`; it is not an Action input and is never written to arguments, output, artifacts, comments, or diagnostics.

GitHub gives pull requests from forks a read-only token. The Action still scans and gates that untrusted checkout without running repository scripts, skips comment/check publication, emits a visible warning, and preserves the gate exit status. Do not use `pull_request_target` to run fork content with write credentials.

## Generated consumer workflow

`trustbom init` generates one consumer workflow with a single read-only
`pull_request` review job. It pins the checkout Action to a tested immutable
commit and uses full history. Existing workflow files are preserved rather than
replaced. Because `init` keeps `card.md`, `questionnaire.yml` and
`findings.sarif` out of Git, the job uploads them as one `ai-trustbom`
artifact (kept for 7 days, uploaded even when the gate fails) and writes the
card to the job summary. Nothing is uploaded to GitHub code scanning; the
check's annotations show the findings. The workflow passes `trust/approvals.yml` to the gate unchanged: the
gate itself maps only the marked, comment-only unsigned template to the
missing-approval bootstrap result (exit `3`), never to an approval.

No automation writes approval state. The approved graph commit is derived at
read time from the default branch's first-parent history, so a ruleset that
requires pull requests, forbids bypass and requires CODEOWNER review on
`trust/**` is fully supported. The derived commit is a graph baseline, not a
decision stamp; see [Approved export](approved-export.md).

`trustbom init --interactive` collects real reviewers and previews
single-system setup changes. With consent it can inspect GitHub configuration
using existing `gh` authentication, but cannot grant workflow permissions or
enable protection. Confirm the protected approval journey without broadly
disabling review.
Workflow regeneration replaces custom content only after explicit preview
and confirmation. Bare `init` continues to preserve existing workflows.
