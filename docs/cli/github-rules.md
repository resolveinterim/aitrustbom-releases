# `trustbom github-rules`

## Synopsis

```text
trustbom github-rules apply [--dry-run] [--yes]
trustbom github-rules check [--github] [--require-enforced]
```

## Purpose

`apply` creates or updates the `trustbom-approval` ruleset that makes approvals
independent: the ruleset `init` writes to `.github/rulesets/trustbom.json`
(active on the default branch, no bypass actors, a pull request with one
approval, CODEOWNER review, stale approvals dismissed, approval of the most
recent reviewable push, no force pushes or deletion, and merge commits only).
`check` reports how approval changes were controlled. Neither command changes
approval state, a graph or a hash.

## Options

| Subcommand and option | Meaning |
| --- | --- |
| `apply --dry-run` | Show what would change without calling a write endpoint. |
| `apply --yes` | Apply without the confirmation prompt (for automation). |
| `check --github` | Read the repository's rulesets and pull request history from GitHub using your `gh` login. |
| `check --require-enforced` | Exit `3` unless the level is `enforced`. |

Gate and export can also report the level: add `--github-rules` to
[`gate`](gate.md#options) or [`export`](export.md#options).

## Apply the ruleset

```powershell
trustbom github-rules apply --dry-run
trustbom github-rules apply
```

`apply` uses your existing `gh` login, previews the change and asks for
confirmation (`--yes` skips the prompt for automation). It creates the ruleset
or updates it in place by name, and a rerun makes no change. It needs
repository admin rights: without them it makes no write, exits `2` and prints
the settings URL, the payload path and the command an administrator can run.
Rulesets on private repositories need a GitHub plan that supports them. See
[Enforcement levels](../enforcement-levels.md) for what each level guarantees.

## Check the enforcement level

```powershell
trustbom github-rules check
trustbom github-rules check --github
trustbom github-rules check --github --require-enforced
```

`check` reports one level and the status of each of the seven rules
(`verified`, `missing`, `unverified`):

| Level | Meaning | Exit |
| --- | --- | --- |
| `enforced` | All seven rules are verified and the audit found no violation, so GitHub prevents uncontrolled approval changes. | 0 |
| `audited` | Rules are not all verified, but every change to an approvals file since the approved graph baseline was merged by a pull request approved on its final push by a CODEOWNER who was neither the author nor the last pusher. | 0 |
| `violation` | An approvals-file change in that window was a direct push or lacked such an approval; the finding names the commit. It outranks verified rules until a reviewed pull request re-approves a changed graph (waivers alone do not clear it). | 3 |
| `unverified` | The rules or history could not be read or proven (offline, shallow clone, no access). | 0 |

Without `--github` nothing leaves your machine and the level is `unverified`;
it still reports a CODEOWNERS file that does not cover an approvals file, with
the exact line to add. `--require-enforced` exits `3` for any level below
`enforced`. The level describes how approval changes were controlled; it is
never an approval and changes no graph, hash or approval file.

Limits: only repository rulesets are verified (not legacy branch protection or
organisation rulesets); reading bypass actors may need admin rights; the last
pusher is taken from the final commit's author on the pull request; a team
CODEOWNER login that GitHub reports as not a member counts as not an owner.

To report the level where reviewers read it, add `--github-rules` (needs `gh`
and a token; off by default so `gate` and `export` stay offline):

```powershell
trustbom gate --github-rules
trustbom export --github-rules
```

`gate` prints an "Enforcement level" section: a level below `enforced` is a
finding, and only `violation` fails the gate (exit `3`). `export` appends an
"Approval controls" section to `export/card.md` only; the manifest hashes the
written card, and nothing enters `card.json` or any approval hash. The buyer
pack does not state the level because it is a per-repository fact that cannot
be proven centrally. `--github-rules` cannot be combined with `--draft`.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success, including `check` levels `enforced`, `audited` and `unverified`, and a rerun of `apply` with nothing to change. |
| `1` | Parser or usage error. |
| `2` | `apply` failed: no repository admin rights (nothing written, the admin steps are printed), no usable `gh`, or an operational error. |
| `3` | `check` found `violation`, or a level below `enforced` with `--require-enforced`. |

## Licence

Free. `github-rules` never checks a licence.

## Example

An offline check in a repository where `init` has run (no GitHub calls):

```powershell
trustbom github-rules check
```

```text
AI TrustBOM enforcement level: unverified
  [unverified] Active ruleset on the default branch, no bypass actors: not checked (offline)
  [unverified] Pull request with 1 or more approvals: not checked (offline)
  [unverified] CODEOWNER review required: not checked (offline)
  [unverified] Stale approvals dismissed on new commits: not checked (offline)
  [unverified] Most recent reviewable push approved: not checked (offline)
  [unverified] Force pushes and branch deletion blocked: not checked (offline)
  [unverified] Merge commits only: not checked (offline)
Approval history audit: unverified (not checked (offline))
No GitHub calls were made. Pass --github to verify the rules and audit approval history.
This level describes how approval changes were controlled. It is not an approval.
```

Exit code `0`. Add `--github` to read the repository's rulesets through `gh`.

## Common errors

- `apply` without repository admin rights: exit `2`, nothing written; give the
  printed settings URL, payload path and command to an administrator.
- `check --github` without a usable `gh` login: the level stays `unverified`.
- `check --require-enforced` below `enforced`: exit `3`.

## See also

- [Enforcement levels](../enforcement-levels.md)
- [`init`](init.md), which writes `.github/rulesets/trustbom.json`
- [`gate`](gate.md) and [`export`](export.md) for `--github-rules`
- [`request-approval`](request-approval.md) for `--check-controls`
