# Enforcement levels

AI TrustBOM's approval is a pull request that changes `trust/approvals.yml`,
reviewed by a CODEOWNER who is not its author. GitHub rules make that control
**preventive**. Without them it can still be **detective**: the history shows
whether every approval change was reviewed. This guide explains the four
levels `trustbom github-rules check` reports, what each does and does not
guarantee, and how to move up.

A level describes how changes to approval files were controlled. It is never an
approval, and it changes no graph, hash or approval file.

## The four levels

| Level | Meaning | Preventive? | Exit |
| --- | --- | --- | --- |
| `enforced` | All seven rules below are verified on the default branch and the audit found no violation. | Yes: GitHub blocks an uncontrolled approval change. | `0` |
| `audited` | Rules are not all verified, but every approvals-file change in the audit window was merged by a pull request approved on its final push by a CODEOWNER who was neither the author nor the last pusher. | No, detective only: it proves what happened, not that it could not have happened otherwise. | `0` |
| `violation` | An approvals-file change in the window was a direct push or lacked that review. The finding names the commit. It outranks verified rules: they prevent future bypass, not a past one. A reviewed pull request that re-approves a changed graph (a new `approved_graph_sha`) starts a new window and clears it; a waivers-only change does not. | No | `3` |
| `unverified` | Rules or history could not be read or proven (offline, shallow clone, no access, no `gh`). | No | `0` |

`unverified` is never `audited` or `enforced`: absence of evidence is unknown.
`--require-enforced` exits `3` for every level below `enforced`.

## The seven rules

`enforced` needs the `trustbom-approval` repository ruleset to be:

1. active, targeting the default branch, with no bypass actors;
2. requiring a pull request with one approval;
3. requiring CODEOWNER review;
4. dismissing stale approvals when new commits arrive;
5. requiring approval of the most recent reviewable push;
6. blocking force pushes and branch deletion;
7. allowing merge commits only.

Each rule reports `verified`, `missing` or `unverified`.

## The audit window

The audit covers every commit that changed an approvals file
(`trust/approvals.yml` or `trust/systems/*/approvals.yml`) from the approved
graph baseline to the branch tip. The baseline is derived from default-branch
first-parent history and is never stored. A repository with no approvals yet
has an empty, clean window.

Limits, so you can judge what `audited` is worth:

- The last pusher is approximated by the author or committer of the pull
  request's final commit.
- Only repository rulesets are verified, not legacy branch protection or
  organisation rulesets.
- Bypass actors are hidden from non-admin logins (confirmed live), so the
  first rule is `unverified` and the level is `audited`. This also applies to
  a workflow's default `GITHUB_TOKEN`. Run `trustbom github-rules check
  --github` with an administrator login to see `enforced`.
- A shallow clone cannot be audited.

## Moving from `audited` to `enforced`

You do not need to be an administrator to adopt AI TrustBOM, and the gate works
at `audited`. Making it `enforced` needs one administrator action:

```powershell
trustbom github-rules apply --dry-run
trustbom github-rules apply
trustbom github-rules check --github --require-enforced
```

`init` writes the ruleset as `.github/rulesets/trustbom.json`. If you are not an
administrator, `apply` changes nothing and prints the exact steps for one;
send those to them. Re-run `check --github` afterwards.

## Where the level appears

- `trustbom github-rules check [--github]` reports the level and each rule.
- `trustbom gate --github-rules` adds an "Enforcement level" section. A level
  below `enforced` is a finding; only `violation` fails the gate.
- `trustbom export --github-rules` appends an "Approval controls" section to
  `export/card.md`. The buyer pack does not state the level because it cannot be
  proven centrally.

All three are off or offline by default; they need `gh` and a token only when
asked.

## Related

- [CLI reference: applying and checking the ruleset](cli/github-rules.md#apply-the-ruleset)
- [GitHub Action guide](github-action.md#recommended-repository-controls)
- [Reducing review friction](reducing-review-friction.md)
- [Approved export](approved-export.md)
