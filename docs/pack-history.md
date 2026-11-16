# Review pack history

`trustbom changelog` compares committed `card.json` graphs using Git history.
With no options, it compares the last two **distinct, historically approved**
graph baselines for the root card:

```powershell
trustbom changelog
```

To compare an older approved baseline with a committed branch draft, give
**both** commit SHAs:

```powershell
trustbom changelog --from <approved-commit-sha> --to <branch-commit-sha>
```

`--from` must be an approved graph baseline: the oldest commit of a contiguous
run on the default branch's first-parent history in which the graph and its
approval hash match (derived, never stored). `--to` may be a
committed branch graph; if it has no independently verified baseline, the
changelog says **internal draft - unapproved - not for customer use**. To
review a portfolio system, add `--system <id>` to either form; this selects
`trust/systems/<id>/card.json` and its own approvals. Historical approval
validation uses the `medium` finding threshold by default; set
`--fail-on high` (or your configured threshold) when that was the policy
used for the baselines being compared.

The output is Markdown on stdout. It lists semantic additions, removals, and
changes including models, tools, MCP servers, model regions, and evaluation
pass/fail or score changes. For example, an added binding appears as
`+ Model Anthropic claude-haiku-4`. It does not write an export artifact,
change approvals, or use the current uncommitted checkout. Changes to scan
timestamps, evidence line numbers, and renderer wording do not appear as
graph changes.

Approval is verified using the default branch's historical approval file
and the hash-checked graph at the baseline commit. A **historically approved**
label describes the state at that baseline; it is not permission to publish
that old graph today. Use `trustbom export` or `trustbom publish` to enforce
the current default-branch approval and publication policy.

An unavailable default branch, missing or shallow history, unverified
baseline, malformed or tampered graph, or fewer than two verified baselines
in implicit mode causes a nonzero error with no Markdown. Fetch the relevant
Git history when using a shallow checkout; there is no fallback to a draft.
History discovery supports at most 1,000 changes to the selected approval
file, and a changelog with more than 500 semantic changes is refused rather
than silently truncated.
