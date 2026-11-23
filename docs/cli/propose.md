# `trustbom propose`

## Synopsis

```text
trustbom propose [--by <identity>] [--system <id>] [--baseline <path>]
    [--fail-on <severity>] [--high-risk yes|no] [--article-50 yes|no]
    [--dry-run]
```

## Purpose

Writes everything a sign-off needs, in one step: the approval-file proposal
that an independent CODEOWNER reviews, and the scan artifacts (`card.json`,
`card.md`, `questionnaire.yml`, `findings.sarif`) from the same scan, so no
rescan is needed. It prints the `git add` line for the two tracked files. There
is no `trustbom approve` command: approval is a normal, reviewed git change.

After `trustbom scan` reports **Not approved, Ready for sign-off**, run
`trustbom propose`, commit what it prints, open a pull request, and have Trust
review and merge it.

## Approve a graph in git

Approval is a normal, reviewed git change to `trust/approvals.yml`: there is
no separate UI or button. A Trust reviewer (the human authorized to publish
on behalf of your company) reviews the proposal, which records the **full**
graph fingerprint (not the 12-character display prefix), for example:

```yaml
approved_graph_sha: 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef
approved_by: trust@example.com
approved_at: 2026-09-23T15:30:00Z
high_risk: no
article_50: yes
waivers: []
rejections: []
```

`high_risk` and `article_50` are human decisions the scanner never infers, and
the only accepted values are `yes` and `no`. (Buyer-facing output words a
`yes` for Article 50 as "applies" and a `no` as "not applicable".) Waivers
cover specific findings until a `yyyy-MM-dd` date; non-empty `rejections`
always keep the gate failing until resolved. This change is protected by
CODEOWNERS like any other reviewed file, so merging it is the signature.
Anyone with write access can propose the entry, but GitHub does not let an
author approve their own pull request, so a CODEOWNER with the Trust role who is
not the author must approve it; your company therefore needs at least two people
able to review. `approved_by` and `approved_at` are unverified claims (output
labels `approved_by` "claimed, unverified"); the GitHub review and merge record
are authoritative. The CLI records `approved_at` but does not enforce it (a
future timestamp is not rejected).

Once merged to the default branch, no automation writes approval state: the
approved graph commit is derived at read time from default-branch first-parent
history (the oldest commit of the contiguous run carrying the approved graph
hash). It is a graph baseline, not a decision stamp, and is never stored in
`trust/approvals.yml`. From this point `scan`, `gate`, `card.md` and the
questionnaire show **Approved** instead of **draft** for that derived commit.
**Approved** is shown, and approved export is allowed, only when the default
branch can be proven (run `git remote set-head origin -a`; a local `main` or
`master` is not evidence, and a fetched `origin/<base>` named by
`GITHUB_BASE_REF` also counts), the clone is not shallow, and the tip graph
matches the approval.

## How it works

`propose` scans tracked inputs and evaluates the same readiness rules as the
gate. It refuses with exit `2` and writes nothing when anything blocks a
proposal: unwaived findings at the threshold (default `medium`), rejections,
break-glass, invalid manual input, an invalid approval file or a missing
identity. Each blocker is listed with a `Next:` line naming the fix.

- **First sign-off.** Both decisions are required. At a terminal, `propose`
  asks the two questions in plain words when a flag is missing; in a
  non-interactive run, pass `--high-risk` and `--article-50`.
- **Later sign-offs.** The recorded decisions are carried forward, so a changed
  graph needs only a plain `trustbom propose`. Pass a flag to change a decision.
- **Unchanged rerun.** When the proposal already matches the graph and
  `trust/card.json` is current, it prints "already recorded" and changes nothing.
  If only the artifacts are stale, it refreshes them.
- **Atomic write.** `trust/approvals.yml` is written through a temporary file;
  existing waivers and comments are preserved.
- **Identity.** Without `--by`, it uses `git config user.email`; neither value
  verifies identity.

## Options

