# How the approval is proven

This is a reference page for the tutorial. It uses the precise terms that the
tutorial pages avoid; the glossary defines
them. The authoritative rules are in
[approving a graph in Git](../cli/propose.md#approve-a-graph-in-git) and the
[approved export guide](../approved-export.md).

## The record is a graph with a fingerprint

`trust/card.json` holds the **system graph**: the models, tools, MCP servers,
declarations, findings and classifications, each with its evidence. Cards,
questionnaires, CycloneDX files, SARIF and buyer packs are rendered from it;
none of them derives facts of its own.

The graph **fingerprint** is a SHA-256 hash of the graph content. It excludes
volatile metadata (the commit id, scan time and CLI version) and renderer
wording. So:

- rescanning after an unrelated commit leaves `card.json` untouched, and
  needs no new sign-off: its `commit`, `scanned_at` and `cli_version` record
  the last graph change, not the last scan;
- changing a model, provider, declaration, finding or decision changes the
  fingerprint, and the old approval no longer matches.

## A proposal binds one fingerprint

`trustbom propose` writes the full 64-character fingerprint to
`trust/approvals.yml` as `approved_graph_sha`, with the two human decisions
(`high_risk`, `article_50`), `approved_by` and `approved_at`. `approved_by`
comes from `git config user.email` and is a claim, not a verified identity;
GitHub's review and merge record is the authoritative evidence of who
approved.

On a branch, the file is only a proposal, and the status is
**Proposed, not approved**.

## Approval is the merge to the default branch

A graph is **Approved** only when `trust/approvals.yml` on the **default
branch** records the fingerprint of the graph being evaluated. No automation
writes approval state. The merge of the reviewed pull request is the
signature.

The CLI proves the default branch from the remote (`origin/HEAD`, or the
`origin/<base>` that a pull-request run names), never from a local `main`,
and refuses on a shallow clone. That is why tutorial page 6 runs `git pull`
before the approved commands: they read what GitHub holds.

The **approved graph commit** is derived, at read time, from the default
branch's first-parent history: the oldest commit of the latest contiguous
run that carries the approved fingerprint. It is a **baseline** for later
comparisons, not a decision stamp, and it is never stored. Merge commits keep
first-parent history to one commit per merged pull request, which is why the
tutorial merges with a merge commit and the ruleset allows merge commits
only.

## What the pull-request check compares

The GitHub Action scans the pull request's checkout and runs the **gate**:

| Situation | Gate status | Exit |
| --- | --- | --- |
| No approval on the default branch yet (tutorial pages 3 and 4) | `BootstrapRequired` | `3` |
| The graph differs from the approved baseline, a finding is at the threshold, or `trust/card.json` is stale (page 9) | `Failed` | `3` |
| The graph matches the approval in the checkout, with no blocking finding (pages 5 and 9) | `Passed` | `0` |

The check passes on a proposal because the checkout's `approvals.yml` matches
its graph. That does not make it **Approved**: until the merge, the default
branch still holds the old approval, and approved export uses that. Findings
never fail `scan`; only the gate evaluates them.

A **stale** `card.json` is one whose committed graph differs from the pull
request's scan. The check fails so that the reviewer sees the real graph and
export does not refuse after the merge. See
[stale `card.json`](../github-action.md#stale-cardjson).

## Export always uses the approved graph

Approved `export`, `answer` and `publish` use the default branch's approved
graph, whatever branch is checked out. `export --draft` writes the current
branch's graph, watermarked as an internal draft. A break-glass pass of the
gate is always a draft and never authorizes an approved export.

## What enforcement adds

Without the ruleset, the sign-off is a convention: anyone with write access
can merge an approval, including its author (tutorial pages 1 to 8). The
ruleset that `trustbom github-rules apply` creates requires a pull request
approved by someone other than the last pusher, CODEOWNER review, the
**Scan and gate** check and merge commits. `trustbom github-rules check
--github` reports the level (`enforced`, `audited`, `violation` or
`unverified`); see [enforcement levels](../enforcement-levels.md). The level
describes how approval changes were controlled. It is never an approval and
changes no graph or hash.
