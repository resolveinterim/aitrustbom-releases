# `trustbom init`

## Synopsis

```text
trustbom init [--dry-run] [--interactive] [--portfolio] [--upgrade]
```

## Purpose

Writes the starter `trust/` files (`trust/system.yml`, an empty
`trust/manual.yml` template, an unsigned `trust/approvals.yml` template, and
CODEOWNERS guidance) plus a consumer GitHub Actions workflow. Existing files are
preserved, not overwritten. It also adds an `export/` line to `.gitignore`
(creating the file if needed, appending once and keeping existing content), so a
draft export is not committed by `git add -A`. To commit an export deliberately,
use `git add -f export/` or remove that line. Use `--dry-run` first if you want
to see the plan without creating anything.

`init` and `init --dry-run` also work in a new repository before its first commit;
`scan`, `gate` and `export` still require a commit.

Review the generated files, replace the CODEOWNERS example team names with
your own reviewers, then commit them.

`init` never contacts the license service, never asks for a trial key and keeps
no local first-use state. A trial key is requested later, from the first paid
command or `trustbom license request-trial`, and the 30-day trial starts when
that key is issued. See the [licensing guide](../licensing.md).

`init` also writes `.github/rulesets/trustbom.json`, the ruleset that makes
CODEOWNER approval independent: active on the default branch, no bypass actors,
a pull request with one approval, CODEOWNER review, stale approvals dismissed,
approval of the most recent reviewable push, no force pushes or deletion, and
merge commits only. It also requires the gate's status check: `Scan and gate`,
or `portfolio / Portfolio approval gate` with `--portfolio`. Apply it with [`github-rules apply`](github-rules.md#apply-the-ruleset).

## Options

| Option | Meaning |
| --- | --- |
| `--dry-run` | Show the initialization plan without creating files. |
| `--interactive` | Guide single-system setup with previews, confirmed edits and optional read-only GitHub checks. Requires a terminal. |
| `--portfolio` | Scaffold a portfolio (several AI systems under one Trust), its workflow and per-repository Action instead of the single-system starter. Creates a portfolio template, CODEOWNERS guidance, a local reusable portfolio workflow and caller, and the pinned per-repository Action without overwriting existing files. See [Portfolio in one repository](../portfolio-one-repository.md). |
| `--upgrade` | Rewrite only unmodified files that carry the generated-file marker, bumping the pinned `trustbom` version. Combine with `--dry-run` to preview. |

`--upgrade` cannot be combined with `--portfolio` or `--interactive` (exit `2`,
nothing written); `--dry-run` works with `--upgrade`. `--interactive --portfolio`
is unsupported.

### Guided single-system setup

```powershell
trustbom init --interactive --dry-run
trustbom init --interactive
```

The wizard requires non-redirected terminal input/output and an existing Git
repository with a commit. It asks for service name, description,
customer-facing name, Trust `@username` or `@organization/team`, and the
workflow's default branch. Existing identity/reviewer values supply defaults
where they can be read safely. It does not declare model use or control facts.
`--interactive --portfolio` and existing portfolio repositories are unsupported;
use the [portfolio setup guide](../portfolio-one-repository.md) instead.

The preview distinguishes new files from proposed updates. Existing
`trust/manual.yml` and `trust/approvals.yml` are always preserved; new
approvals remain unsigned. Supported single-line identity YAML can be edited
without removing comments. Custom, tagged, anchored or multiline identity
YAML is preserved with a manual-edit explanation. Selected CODEOWNERS exact
entries for approvals, CODEOWNERS itself and the TrustBOM workflow go last,
overriding earlier wildcard owners for those paths. Review that precedence
change before approving it. If GitHub currently uses root or docs CODEOWNERS,
their existing rules are carried into the new, higher-precedence
`.github/CODEOWNERS`; the original file is not overwritten.

Regenerating `.github/workflows/ai-trustbom.yml` requires a separate explicit
confirmation: it replaces custom jobs, filters, permissions and pins with
the release's tested workflow. Decline to retain your existing workflow.
Every existing-file change is previewed, and final Apply defaults to No.
Changed-since-preview files are rejected. Failed application attempts roll
back this run's changes without intentionally overwriting subsequent edits;
any restoration/cleanup failure identifies files needing inspection.

With consent, an optional installed and authenticated GitHub CLI (`gh`)
performs bounded **GET-only** checks on `github.com`: repository/default
branch, caller identity/admin access, reviewer/team access and visibility,
Actions permissions, base-branch CODEOWNERS errors, legacy branch protection,
and effective rulesets. The wizard never installs/authenticates gh, requests
a token, changes remote settings, or grants access. Missing gh/auth,
permission errors, unsupported origins/GHES and unavailable API data remain
**unverified**, not implicitly configured. Proposed local CODEOWNERS are
not yet the remotely effective base-branch rules.

Local success exits `0` even with outstanding administrator tasks. Check
the reported tasks and settings links: require approving CODEOWNER review,
dismiss stale reviews, and select the actual workflow gate job after its
first run. An observed protection setting is not proof that bypass actors
are constrained. Private organization
repository protection needs a paid GitHub plan; use synthetic public
repositories for GitHub Free enforced-review tests.

Dry-run asks questions and may perform consented remote reads, but writes
nothing and never scans. Declining Apply changes nothing and exits `0`.
Cancellation/EOF, invalid mode/terminal, local file failures and a requested
scan's operational failure exit `2`. Pressing Ctrl+C at a prompt was observed
to exit `130` (the shell's 128 + SIGINT) in a Linux container, with the same
message and nothing written before Apply. A scan failure after setup retains the
completed setup and reports that explicitly.

After applying, the wizard lists untracked setup files and pauses so you
can stage intended inputs in another terminal yourself. It never runs
`git add`. The opt-in scan writes draft artifacts and displays findings;
findings alone do not fail scan. The final Administrator/Engineer/Trust
handoff explains protected approval and the derived approved graph baseline. Local
onboarding never means a graph is approved or ready for buyer export.

### Upgrading generated files

```powershell
trustbom init --upgrade --dry-run
trustbom init --upgrade
```

Files `init` owns carry a first-line marker
`# trustbom-generated: version=<trustbom version> sha256=<body hash>`. `--upgrade`
covers the single-system and portfolio files that exist, prints one line per file
(`upgrade`, `unchanged`, `preserve`, `create`) and rewrites only
files that carry the marker, still match its hash, and were generated by an
older or equal version, so the pinned `trustbom` version in the Action's
install action (`.github/actions/trustbom/install/action.yml`) moves to the
running CLI's version. First install the new exact package version (and its
platform packages) into the registry used by the Action; then run
`init --upgrade`, review and commit the generated changes. An internal
registry can be selected with `npm_config_registry` in the workflow. A missing install action is reported as `create` and
written. Files without the marker,
edited marked files, and files from a newer version are preserved; delete a
marker to take ownership of a file. It never touches `trust/*.yml`,
`approvals.yml` or CODEOWNERS, and it never stages or commits. The run
refuses (exit `2`) on a development build unless `TRUSTBOM_ACTION_VERSION` is set.

There is no migration from earlier private-repository
workflows: no customer repositories used them. Regenerate such a repository
with `init` after removing the old workflow files.

A repository set up with the removed `--growth` option has
`.github/workflows/ai-trustbom-growth.yml` and `.github/workflows/growth.yml`.
`--upgrade` lists each as `preserve` with the new name and changes nothing, and
the exit code is unchanged. Rename the workflows by hand (and the caller's
`uses:` path) in a reviewed pull request, or delete them and run
`init --portfolio`. Either way, update the CODEOWNERS lines to the new paths:
`init` never replaces an existing CODEOWNERS file, so the new workflows are
otherwise left without code-owner review. A ruleset written by `--growth`
requires no status check; replace its `.github/rulesets/trustbom.json` with
the one `init --portfolio` writes (delete it and rerun), then run
`github-rules apply`.

`init --portfolio` in a repository already set up for one system keeps that
ruleset, which requires `Scan and gate`, a check the portfolio workflow never
reports. It prints a warning; change the required check in
`.github/rulesets/trustbom.json` to `portfolio / Portfolio approval gate` and
run `github-rules apply`, or no pull request can merge once the ruleset is
applied.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success, including a dry run, a declined interactive Apply and outstanding administrator tasks. |
| `1` | Parser or usage error (unknown option). |
| `2` | Operational failure: Git or file failures, an incompatible option combination, cancelled interactive setup or invalid terminal, a failed requested scan, or `--upgrade` on a development build without `TRUSTBOM_ACTION_VERSION`. Ctrl+C at an interactive prompt was observed to exit `130`. |

## License

Free. `init` never checks a license, prompts or contacts the license service.

## Example

In a repository with one commit, preview and then write the scaffolding. The
output below is from a development build with
`TRUSTBOM_ACTION_VERSION=1.0.0` set (a development build cannot pin the Action
to itself); a released build needs no variable.

```powershell
trustbom init --dry-run
trustbom init
```

```text
Initialization plan for <repository>:
  create: trust/system.yml
  create: trust/manual.yml
  create: trust/approvals.yml
  create: .github/CODEOWNERS
  create: .github/rulesets/trustbom.json
  create: .github/workflows/ai-trustbom.yml
  create: .github/actions/trustbom/action.yml
  create: .github/actions/trustbom/install/action.yml
  create: .gitignore
Dry run complete; no files were created.
```

The second command prints the same plan, then:

```text
Created 9 file(s); preserved 0 existing file(s).
Next: review the generated files, replace CODEOWNERS examples, commit them, then run `trustbom scan` and `trustbom gate`.
Approval protection: .github/rulesets/trustbom.json holds the ruleset. A repository admin applies it with `trustbom github-rules apply`, or imports it in GitHub (Settings > Rules > Rulesets > Import a ruleset). Until then approvals are advisory.
```

Exit code `0` for both.

## Common errors

- **"This development build of trustbom is not published, so it cannot pin the
  generated Action to itself"** (exit `2`): install a released `trustbom`, or
  set `TRUSTBOM_ACTION_VERSION` to a published package version.
- Unknown option: the parser prints the usage text, writes nothing and exits `1`.

## See also

- [Installing the CLI](../installing.md)
- [`scan`](scan.md), the next command
- [`github-rules`](github-rules.md) to apply and check the approval ruleset
- [GitHub Action guide](../github-action.md) and [Enforcement levels](../enforcement-levels.md)
- [Portfolio in one repository](../portfolio-one-repository.md)