| Option | Meaning |
| --- | --- |
| `--by <identity>` | Claimed proposal identity (defaults to `git config user.email`; not verified). |
| `--system <id>` | Select exactly one portfolio system to propose independently. |
| `--baseline <path>` | Repository-relative approval file to write (default `trust/approvals.yml`). |
| `--fail-on <severity>` | Minimum finding severity requiring a waiver (default `medium`). |
| `--high-risk yes\|no` | Trust's decision on EU AI Act high-risk status. Required on the first sign-off (asked at a terminal if omitted); afterwards it replaces the recorded value. |
| `--article-50 yes\|no` | Trust's decision on whether the Article 50 transparency duties apply. Required on the first sign-off (asked at a terminal if omitted); afterwards it replaces the recorded value. |
| `--dry-run` | Print the proposal without writing any file. |

```text
trustbom propose --high-risk no --article-50 yes
trustbom propose --system payments --fail-on high --dry-run --high-risk no --article-50 no
trustbom propose
```

Repository controls are not checked here: see
[`github-rules check`](github-rules.md#check-the-enforcement-level).

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The proposal was written (or printed with `--dry-run`), or an unchanged rerun. |
| `1` | Parser or usage error. |
| `2` | Refused with no changes: blockers (unwaived findings at the threshold, rejections, break-glass, invalid manual input or approval file), missing identity, missing first-sign-off decisions in a non-interactive run, or an invalid option value. |

## License

Free. `propose` never checks a license and never approves anything.

## Example

Against a repository whose findings are resolved (real output; the timestamp
is the time of the run). Preview first:

```powershell
trustbom propose --dry-run --high-risk no --article-50 yes
```

```text
Dry run for trust/approvals.yml (no files written):
approved_graph_sha: 3dce16d517c53a3faee91a3384f4e9f5df9b241d446ea108a605d28aa72a93c8
approved_by: "trust@example.com"
approved_at: 2026-11-20T02:22:37Z
high_risk: no
article_50: yes
```

Exit code `0`. Then write it:

```powershell
trustbom propose --high-risk no --article-50 yes
```

```text
Wrote proposal to trust/approvals.yml (graph 3dce16d517c53a3faee91a3384f4e9f5df9b241d446ea108a605d28aa72a93c8).
Wrote <repository>\trust\card.json
Wrote <repository>\trust\card.md
Wrote <repository>\trust\questionnaire.yml
Wrote <repository>\trust\findings.sarif
Found 1 model(s), 0 tool(s), 0 finding(s). Review <repository>\trust\card.md.
Status: Proposed, not approved
Next: Commit trust/approvals.yml and trust/card.json, open a pull request, and have a CODEOWNER other than you approve and merge it.
git add trust/approvals.yml trust/card.json
Commit them and open a pull request; a CODEOWNER other than its author approves and merges it (GitHub enforces this only after `trustbom github-rules apply`). approved_by is a claimed identity; only the merged default-branch graph is Approved and exportable.
```

Commit the two files and open the pull request; after the merge the status is
**Approved**. A rerun with nothing changed prints
`Proposal already recorded in trust/approvals.yml; no file changes.`

## Common errors

With unresolved `medium` findings the command refuses (exit `2`, nothing
written):

```text
Proposal refused: 2 item(s) must be resolved first.
  - [medium] region-unknown: The deployment region for gemini could not be determined.
    Next: Declare the region for 'gemini' under models: in trust/manual.yml (provider, model_id, region, reason).
  - [medium] subprocessor-undeclared: Provider 'gemini' has no declared subprocessor.
    Next: Add 'gemini' to trust/subprocessors.yml, or list the path under not_in_scope: in trust/manual.yml if it is a false positive.
```

Follow each `Next:` line (see [manual declarations](../manual-declarations.md)),
commit the change, and run `propose` again. Other refusals:

- **`The first sign-off needs both decisions`**: pass `--high-risk yes|no` and
  `--article-50 yes|no` (a non-interactive run cannot ask).
- **`'applies' is no longer accepted`** (or `unset`, `not_applicable`, or any
  value other than `yes|no`) in `trust/approvals.yml`: earlier releases accepted
  other words. Run `trustbom propose --high-risk yes|no --article-50 yes|no` to
  replace them, or edit the file.
- **`Proposal refused: provide --by or configure git user.email`**: pass `--by`
  or set `git config user.email`.

## See also

- [`scan`](scan.md), which reports when the repository is ready for sign-off,
  [`gate`](gate.md) and [`export`](export.md)
- [`github-rules`](github-rules.md): the repository controls that make the
  review binding
- [Reducing review friction](../reducing-review-friction.md): keep this review
  step light without weakening it
- [Enforcement levels](../enforcement-levels.md)
