# Portfolio in one repository

A **portfolio** is several AI systems under one Trust. Each system keeps its
own graph and its own reviewed approval; Trust approves them one by one, and
approving one system never approves another.

## Which setup do I need?

| Your systems | Use |
| --- | --- |
| Live in one repository (a monorepo) | This page. |
| Live in separate repositories | Give each repository its own TrustBOM, as in the [tutorial](tutorial/README.md), and deliver each one's evidence separately. |

For a worked example, follow
[Several AI systems in one repository](tutorial/portfolio-monorepo.md).

The [portfolio reference](portfolio-reference.md) lists every workflow input,
token and CLI interface.

## Set up

Run `trustbom init --portfolio` from the repository. It creates any missing
portfolio example (`trust/portfolio.yml`), CODEOWNERS example,
`.github/workflows/ai-trustbom-portfolio.yml` caller, the reusable
`.github/workflows/portfolio.yml` it calls, and the per-repository Action in
`.github/actions/trustbom/` (the same `action.yml` and `install/action.yml`
that `trustbom init` writes). `--dry-run` lists the same files without writing
them. Nothing refers to the AI TrustBOM source repository. Existing files are
**never replaced**.

Then:

1. Edit the example `apps/example` path to a real tracked, non-overlapping
   system path, and add one entry per system.
2. Replace the example CODEOWNERS teams and make sure CODEOWNERS protects both
   `trust/portfolio.yml` and `trust/systems/*/approvals.yml`.
3. Create reviewed, separately merged approvals for each system. `init` never
   fabricates approved graphs, and the workflow never writes approval state for
   Trust.
4. Apply the ruleset with `trustbom github-rules apply`. It requires the
   `portfolio / Portfolio approval gate` check, so a pull request whose gate
   fails cannot merge. If the repository was set up for one system first, its
   ruleset still requires `Scan and gate`, which the portfolio workflow never
   reports; `init --portfolio` warns about it. Change that check in
   `.github/rulesets/trustbom.json` before applying.

Because pull requests run their own copy of the workflow and Action,
`init --portfolio` adds CODEOWNERS rules for
`/.github/workflows/ai-trustbom-portfolio.yml`,
`/.github/workflows/portfolio.yml` and `/.github/actions/trustbom/`; require
code-owner review on those paths.

### A repository set up with `--growth`

Earlier versions called this setup "Growth" and wrote
`ai-trustbom-growth.yml` and `growth.yml`. `trustbom init --growth` is no
longer an option, and `trustbom init --upgrade` neither renames nor rewrites
those files; it lists them and the new names. Either rename the two workflows
(and the caller's `uses:` path) by hand through a reviewed pull request, or
delete them and run `trustbom init --portfolio`. Either way, change the
CODEOWNERS lines to the new paths: `init` never replaces an existing
CODEOWNERS file, so the new workflows would otherwise lack code-owner review.
A ruleset written by `--growth` requires no status check: delete
`.github/rulesets/trustbom.json`, rerun `trustbom init --portfolio` to write
one that requires the portfolio gate, and apply it.

## What runs

The caller listens to pull requests and pushes to the detected default branch.
It grants `contents: read`, `pull-requests: write` and `checks: write`; the
called workflow confines write scopes to the pull-request review job. Fork pull
requests still scan and gate, but the Action skips privileged comment and check
publication. Do not use `pull_request_target`.

On a pull request, the portfolio Action scans and gates each declared system,
publishes one marked portfolio comment and a bounded check when permitted, and
uploads a draft card per system. It enforces the gate after those uploads: an
Action failure, missing output, blocked policy or failed draft upload fails the
job. A break-glass soft pass remains a draft and cannot authorize approved
publication.

Only a push to the default branch can run the buyer job. It requires the
Action's `passed` gate status (not `breakglass`), then runs
`trustbom pack`, optional `trustbom answer --portfolio`, and
`trustbom publish`, each validating the approved graph independently; a
partial or failed run uploads no buyer-ready artifact. A successful run uploads `trustbom-APPROVED-buyer-bundle` and one
`trustbom-APPROVED-<system-id>-card` per system. It never uploads pull-request
draft cards as approved bundles.

`fail-on` defaults to `medium`. `questions-csv` is optional and must name a
tracked CSV with `id,text` (and optionally `section`); unmatched answers cause
a nonzero exit and prevent buyer-ready upload, so Trust reviews them first.

## Licensing

Scanning and gating a portfolio are free for any number of systems. Delivery
(`pack`, `publish`) and answering (`answer`) use the organization license, so
the default-branch buyer job needs the `license-key` secret.
`trustbom init --portfolio` maps it from a repository secret named
`TRUSTBOM_LICENSE_KEY` and passes it to that job only; see
[Create the `license-key` secret](licensing.md#create-the-license-key-secret).
Pull requests (including forks) run only the free review job and never receive
the key. An empty secret on a push fails with the CLI's `license-missing`
diagnostic.

## Upgrade

The engine is the published `trustbom` npm package, pinned in
`.github/actions/trustbom/install/action.yml` to the version of the CLI that ran
`init --portfolio` (a development build refuses to pin itself; set
`TRUSTBOM_ACTION_VERSION` to a published version to override). The runner needs
Node.js, npm and Git. Upgrade with `trustbom init --upgrade` (see
[upgrading generated files](cli/init.md#upgrading-generated-files)) or by
changing the version in the install action through a reviewed pull request.
