# `trustbom request-approval`

## Synopsis

```text
trustbom request-approval [--by <identity>] [--system <id>] [--baseline <path>]
    [--fail-on <severity>] [--high-risk yes|no] [--article-50 applies|not_applicable]
    [--dry-run] [--check-controls]
```

## Purpose

Authors the approval-file proposal that an independent CODEOWNER reviews. There
is no `trustbom approve` command: approval is a normal, reviewed git change.

## Approve a graph in git

Approval is a normal, reviewed git change to `trust/approvals.yml`: there is
no separate UI or button. A Trust reviewer (the human authorized to publish
on behalf of the vendor) edits the file to record the **full** graph fingerprint
shown in the gate's approval example (not the 12-character display prefix),
for example:

```yaml
approved_graph_sha: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
approved_by: trust@example.com
approved_at: 2026-09-23T15:30:00Z
high_risk: unset
article_50: unset
waivers: []
rejections: []
```

`high_risk` and `article_50` are human-only decisions the scanner never
infers (`unset`, or `no`/`yes` and `applies`/`not_applicable` once Trust
decides). Waivers cover specific findings until a `yyyy-MM-dd` date;
non-empty `rejections` always keep the gate failing until resolved. This
change is protected by CODEOWNERS like any other reviewed file, so merging it
is the signature. Anyone with write access can propose the entry, but GitHub
does not let an author approve their own pull request, so a CODEOWNER with the
Trust role who is not the author must approve it; a vendor therefore needs at
least two people able to review. `approved_by` and `approved_at` are unverified
claims; the GitHub review and merge record are authoritative. The CLI records
`approved_at` but does not enforce it (a future timestamp is not rejected); the
merged approval in Git history, not this timestamp, is the authority. Committing
the approval changes `HEAD`, so rescan after committing it before a draft
export. To author the proposal without copying a fingerprint or timestamp, run
the command below.

The command scans tracked inputs without changing scan artifacts, checks
whether the candidate entry would pass the gate, and writes only the selected
approval file. It refuses with exit `2` and no changes for unwaived findings
at the threshold (default `medium`), rejections, break-glass, invalid manual
input or missing identity. A threshold refusal lists the blocking finding ids;
add a waiver for each in the approval file, or choose another `--fail-on`. Without `--by`, it uses `git config user.email`;
neither value verifies identity. An unchanged rerun leaves the approval file
byte-identical.

Once merged to the default branch, no automation writes approval state: the
approved graph commit is derived at read time from default-branch first-parent
history (the oldest commit of the contiguous run carrying the approved graph
hash). It is a graph baseline, not a decision stamp, and is never stored in
`trust/approvals.yml`. From this
point, `card.md` and the questionnaire are labelled `approved` instead of
`draft` for that derived commit. Approved output is refused when the default
branch cannot be proven (run `git remote set-head origin -a`; a local `main` or
`master` is not evidence, and a fetched `origin/<base>` named by `GITHUB_BASE_REF`
also counts), the clone is shallow, or the tip graph does not match the approval.

## Options

| Option | Meaning |
| --- | --- |
| `--by <identity>` | Claimed proposal identity (defaults to `git config user.email`; not verified). |
| `--system <id>` | Select exactly one portfolio system to propose independently. |
| `--baseline <path>` | Repository-relative approval file to write (default `trust/approvals.yml`). |
| `--fail-on <severity>` | Minimum finding severity requiring a waiver (default `medium`). |
| `--high-risk yes\|no` | Explicit human decision; otherwise it stays `unset` or retains the existing value. |
| `--article-50 applies\|not_applicable` | Explicit human decision; otherwise it stays `unset` or retains the existing value. |
| `--dry-run` | Print the proposal without writing it. |
| `--check-controls` | Opt-in, read-only advisory report of GitHub review controls (verified, missing, unverified, outstanding); not proof of approval. It includes the recommended required `Scan and gate` status check and "require branches to be up to date" policy. A missing `gh` or unreadable API is unverified. |

```text
trustbom request-approval --by trust@example.com
trustbom request-approval --system payments --fail-on high --dry-run
trustbom request-approval --check-controls
```

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The proposal was written (or printed with `--dry-run`), or an unchanged rerun. |
| `1` | Parser or usage error. |
| `2` | Refused with no changes: unwaived findings at the threshold, rejections, break-glass, invalid manual input, missing identity, or an invalid option value. |

## Licence

Free. `request-approval` never checks a licence and never approves anything.

## Example

Against a repository whose findings are resolved (real output; the timestamp
is the time of the run):

```powershell
trustbom request-approval --by trust@example.com --dry-run
```

```text
Dry run for trust/approvals.yml (no files written):
approved_graph_sha: 42ff94542d7806435413a1de532e05b98f5cf7c5cdc150fd58d059c41902ac1f
approved_by: "trust@example.com"
approved_at: 2026-11-18T14:02:23Z
high_risk: unset
article_50: unset
Rescan, then commit the proposal file trust/approvals.yml with the rescanned trust/card.json and trust/card.md in a pull request; a CODEOWNER other than its author should review and approve the PR; GitHub enforces this only after `trustbom github-rules apply`. approved_by is an unverified claim; only a merged default-branch graph can be exported.
```

Exit code `0`. Without `--dry-run`, and with the human decisions stated:

```powershell
trustbom request-approval --by trust@example.com --high-risk no --article-50 not_applicable
```

```text
Wrote proposal to trust/approvals.yml (graph 42ff94542d7806435413a1de532e05b98f5cf7c5cdc150fd58d059c41902ac1f).
Rescan, then commit the proposal file trust/approvals.yml ...
```

## Common errors

With three unresolved `medium` findings the command refuses (exit `2`, nothing
written):

```text
Proposal refused: 3 finding(s) at or above 'medium' have no active waiver (model-id-undeclared, no-models-detected, subprocessor-undeclared).
  model-id-undeclared: The anthropic package is present but no model ID was found in an allowlisted configuration file.
  ...
The default --fail-on threshold is 'medium'; this run used 'medium'. Fix the findings, add a waiver for each in trust/approvals.yml, or choose another threshold with --fail-on, then run request-approval again.
```

Fix the findings (see [manual declarations](../manual-declarations.md)), add a
waiver for each, or choose another `--fail-on`.

## See also

- [`gate`](gate.md) and [`export`](export.md)
- [Reducing review friction](../reducing-review-friction.md): keep this review
  step light without weakening it
- [Enforcement levels](../enforcement-levels.md) and [`github-rules`](github-rules.md)
